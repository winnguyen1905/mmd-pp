```mermaid
flowchart LR;

%% === Clients ===
subgraph CL["Clients"]
  CA["Client A"];
  CB["Client B"];
end

%% === Realtime Layer ===
subgraph RT["Realtime Layer"]
  RT0["Room Auth<br/>(Token/Policy)"];
  RT1["Signaling Gateway<br/>(WebSocket)"];
  RT2["STUN/TURN<br/>(coturn)"];
  RT3["PeerConnection<br/>(SRTP/DTLS)"];
  RT4["DataChannel<br/>(Overlay/Measure)"];
  RT5["Snapshot & Metrics Emitter"];
  RT6["Media Server / SFU<br/>(optional)"];
end

%% === AI/CV Layer ===
subgraph AI["AI/CV Layer"]
  AI1["CV Worker<br/>(Diff/Object Detection)"];
end

%% === Offchain Layer ===
subgraph OF["Offchain Layer"]
  OC1["Check-in Orchestrator"];
  OC2["Evidence Store"];
  MQ[(Event Bus / RabbitMQ)];
  DB[(Operational DB / Postgres)];
end

%% Step 1-2 Auth & room setup
CA -- "1. Auth & request room" --> RT0;
CB -- "1'. Auth & join room" --> RT0;
RT0 -- "2. Issue room token + ICE servers" --> CA;
RT0 -- "2'. Issue room token + ICE servers" --> CB;

%% Step 3-8 Signaling (WS)
CA -- "3. SDP Offer (WS)" --> RT1;
RT1 -- "4. Deliver Offer" --> CB;
CB -- "5. SDP Answer (WS)" --> RT1;
RT1 -- "6. Deliver Answer" --> CA;
CA -- "7. ICE candidates" --> RT1;
CB -- "7'. ICE candidates" --> RT1;
RT1 -- "8. Exchange ICE to peers" --> CA;
RT1 -- "8'. Exchange ICE to peers" --> CB;

%% Step 9 Connectivity checks (STUN/TURN)
CA -- "9. STUN/TURN checks" --> RT2;
CB -- "9'. STUN/TURN checks" --> RT2;

%% Step 10-11 Media & DataChannel
CA -- "10. SRTP media (P2P)" --> CB;
CA -- "10*. SRTP via SFU (optional)" --> RT6;
RT6 -- "10*. Forwarded media" --> CB;

CA -- "11. DataChannel open" --> CB;
CB -- "11'. DataChannel open" --> CA;

%% Step 12 Overlay events & snapshots
CA -- "12. Overlay JSON (DataChannel)" --> CB;
CB -- "12'. Overlay JSON (DataChannel)" --> CA;
CA -- "13. Snapshot trigger (REST)" --> RT5;
CB -- "13'. Snapshot trigger (REST)" --> RT5;
RT5 -- "14. Enqueue snapshot job" --> MQ;

%% Step 15-16 AI/Offchain linkage
MQ --> AI1;
AI1 -- "15. Diff/Score (result)" --> OC1;
OC1 --> DB;
OC1 -- "16. Store artifacts" --> OC2;

%% (Optional) tie PC/DataChannel visually
RT3 --> RT4;

%% (Optional) subgraph styling (uncomment if your renderer supports)
%% style RT fill:#0b7285,stroke:#0b7285,color:#fff
%% style AI fill:#0b7285,stroke:#0b7285,color:#fff
%% style OF fill:#0b7285,stroke:#0b7285,color:#fff
