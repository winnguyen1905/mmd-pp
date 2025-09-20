```mermaid
graph TB
    %% User Interface Layer
    subgraph "Frontend Components"
        HomePage[Home Page<br/>- Room Creation<br/>- Room Joining]
        RoomPage[Room Page<br/>- Video Grid<br/>- Media Controls<br/>- Chat Panel]
        RoomsPage[Rooms List<br/>- Browse Rooms<br/>- Join Existing]
    end

    %% Core WebRTC Management
    subgraph "WebRTC Core"
        WebRTCManager[WebRTC Manager<br/>- Peer Connections<br/>- Media Stream Management<br/>- Data Channels<br/>- Event Handling]
        SocketClient[Socket Signaling Client<br/>- Socket.IO Connection<br/>- Message Routing<br/>- Room Management]
    end

    %% Media Components
    subgraph "Media Layer"
        LocalStream[Local Media Stream<br/>- Camera Access<br/>- Microphone Access<br/>- Screen Sharing]
        RemoteStreams[Remote Media Streams<br/>- Peer Video/Audio<br/>- Stream Display<br/>- Quality Control]
    end

    %% Signaling Infrastructure
    subgraph "Signaling Server"
        SocketServer[Socket.IO Server<br/>Port: 3001<br/>- Room Management<br/>- User Tracking<br/>- Message Forwarding]
        RoomManager[Room Manager<br/>- User Join/Leave<br/>- Participant Lists<br/>- Room State]
    end

    %% Peer-to-Peer Network (Mesh)
    subgraph "P2P Mesh Network"
        UserA[User A<br/>RTCPeerConnection]
        UserB[User B<br/>RTCPeerConnection]
        UserC[User C<br/>RTCPeerConnection]
        UserD[User D<br/>RTCPeerConnection]
    end

    %% External Services
    subgraph "External Services"
        STUNServer1[STUN Server<br/>stun.l.google.com:19302]
        STUNServer2[STUN Server<br/>stun1.l.google.com:19302]
    end

    %% UI to Core Connections
    HomePage --> WebRTCManager
    RoomPage --> WebRTCManager
    RoomsPage --> SocketClient

    %% Core Internal Connections
    WebRTCManager --> SocketClient
    WebRTCManager --> LocalStream
    WebRTCManager --> RemoteStreams
    SocketClient --> SocketServer

    %% Server Side Connections
    SocketServer --> RoomManager

    %% Peer Connections (Mesh Topology)
    UserA -.->|Direct P2P| UserB
    UserA -.->|Direct P2P| UserC
    UserA -.->|Direct P2P| UserD
    UserB -.->|Direct P2P| UserC
    UserB -.->|Direct P2P| UserD
    UserC -.->|Direct P2P| UserD

    %% Signaling Flow
    UserA -->|Signaling Messages| SocketServer
    UserB -->|Signaling Messages| SocketServer
    UserC -->|Signaling Messages| SocketServer
    UserD -->|Signaling Messages| SocketServer

    %% NAT Traversal
    UserA -.->|ICE/STUN| STUNServer1
    UserB -.->|ICE/STUN| STUNServer1
    UserC -.->|ICE/STUN| STUNServer2
    UserD -.->|ICE/STUN| STUNServer2

    %% Styling
    classDef frontend fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef core fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef media fill:#e8f5e8,stroke:#1b5e20,stroke-width:2px
    classDef server fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef peer fill:#fce4ec,stroke:#880e4f,stroke-width:2px
    classDef external fill:#f1f8e9,stroke:#33691e,stroke-width:2px

    class HomePage,RoomPage,RoomsPage frontend
    class WebRTCManager,SocketClient core
    class LocalStream,RemoteStreams media
    class SocketServer,RoomManager server
    class UserA,UserB,UserC,UserD peer
    class STUNServer1,STUNServer2 external
