# Hopzone - A full-stack 2D Multiplayer game made in P5.js

## Table Of Contents

1. [Project Description](#project-description)
2. [Research Question](#research-question)
3. [Requirements](#requirements)
4. [Technical Research](#technical-research)
5. [Backend](#backend)
6. [Frontend](#frontend)
7. [Installation & Usage](#installation)
8. [Sources](#sources)
9. [Milestones](#milestones)



<a name="project-description">

## :memo: Project Description
</a>

### Game
Hopzone is a 2D multiplayer platformer game. It is a browsergame where players hop platforms in space. The goal of the game is to successfully jump platforms, dodge monsters and survive the longest!

### Project
This project is a student research project of the bachelor **[Multimedia & Creative Technologies](https://mct.be/)**. As part of this module we were given the assignment to research a topic that hasn't been covered in the learning program. The application will be a full-stack web application.

<a name="research-question">

## :question: Research Question
</a>
At the start of this project we had to choose a topic we wanted to research and formulate a research question about this topic.

:information_source: **What are the possibilities of a multiplayer 2D game based on Processing(P5.js)?**


### Sub-Questions

To research this topic, the main research question has to be divided into multiple sub-questions. These questions will guide me to find an answer to the main research question.

- Which backend framework is the most suitable for this application?
- In what ways can 2D level data be stored? Which way is the best for this application?
- How do you keep track of the state of an active game session?
- What ways are there to realise realtime multiplayer?
- What is a good workflow in P5?

<a name="requirements">

## :heavy_check_mark: Requirements
</a>

These are the requirements to conclude I have delivered a **successful** project. 


---
The finished project has to be a working multiplayer browsergame. A platformer game where a player has to jump on platforms to get as high as possible, while also dodging obstacles.

### Guest users
- Play with other players
- Play single player

### Logged in user
- See history
- High scores

### Nice to have
- Creating your own level (optional)

### The Game
- Platform generation algorithm
- Moving platforms
- Boosted platforms
- Obstacles

<a name="technical-research">

## :mag: Technical Research
</a>

In this researchproject I will investigate what the possibilities are when creating a multiplayer game in the browser based on the javascript library of Processing (P5.js). 

### Data storage
It will be important to find a good way to store and read game data. What ways are there and which one is most suitable?

### Communication server-client
- How are running sessions created?
- How do we keep track of the state of an active game session?
- How do we send the game state to the client?
- Security?

### Game design
- Theme of the game?
- What assets to create?

<a name="backend">

## :1234: Backend
</a>

### Architecture
To realize multiplayer, a solid game engine has to be created. There are 2 main ways to realize realtime apps [1]: 
- Peer 2 Peer
- Server - client

After evaluating the pros and cons of these 2 strategies, I settled on the **server-client** strategy. This strategy allows the game to be centralized in a single server. It also makes the game independent of players to host games. 


![](https://i.imgur.com/Oy3nKTn.png)

> This diagram shows the architecture of both the server and client side part of the game. It is inspired by Michał Męciński's model who also tackled a similar problem. [2]
> 
The server is made in **Express**, a **Node js**  web application framework, with the main communication being handled by Socket.io. The Node.js framework was deliberately chosen for easy syncing between client/server code, as they are both written in **TypeScript**. A more perfomant web framework like **ASP .NET Core** and its multithreading capabilities with **SignalR** as websocket handler would have also been possible, but this would really have slowed down the developing process of the game.

Let's talk about the individual components of the system. I will focus on the server part here. 

- **Database**: The central storage unit. The database holds and persists the state of all active game rooms. For data storing, **MongoDB** is used. This choice was made because MongoDB is a document-based database, and not bound to a database structure. This allows for easily altering how a game should look structure wise.
- **Server**: The server is the central component of the system. It handles new client connections, security, logging and other central tasks. It is responsible for creating and destroying games, and also for persisting data to the database.
- **Game Controller**: This object exists for each active game lobby/session. It holds the game state and is responsible for altering this state when receiving updates from the worker thread. It is also responsible to sending out the state of the game to each of its connected clients.
- **Worker Thread**: Because there are many calculations to be made when a game starts, this computing load is decoupled from the game controller in a worker thread. The thread receives information about player inputs while also receiving the previous state of the game. It performs calculations on this state and sends this updated state back to the game controller. This seems the most ideal solution to this problem to decouple this load from the main process.[3]
- **Player Controller**: For each connected client there is a player controller on the server side. It receives messages from the client, and based on these messages performs an action in the game on behalf of the player.

### The Game State

The game state is stored and modified using an **object oriented** approach. A single game's state looks like this

```typescript=
interface Game {
    players: PlayerObject[]
    platforms: Platform[]
    movingPlatforms: MovingPlatform[]
    boostedPlatforms: BoostedPlatform[]
    enemies: Enemy[]
    alivePlayers: number
    deadPlayers: number
}
```
Every object inside the game state inherits from the same base class **GameObject**. This class holds basic properties like their x and y coordinates, width and height, aswell as methods for detecting intersections with other game objects (hit detection).

When a game is started, the worker thread is responsible for updating this game state and sending it over to the frontend.

### Security
For authorization, Google's **Firebase** is used. This choice was made because it is easy to use and integrate. Middleware is added to the socket server to perform authorization on each socket connection. To authorize users, they have to provide a valid token with their connection.

<a name="frontend">

## :computer: Frontend 
</a>

The frontend is built in the Next.js framework. 

### Styling
For quick development, the web application is styled using the CSS framework **TailwindCSS**.

### Auth
The application uses **Google's Firebase** for authentication.

### Communcation Backend
The communication with the backend happens over the WebSocket protocol. This allows for a bidirectional communication between server and client. the Socket.io library is used for handling this connection. The client listens to different events from the backend such as game state updates, while also sending out messages to the backend such as when a player wants to move.

TypeScript was used to its advantage by having a global enum containing all possible socket messages. Each message is prefixed by either `b2f_` (backend to frontend) of `f2b_` (frontend to backend), to easily distinguish from where the message originates.

```typescript=
enum SocketMessages {
  connectionFailed = 'connect_error',
  connectionSuccess = 'connect',
  activeRooms = 'b2f_gamerooms',
  lobbyInfo = 'b2f_lobby',
  joinLobby = 'f2b_joinLobby',
  leaveLobby = 'f2b_leaveLobby',
  gameState = 'b2f_gameState',
  moveLeft = 'f2b_moveLeft',
  moveRight = 'f2b_moveRight',
  stopMoving = 'f2b_stopMoving',
  gameLoading = 'b2f_gameLoading',
  getScoreboard = 'f2b_scoreboard',
  scoreboard = 'b2f_scoreboard',
  newLobby = 'f2b_newLobby',
  startGame = 'f2b_startGame',
  restartGame = 'f2b_restartGame'
}
```

### The P5 Instance

The game itself is rendered using the javascript library from processing (P5.js) [4]. The main purpose of processing is for non experienced programmers to learn to code in a visual context. It allows for quickly and easily creating visuals in a programming context. To use the P5 context within our Next.js application, the **react-p5** npm package is used [5]. This package gives us access to the P5 instance of our application.

For this project, P5.js is used for rendering the game. When a game is started, it creates a canvas element where the game's state is rendered. An object oriented approach was taken to define every single game object. The P5 instance receives the game state from the server, and renders every game object inside this state. A single game's state looks like this. The models are mapped **one to one** with the models used on the server side.



### Secrets
The frontend requires some secret files to work correctly such as configuration for Firebase or communication with the backend.
#### .env
```
NEXT_PUBLIC_FB_APIKEY=****
NEXT_PUBLIC_FB_AUTHDOMAIN=****
NEXT_PUBLIC_FB_PROJECTID=****
NEXT_PUBLIC_FB_STORAGEBUCKET=****
NEXT_PUBLIC_FB_MESSAGINGSENDERID=****
NEXT_PUBLIC_FB_APPID=****
NEXT_PUBLIC_BACKEND=URL TO THE BACKEND SOCKET SERVER
```


<a name="installation">

## :wrench: Installation & Usage
</a>

Please refer to the installation and user manual for setting up this project to use for yourself.

- [Hopzone - Installation Manual](Manuals/Installation_Manual.md)
- [Hopzone - User Manual](Manuals/User_Manual.md)

<a name="project-timeline">

## :hourglass: Project timeline
</a>

![](https://i.imgur.com/RB820q3.png)

<a name="sources">

## Sources
</a>

[1]S. Neelakantam, “Building a realtime multiplayer browser game in less than a day — Part 3/4,” The Startup, Mar. 18, 2021. https://medium.com/swlh/building-a-realtime-multiplayer-browser-game-in-less-than-a-day-part-3-4-ede95eb924a0 (accessed Dec. 22, 2021).


[2] M. Męciński, “Architecture of a Node.js multiplayer game,” Medium, Nov. 15, 2018. https://medium.com/@MichalMecinski/architecture-of-a-node-js-multiplayer-game-a9365356cb9 (accessed Jan. 04, 2022).

[3]A. lal, “Single thread vs child process vs worker threads vs cluster in nodejs.” https://alvinlal.netlify.app/blog/single-thread-vs-child-process-vs-worker-threads-vs-cluster-in-nodejs (accessed Jan. 10, 2022).

[4]“home | p5.js.” https://p5js.org/ (accessed Jan. 31, 2022).

[5]‘react-p5’, npm. https://www.npmjs.com/package/react-p5 (accessed Jan. 11, 2022).


<a name="milestones">

## :checkered_flag: Milestones
</a>

### 31/12
- [x] Visual representation in contract plan
- [x] Setting up git repos
- [x] Research on backend framework
- [x] Research on level data storage
### 07/01
- [x] Choosing a frontend framework
- [x] Choosing a backend framework
### 14/01
- [x] App design
- [x] Backend structure
- [x] Core game mechanics
- [x] setup auth
- [x] setup db
### 21/01
- [x] Hooking up frontend with backend
- [x] Creation of sessions
- [x] Joining sessions
- [x] Game design
### 28/01
- [x] Integrate moving platforms
- [x] Integrate boosted platforms
- [x] Bug fixing
- [x] Project docs
- [x] User manual
- [x] Bundling research
- [x] Life long learning - Kepping up with the community
### 31/01
- [ ] 🥳 Project handin

