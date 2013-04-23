---
layout: post
title: "On the stuff that you dont see in a Game"
date: 2013-04-23 12:53
comments: true
categories: Game,Server,Design,Perfomance
---
Many times, when I happen to talk to prospective clients or friends from the industry about multiplayer internet-based games and the issues related to their implementations, usually their reaction is a mixture of excitement about fetching User interfaces and how engaging the games are. Other common features like being able to play on mobile devices and quick sharing of scores through Facebook and Twitter also make it to the discussion. However, most do not appreciate, if acknowledge, the complexitites involved in designing and implementing the Servers for such multiplayer, online games. Games, in general, are associated with casual fun. Somehow, this association of non-seriousness is also extended to the technologies behind such games. To be specific, people do not seem to appreciate that to keep players engaged in games where fellow players may connect from faraway continents, opponents' moves travel across unreliable networks and yet, effect of these moves must be immediately visible, a lot of work goes on under the hood. This is the stuff that you don't see in a game but the stuff must not only be present but also be working all the time, reliably and predictably.

After being closely associated with building the stack for this 'under the hood' stuff – another name for the server-side machinery – on quite a few occasions, I find myself disappointed by the triviality ascribed to the Game Servers and the resultant disesteem, if I may add.  So, here is an attempt to describe a very high-level description of complexities that the multiplayer Game Servers have to deal with. 

I am restricting myself here to two broad categories of Games:

 1. Based on 'turns' taken by players; all the Players have to be present when the game continues; the rules of the game enforce the turns and provide actions when things happen out of turn; example: Poker.

   A subset of this category are the games which are based on turns taken by Players but – and here is the important difference – the all Players do not have to be present when one of them takes her turn; example: Word puzzle.

 2. Based on simultaneous actions by players: the rules of the game allows Players to act at the same time; example: PingPong!

Designing a Server for hosting such games is not trivial. Unreliable network connections and unpredictable latencies only form the preface. A number of other requirements make the plot quite intricate. Allow me to enumerate them:

##### Server is given no guarantee of timely receipt of messages #####

Firstly, user-interfaces (clients) of most of these games are based on
browsers (some are handcrafted C++ applications) and as such, the  interaction between the Client and Server is dependent on the Internet. Actions taken by the Player are sent as messages to the Server over the Internet and the responses from the Server are sent back via the same medium. It is perhaps obvious that the uncertainty about the timely receipt of messages from the Clients requires the Server to be extra cautious and be ready with fallback options. A message from a Player may not reach the Server within an expected interval - and at times may not reach ever – but the Server has to ensure that the game does not stall. In online, competitve games like Poker, this behaviour has tremendous significance.

The situation outlined above becomes a little more complex, when a message reaches the Server trifle late than when it is expected and the Server has continued meanwhile. The Player thinks that she has made the right move and expects the game to behave accordingly, but the Server takes a view that she has not responded at all. So, what gives? 

Usually, the Game Designer / Game Product Management provides subrules to be followed in such cases. These subrules focus mostly on User  Experience (I need feedback about my last move immediately) or Game's continuity (Ignore the message arrived late and proceed). In either of these or similar cases, the Server guarantees that the receipt of the message is acknowledged and acted upon; there is no such thing as a message unaccounted for, as far as the Server is concerned.

##### Server must enforce Game's rules #####

Simple or otherwise, every Game has its rules, and Players are expected to follow them. Even if they do not, the Server is supposed to enforce these rules. The Server validates every move by a Player against the ruleset and current state of the game and if found incorrect, generates an appropriate response. This response is meant only for the Player making the move but also for other Players. At the end of it all, the Server's job is to ensure that the Game remains in a predictable state unaffected by wrong moves - unwitting or otherwise – by Players.

##### Server must maintain and preserve Game's State #####

Amongst the key responsibilities of a multiplayer Game Server, the most important perhaps is the need to preserve the game's State. Here, the State  denotes a collection of a number of entities inlcuding:

* Identifier of the game's session

	An session of a Game is a programmable represenation of a match being played between at least two opponents. Soccer is a game, but a match  between two clubs is an session of Soccer. Poker is a game, but a table where a bunch of Poker experts are measuring each other up, is a session of Poker.

* TimeStamp of when this abovementioned session came into being

* Identifiers of the current Players associated with the session

	Every Player is identified in the system by an unique identifier. It is important to note, that the Players associated with an session of the Game  may vary with time. For example, in certain versions of Poker, Players are allowed to leave the table and rejoin later while other Players continue playing.

* Scores so far, of each Player or of the session of the Game

* Identifiers of the Observers of the Game\

	An session of the Game may have many observers. Imagine: spectators of a soccer match.

* Monetary data, if the Players and observers have placed bets

* Current stage of the Game

	The session of the Game may be in one of the different applicable stages, determined by the rules of the Game. A Poker table may be waiting for more Players to join. A soccer match may be in the middle of a break at the half-time. An Word-puzzle game may simple point to the Player who has to take the next turn.

