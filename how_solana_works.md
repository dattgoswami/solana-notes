# Solana: How the Protocol Works

## Proof of History (PoH)
Solana's unique approach to consensus leverages a cryptographic technique called Proof of History (PoH). PoH provides a verifiable time source that allows the network to order and timestamp transactions efficiently. By using PoH, Solana eliminates the need for nodes to communicate with each other to agree on the order of transactions, reducing latency and increasing throughput.

PoH creates a decentralized solution by generating a verifiable sequence of time-stamped events. It functions as a high-frequency Verifiable Delay Function (VDF), requiring a specific number of sequential steps to evaluate. This sequence is generated using a pre-image resistant hash function (e.g., SHA256) that continuously hashes over itself, ensuring real time has passed between each step.

### Key Features of PoH:
- **Timestamps:** Creates a historical record proving events occurred at specific moments.
- **Sequential Hashing:** Uses a hash function that cannot be parallelized, ensuring the integrity of the sequence.
- **Data Insertion:** Data can be inserted into the sequence by hashing it with the current state, proving it was created before the next state.
- **Verification:** While the sequence is generated on a single CPU core, its output can be verified in parallel across multiple cores.

## Tower BFT Consensus
Solana's Tower Consensus algorithm enables the network to process transactions in parallel across multiple nodes. This parallel processing capability significantly boosts the network's throughput by allowing transactions to be validated concurrently. Tower Consensus is a key component in Solana's ability to achieve high TPS.

Tower BFT enhances the Practical Byzantine Fault Tolerance (PBFT) by leveraging PoH as a network clock to reduce messaging overhead and latency. PoH creates a verifiable sequence of time-stamped events, enabling nodes to trust timestamps without peer-to-peer communication.

### Key Features of Tower BFT:
- **Network Clock:** PoH acts as a global time source, allowing time-outs in PBFT to be computed and enforced using PoH.
- **Sequential Hashing:** PoH uses a hash function to create a verifiable delay function, recording events in a sequence that guarantees their order and time.
- **Voting Mechanism:** Validators cast votes for PoH hashes with time-outs that double for each predecessor vote, ensuring consistency and minimizing rollback.

## Turbine Block Propagation
Inspired by BitTorrent, Turbine optimizes data propagation to solve the scalability trilemma. It leverages a tree structure to distribute data efficiently, reducing the bandwidth required for transmitting blocks to all validators.

### How Turbine Works:
- **Block Division:** The leader divides a 128 MB block into 64KB packets, resulting in 2,000 packets.
- **Packet Transmission:** Each packet is sent to a different validator.
- **Neighborhood Structure:** Validators retransmit packets to a group of peers, forming a tree of neighborhoods.
- **Efficient Propagation:** In a three-level network with neighborhoods of 200 nodes, data can reach 40,000 validators in approximately 200 milliseconds.

## Gulf Stream - Mempool-less Transaction Forwarding
Gulf Stream addresses transaction issues by pushing caching and forwarding to the network's edge. Validators forward transactions to the upcoming leaders, allowing them to execute transactions ahead of time, reducing confirmation times, and minimizing memory pressure from unconfirmed transactions.

### How Gulf Stream Works:
- **Transaction Signing:** Clients sign transactions referencing a recent block-hash that is fully confirmed.
- **Forwarding:** Once signed, transactions are forwarded to validators, who then forward them to upcoming leaders.
- **Execution:** Validators can execute transactions ahead of time, dropping any that fail.
- **Prioritization:** Leaders prioritize transactions based on the stake weight of the forwarding validator, enhancing network resilience during high load.

## Sealevel Virtual Machine (SVM)
Sealevel is a key component in Solana's parallel transaction processing engine that leverages LLVM (Low-Level Virtual Machine) to optimize and execute BPF (Berkeley Packet Filter) programs efficiently on the Solana blockchain.

Sealevel acts as a bridge between the BPF programs and the underlying hardware, providing a high-performance execution environment. By utilizing LLVM, Sealevel can compile BPF programs into machine code that can be executed directly on the hardware, maximizing performance and efficiency.

Sealevel's integration with LLVM allows developers to write BPF programs in a high-level language like Rust and have them compiled down to optimized machine code for execution on the Solana blockchain. This approach ensures that BPF programs run efficiently, maintaining the scalability and performance of the Solana blockchain.

