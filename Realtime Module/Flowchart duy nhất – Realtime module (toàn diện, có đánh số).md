```mermaid
flowchart LR;

%% === Clients ===
subgraph CL["Clients"]
  CA["Client A"];
  CB["Client B"];
  CL["Client Apps"];
end

%% === Realtime Layer ===
subgraph RT["Realtime Layer"]
  RT0["Room Auth<br/>(Token/Policy)"];
  RT1["Signaling Gateway<br/>(WebSocket)"];
  RT2["STUN/TURN<br/>(coturn)"];
  RT3["PeerConnection<br/>(SRTP/DTLS)"];
  RT4a["Control Channel"];
  RT4b["Overlay Channel"];
  RT5["Snapshot & Metrics Emitter"];
  RT6["Media Server / SFU<br/>(optional)"];
  RT7["Server-side Capture<br/>(optional)"];
end

%% === AI/CV Layer ===
subgraph AI["AI/CV Layer"]
  AI1["CV Worker: Diff/Detection"];
end

%% === Offchain Layer ===
subgraph OF["Offchain Layer"]
  OC1["Check-in Orchestrator"];
  OC2["Evidence Store<br/>(S3/IPFS Connector)"];
  MQ[(Event Bus / RabbitMQ)];
  DB[(Operational DB / Postgres)];
end

%% ======== A. Room & Signaling ========
CA -- "A1: Request room" --> RT0;
CB -- "A1': Join room" --> RT0;
RT0 -- "A2: Issue room token + ICE servers" --> CA;
RT0 -- "A2': Issue room token + ICE servers" --> CB;
CA -- "A3: SDP Offer (WS)" --> RT1;
RT1 -- "A4: Deliver Offer" --> CB;
CB -- "A5: SDP Answer (WS)" --> RT1;
RT1 -- "A6: Deliver Answer" --> CA;
CA -- "A7: Trickle ICE (candidates)" --> RT1;
CB -- "A7': Trickle ICE (candidates)" --> RT1;
RT1 -- "A8: Exchange candidates" --> CA;
RT1 -- "A8': Exchange candidates" --> CB;

%% ======== B. Connectivity & Media ========
CA --- "B1: STUN binding/relay?" --- RT2;
CB --- "B1': STUN binding/relay?" --- RT2;
CA <-- "B2: DTLS handshake" --> CB;
CA <-- "B3: SRTP media flow (P2P or via SFU/TURN)" --> CB;
CA <-- "B4: Open DataChannels" --> CB;

%% ======== C. In-call control & overlay ========
CA -- "C1: control msg (mute/swap/screen)" --> RT4a;
CB -- "C1'" --> RT4a;
CA -- "C2: overlay JSON (pointer/measure)" --> RT4b;
CB -- "C2'" --> RT4b;
CA -- "C3: Renegotiate (new track/SSRC)" --> RT1;
RT1 -- "C4: Deliver renegotiation" --> CB;

%% ======== D. Snapshots, metrics, session events ========
CA -- "D1: snapshot trigger (REST)" --> RT5;
CB -- "D1'" --> RT5;
RT5 -- "D2: enqueue snapshot job" --> MQ;
RT6 -- "D3: emit session.created/closed" --> MQ;
RT5 -- "D4: push call stats (RTCP/quality)" --> DB;

%% ======== E. AI/Offchain linkage ========
MQ --> AI1;
AI1 -- "E1: diff/score result" --> OC1;
OC1 -- "E2: store artifacts" --> OC2;
OC1 --> DB;

%% ======== F. Client ↔ Offchain (results) ========
CL -- "F1: fetch reports/links" --> OC1;

%% ======== G. Optional: server-side capture ========
RT7 -- "G1: capture keyframe (if not E2EE media)" --> RT5;

%% Optional subgraph styling (uncomment if supported by your renderer)
%% style RT fill:#0b7285,stroke:#0b7285,color:#fff
%% style AI fill:#0b7285,stroke:#0b7285,color:#fff
%% style OF fill:#0b7285,stroke:#0b7285,color:#fff