Such information related to State of the Game assumes more importance when we begin to deal with issues like *Replication* (in case of some failure on the Server's part, the game must continue) or *Restart* (after the session of the Game has been suspended perhaps) or *Replay* (after the session of the Game is over).

##### Server's must exhibit low latency and high throughput #####

A Game's popularity is largely influenced by its perceived speed;. A Player is keen to see - as quickly as possible - the effect that her move has on the course of the session she is in. A slow feedback almost always makes a Game unpopular. The Server has a huge role to play in this. The time elapsed between when a Player's move arrives at the Server (in the form of an incoming message) and when the Server is ready with the response (in the form of an outgoing message), is an important factor to determine the overall liveness of the Server and how a Player perceives a Server's quickfootedness. This elapsed time per incoming/outgoing pair, is the latency of the Server. One of the aims of a Server's design is to keep latency to a minimum. Of course, the Player who makes the move may still perceive greater latency due to factors like network bandwidth or network congestion etc., but those are out of Server's control anyway.

Minimization of latency is half the story. The Server needs to respond not only to the Player who makes the move but also to the opponent(s) and fellow Players and importantly, to observers. One can easily see that one incoming message can easily give rise to a number of outgoing messages. In a battle game, a pistol shot taken by a Player is considered a move. When this move reaches the Server, the latter processes it using the state of the game viz., the latest position of the Player who was shot at, the shooter's adversary. In either of the cases where the adversary is hit by the bullet or ducks it, the Server needs to send a response to the shooter, the adversary and a number of fencesitters who are observing the game. One also needs to consider the responses which the various components of the Server need to share between themselves, all originating out of the move by the shooter. For session, every hit and miss may have to be sent to a continuously running ScoreBoard component. How many such responses – more accurately, messages – can the Server spew out at a sustained rate is a measure of its throughput.

The importance of latency and throughput is underscored when one takes into account the number of instances of Games that may exist at the same time on a Server. In a reasonably popular Poker site, about 60,000+ peak-time Players and observers are not uncommon. Players expect the latency to be not more than 2 milliseconds (including the network travel time). The throughput of the Server in such cases can be estimated to be upto 100,000 messages per second.

Ideally, a well-behaved Server is supposed to exhibit low latency and high throughput. In an attempt to achieve this, a Server employs measures like carefully chosen Data Structures, minimized explicit locking, smooth coordination between threads, asynchronous message passing between the inner and outer components, optimized message payload, deferred DB-writes (to the extent possible) and handful of others.

##### Server must behave reliably under stress #####

Needless to mention perhaps, that a Game Server must be reliable under heavy load. Even if it has to buckle, it should do so gracefully if possible. Game Servers take measures like Message Throttling to manage sudden surge of simultaneous Players. The Servers also push back requests to initiate new sessions, allowing itself to keep its sanity in tact. Handling disconnection is another important functionality that Server provides. If a Player is disconnected while playing, the Server must capture this incidence and allow associated business logic to execute. In Games where a Player's absence or decision to quit have implications for the outcome of the Session, this is a very important aspect. In Poker for instance, a Player may decide to leave the table while others keep playing and rejoin later. The Server must capture the duration of Player's withdrawal and feed this as input to Server's component which crunches Game's rules. Another relevant case is where In Games where people bet with money, Server has to interact with electronic payment systems, securely and quickly. Audit trails of all such transactions must be maintained too.

##### Server must abiding citizen of a federation #####

OK, here is a bit of digression. Almost always, a Gaming site comes with a package of Services. It is not only about hosting game sessions and executing games' rules. More specifically, it is not only about a Game Server as I have been referring to in previous paragraphs but more than that. Players interact with the Gaming Services in more than one ways, seeking different information. The Gaming Service on its part, updates a connected or non-connected Player with appropriate information from time to time either immediately or post-facto.To a Player who comes in only to register herself, Gaming Service may may offer her a bunch of free games. To another Player who comes in simply to watch what is going on, the Gaming Service offers a Scorecard / Commentary / Lobby view of the goings-on. Some Players may opt to see if her score in a particular type of game has been bettered by a Twitter friend. In fact, Players may want to be notified whenever theirs scores are bettered by their Twitter or Facebook friends! These are just a few of the functionality other than playing games that Players expect from a Gaming Service!

It is obvious that a Game Server cannot serve all the Players with all the facilities that are on the offer. Other specialized Servers (or components, as applicable) have to exist. A Gaming Service is essentially a federation of many such Servers. For example, a Lobby Server's job is to hold the running and past scores, keep track of logged-in Players, collect a Player's history of sessions, so on and so forth. For a Player who has just finished session and scored highest so far, the Gaming Service may need to inform her friends on Facebook. A Facebook Posting Server has to encapsulate all the complexities of connecting to and posting at Facebook. It has to interact with Game Server, Lobby Server and a Database Server to do its job. Keeping a live Leaderboard is another facility that Game Services commonly host. For all these to happen, all these Servers (or components) have to interact continuously and in near-real-time. It follows that these Servers are not only responding to Players' requests but also are exchanging information between themselves, continuously. This adds to Servers' activities, resulting in bigger load on them. 

##### Server must feed precise and timely data to Analytics unit #####

Gaming Services distinguish themselves by offering personalized services to Players. Players are finicky and holding on Players is a constant challenge for the Gaming Services. Keeping track of what a Player usually does, how she behaves and what she prefers to do after she logs in and during the time she is logged in, is extremely important for the Gaming Service providers to know. This data help them to personalize offerings to Players, provide new games and retire unpopular games, not to mention the opportunities to place strategic advertisements. Of course, analysing the data is the job of specialized components and services (loosely called *Analytics*). Feeding the Analytics components with right and timely data is a job of Game Server, Lobby Server and the likes. In a majority of cases, this may simply mean that Game Servers, Lobby Servers (and others as well) write a detailed log to a common log-repository in an uniform manner. But, in some other cases, this may mean that the Game and Lobby and their fellow Servers have to generate a conitnuous stream of distinct, concise and meaningful pieces of data for immediate or later consumption by Analytics components.

This, by no means, is a complete list. Moreover, all the issues and techniques of replication, availability and scalabilty apply to multiuplayer game services as much as they do to any other such Internet-based services. I have not gone deep into those because they are not specific to domain of games.


I hope that this post has given a good ringside view of what goes on if we lift the lid of the world of multiplayer games. Next time, you take fancy towards a fascinating game and invite your friend to play a hand, remember the stuff that you don't see is doing much more crankwork than the stuff you do see! 