## Berkeley Packet Filter (BPF)
The Berkeley Packet Filter (BPF) is employed by Solana for executing smart contracts. Originally designed for network packet filtering, BPF has been adapted to support Solana's high-performance requirements. It provides a secure and efficient environment for running programs, with the added benefit of supporting Just-In-Time (JIT) compilation.

BPF serves as Solana's virtual machine for executing smart contracts, supporting 64-bit instructions and various operations, making it suitable for high-performance environments. It is capable of processing 60 million packets per second with hardware offload and 40 million without, illustrating its capacity for massive data processing.

## Pipeline
### Solana’s Four-Stage TPU Pipeline:
1. **Data Fetching:** Fetches data at the kernel level.
2. **Signature Verification:** Verifies signatures at the GPU level.
3. **Banking:** Processes transactions at the CPU level.
4. **Writing:** Writes transactions to disk at the kernel space.

By the time the TPU sends blocks to validators, it has already fetched, verified, and processed the next set of transactions. Validators run two pipelined processes: the TPU in leader mode to create ledger entries and the Transaction Validation Unit (TVU) in validator mode to validate entries.

Signature verification, a significant bottleneck, is offloaded to the GPU. Additional bottlenecks, such as network driver interactions and smart contract data dependencies, are managed to optimize concurrency. The Solana TPU can handle 50,000 transactions simultaneously, achievable with a computer costing under $5,000.

## Cloudbreak - Horizontally Scaled State Architecture
Cloudbreak addresses these challenges by organizing the accounts database to allow concurrent reads and writes across multiple threads and SSDs, thus fully utilizing the hardware capabilities.

### Key Components:
- **Memory-Mapped Files:** Files are mapped into the virtual address space, behaving like memory, but limited by disk size rather than RAM. This enables efficient use of SSDs for storing account data.
- **Sequential Operations:** By structuring data to maximize sequential access, Cloudbreak takes advantage of CPU prefetching and operating system efficiencies in handling sequential page faults.
- **Data Structure:**
  - Indexes of accounts and forks are stored in RAM.
  - Accounts are stored in memory-mapped files up to 4MB in size, each representing a single proposed fork and distributed across multiple SSDs.
  - Copy-on-write semantics ensure that updates are sequentially written and horizontally scaled.
  - The index is updated after each write.

### Performance:
- **Sequential Writes and Horizontal Scaling:** Account updates are sequentially written and horizontally scaled across multiple SSDs, ensuring efficient concurrent transaction processing.
- **Garbage Collection:** As forks are finalized and accounts updated, old invalid accounts are removed, optimizing memory usage.
- **Merkle Root Computation:** This can be efficiently done with horizontally scaled sequential reads across SSDs.

## Archivers
Archivers are nodes responsible for storing and maintaining large amounts of blockchain data. They enable a decentralized approach to managing the vast data generated by the network, leveraging Proof of Replication (PoRep) for security and efficiency.

### Data Storage Challenge:
Solana’s network can generate up to 4 petabytes of data annually. Storing all this data on every node would centralize the network. Instead, Solana uses PoRep to distribute the ledger across millions of replicator nodes globally, which have minimal hardware requirements and are not consensus participants.

### How Archivers Work:
- **Data Allocation:** Archivers signal available storage space. The network then divides the ledger history into pieces and assigns them to archivers based on replication rate and fault tolerance.
- **Data Storage:** Archivers download their assigned data from consensus validators.
- **Proof of Replication (PoRep):** Archivers must periodically prove they store the data by completing a PoRep challenge, earning rewards for successful proofs.

References:
https://medium.com/solana-labs/proof-of-history-a-clock-for-blockchain-cf47a61a9274

https://medium.com/solana-labs/tower-bft-solanas-high-performance-implementation-of-pbft-464725911e79

https://medium.com/solana-labs/gulf-stream-solanas-mempool-less-transaction-forwarding-protocol-d342e72186ad

https://medium.com/solana-labs/turbine-solanas-block-propagation-protocol-solves-the-scalability-trilemma-2ddba46a51db

https://medium.com/solana-labs/cloudbreak-solanas-horizontally-scaled-state-architecture-9a86679dcbb1

https://medium.com/solana-labs/replicators-solanas-solution-to-petabytes-of-blockchain-data-storage-ef79db053fa1
