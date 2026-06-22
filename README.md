# TCP Group Chat Application

A real-time group chat system built with **Java Sockets (TCP)** and **JavaFX**, following a strict **Model-View-Controller (MVC)** architecture with full separation of concerns.

---

## Architecture

```
┌──────────────────────────────────────────────────────────┐
│                      TCPServer                           │
│  ┌──────────────┐   ┌──────────────┐   ┌─────────────┐  │
│  │  ChatServer  │◄──│ServerControl-│──►│ ServerView  │  │
│  │ (Model)      │   │ler           │   │ (JavaFX UI) │  │
│  │ ClientHandler│   └──────────────┘   └─────────────┘  │
│  └──────────────┘                                        │
└────────────────────────┬─────────────────────────────────┘
                         │ TCP / Socket
         ┌───────────────┴───────────────┐
         │                               │
┌────────▼───────┐             ┌─────────▼──────┐
│  TCPClient A   │             │  TCPClient B   │
│ ChatClient(M)  │             │ ChatClient(M)  │
│ ClientCtrl(C)  │             │ ClientCtrl(C)  │
│ ClientView(V)  │             │ ClientView(V)  │
└────────────────┘             └────────────────┘
```

### Key Design Decisions
| Concern | Where it lives |
|---|---|
| TCP socket logic | `model/ChatServer`, `model/ClientHandler`, `model/ChatClient` |
| UI rendering | `view/ServerView`, `view/ClientView` |
| Wiring & callbacks | `controller/ServerController`, `controller/ClientController` |
| Config | `server.properties`, `client.properties` (loaded at runtime) |

The Model layer contains **zero JavaFX imports**. The View layer contains **zero socket imports**.

---

## Features

### Client
- **Login overlay** — enter username or leave blank for **read-only mode**
- **Real-time messaging** — send with the Send button or press Enter
- **`allUsers`** command — server responds with all active users
- **`end` / `bye`** — graceful disconnect
- **Online/Offline status** indicator (green/red dot)
- Read-only mode locks the input bar automatically

### Server
- **Thread-per-connection** model using a cached thread pool
- **Broadcasts** every message to all connected clients (with timestamp and sender name)
- **Live user list** with random per-user accent colors
- **Activity log** showing connections, disconnections, and messages
- **Online status** indicator

---

## Project Structure

```
chat-app/
├── TCPServer/
│   ├── pom.xml
│   └── src/main/
│       ├── java/server/
│       │   ├── model/
│       │   │   ├── ChatServer.java        ← Core server logic
│       │   │   ├── ClientHandler.java     ← Per-client thread
│       │   │   └── ServerConfig.java      ← Loads server.properties
│       │   ├── controller/
│       │   │   └── ServerController.java  ← Wires model ↔ view
│       │   └── view/
│       │       ├── ServerApp.java         ← JavaFX entry point
│       │       └── ServerView.java        ← Server UI
│       └── resources/
│           └── server.properties
│
└── TCPClient/
    ├── pom.xml
    └── src/main/
        ├── java/client/
        │   ├── model/
        │   │   ├── ChatClient.java        ← Core client logic
        │   │   └── ClientConfig.java      ← Loads client.properties / CLI args
        │   ├── controller/
        │   │   └── ClientController.java  ← Wires model ↔ view
        │   └── view/
        │       ├── ClientApp.java         ← JavaFX entry point
        │       └── ClientView.java        ← Chat UI
        └── resources/
            └── client.properties
```

---

## Build & Run

### Prerequisites
- Java 17+
- Maven 3.8+

### Build

```bash
# Build server JAR
cd TCPServer
mvn clean package -q

# Build client JAR
cd ../TCPClient
mvn clean package -q
```

### Run

```bash
# Start the server
java -jar TCPServer/target/TCPServer-1.0-SNAPSHOT.jar

# Start one or more clients (CLI args override client.properties)
java -jar TCPClient/target/TCPClient-1.0-SNAPSHOT.jar localhost 3000
java -jar TCPClient/target/TCPClient-1.0-SNAPSHOT.jar localhost 3000
```

Or use Maven directly:

```bash
# Server
cd TCPServer && mvn javafx:run

# Client
cd TCPClient && mvn javafx:run -Djavafx.args="localhost 3000"
```

---

## Configuration

### `server.properties`
```properties
server.port=3000
server.maxConnections=50
```

### `client.properties`
```properties
server.host=localhost
server.port=3000
```

CLI arguments always override the properties file:
```bash
java TCPClient <ServerIPAddress> <PortNumber>
java TCPServer
```

---

## Protocol Summary

| Client sends | Server action |
|---|---|
| `<username>\n` (first line) | Registers user; blank → read-only |
| Any text | Formats as `[HH:mm] username: text` and broadcasts |
| `allUsers` | Replies to that client only with active user list |
| `end` or `bye` | Graceful disconnect; notifies all other clients |

---

## UML Diagrams

### Class Diagram (simplified)

```
ChatServer  1──* ClientHandler
ChatServer ◄── ServerController ──► ServerView
ChatClient ◄── ClientController ──► ClientView
ServerConfig              ClientConfig
```

### Deployment Diagram

```
[Server Machine]                [Client Machine 1..N]
 ┌──────────────┐                ┌─────────────────┐
 │  TCPServer   │◄─── TCP/IP ───►│   TCPClient     │
 │  port: 3000  │                │  host:port cfg  │
 └──────────────┘                └─────────────────┘
```

---

## Notes for Submission Checklist

- [x] Fully functional Maven projects (TCPServer + TCPClient)
- [x] Executable JAR artifacts via `mvn package`
- [x] MVC separation — Model has no JavaFX; View has no socket logic
- [x] Runtime configuration via `.properties` files
- [x] Thread-per-connection server model
- [x] Read-only mode for blank usernames
- [x] `allUsers` command
- [x] Graceful disconnect (`end` / `bye`)
- [x] Online status indicator in client UI
- [x] Coloured user list in server UI
- [x] Activity log in server UI
- [ ] Demo video (3 min walkthrough + live demo)
- [ ] GitHub repository with commit history + README
- [ ] Optional: UML sequence/use-case diagrams
- [ ] Optional: Blog post (Medium / Dev.to)
