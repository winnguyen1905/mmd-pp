```mermaid
flowchart LR


subgraph RT[Realtime Layer]
RT1[Realtime Gateway\n(WebRTC Signaling/TURN)]
RT2[AR Sync / Overlay Service]
RT3[Session Manager\n(Genie Vision)]
end


subgraph AI[AI/CV Layer]
AI1[CV Worker\n(Diff/Object Detection)]
AI2[Proctoring AI]
end


subgraph OF[Offchain Layer]
OC1[Check‑in Orchestrator\n(Aladin's Eye)]
OC2[Exam Orchestrator\n(Sultan's Seal)]
OC3[Evidence Store\n(S3/IPFS Connector)]
OC4[DID/VC Issuer\n(PRISM)]
OC5[On‑chain Anchor\n(Cardano)]
end


MQ[(Event Bus / RabbitMQ)]
DB[(Operational DB / Postgres)]
IPFS[(IPFS/Arweave)]


%% Client <-> Realtime
C <--> RT1
RT1 --> RT2
RT2 --> RT3


%% Client to Offchain (uploads/exams)
C --> OC1
C --> OC2


%% Realtime emits snapshots/measure -> jobs
RT3 -- snapshots/metrics --> MQ


%% Offchain orchestrators enqueue jobs
OC1 -- diff jobs --> MQ
OC2 -- proctoring jobs --> MQ


%% MQ fanout to AI workers
MQ --> AI1
MQ --> AI2


%% AI results back to orchestrators
AI1 -- diffs/scores --> OC1
AI2 -- proctor signals --> OC2


%% Storage & credentials
OC1 --> OC3
OC3 --> IPFS
OC2 --> OC4
OC4 --> OC5


%% Datastore usage
RT3 --> DB
OC1 --> DB
OC2 --> DB


%% Client fetches results/VC
C --> OC1
C --> OC4


classDef grp fill:#0b7285,stroke:#0b7285,color:#fff;
class RT,AI,OF grp;