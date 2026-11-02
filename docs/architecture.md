# Architecture — Bellatrix AI Coprocessor

## Design decision

The initial proof of concept reserves **one Cortex-A53 core** for an AArch64 inference worker. This is an explicit simplifying assumption; the project must not prematurely solve core scheduling or Rigel coexistence.

## Ownership boundaries

- **Bellatrix:** core launch, firmware image integration, RAM reservation, model transport from existing storage, system lifecycle.
- **bellatrix-ai:** inference engine, deterministic host tests, worker loop, freestanding runtime contract.
- **Shared interface:** fixed-width command/result descriptors and a platform-specific synchronization adapter.

## Candidate data flow

1. Bellatrix reserves physically addressable memory and initializes a versioned command/result region.
2. Bellatrix starts the AI worker on its reserved core using the existing secondary-core mechanism.
3. A model checkpoint is copied or mapped into a reserved region by the platform owner.
4. A client posts a bounded token request and signals the worker.
5. The worker executes inference and publishes a result with appropriate release/acquire ordering.
6. The client polls or receives notification; no synchronous dependency on token latency.

## Proposed protocol (design only)

Commands: INIT, LOAD_MODEL, GENERATE, CANCEL, STATUS.

States: RESET, READY, MODEL_LOADED, BUSY, COMPLETE, ERROR.

Descriptors should contain a protocol version, operation, sequence number, status, buffer offset, buffer length, and capacity. Do not expose C pointers in the wire format. Validate all lengths and address ranges before access. The actual ABI must account for the 68k big-endian environment and ARM64 little-endian execution.

## Memory and cache correctness

The Pi 3B has only 1 GB shared physical RAM. Document the GPU/firmware reservation, Emu68-visible regions, AROS allocations, worker code, model weights, scratch tensors and KV cache before selecting an arena size. A dedicated core does **not** have dedicated RAM.

Use atomic state transitions and memory barriers. Whether cache cleaning/invalidation is required depends on the actual mapping and coherency properties; verify on hardware. No shared data structure may rely on unverified cache coherence.

## Runtime boundary

Start with a llama2.c-style scalar CPU implementation. Replace hosted file access, mmap, heap allocation, terminal I/O, clock and any threading assumptions with a small platform interface. Keep the inference hot path allocation-free. Only introduce NEON after correctness is proven.

## Integration questions requiring source audit

- Where does Bellatrix/Emu68 initialize secondary cores?
- Which cores are used under each current build/boot configuration?
- Where are the linker script and physical RAM map defined?
- How are AArch64 code and data placed and loaded?
- What existing IPC, barriers, cache operations and logging can be reused?
- How can the existing FAT/SD path stage model weights without duplicate drivers?

Do not claim any of these hooks are implemented until code inspection and hardware tests confirm them.
