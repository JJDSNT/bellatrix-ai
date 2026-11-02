# Proof-of-concept roadmap

This is an **experimental plan**, not an implementation status report.

| ID | Deliverable | Exit criterion |
| --- | --- | --- |
| BAI-01 | Audit Bellatrix and Emu68 boot/memory sources | File-level map of secondary-core entry, linker, memory, IPC and storage hooks |
| BAI-02 | Dedicated-core worker | Worker heartbeat while Bellatrix continues running |
| BAI-03 | Shared memory and bounded IPC | Command/result round trip with ordering and bounds tests |
| BAI-04 | Reproducible host baseline | Pinned model hash, build revision and expected logits/tokens |
| BAI-05 | Freestanding Llama inference | Scalar forward pass builds without hosted OS calls |
| BAI-06 | Model staging | Checkpoint available in reserved RAM with documented peak usage |
| BAI-07 | Integrated AI demonstration | At least one verified generated token alongside Bellatrix |
| BAI-08 | Optional extensions | NEON, quantization, GGUF, GGML and alternative scheduling |

## Priority and freedom to iterate

The agent may reorder independent research and host-only tasks, but must preserve dependencies and must not silently expand the first milestone to a full llama.cpp port or scheduler rewrite. Keep experimental code isolated and document all assumptions.

## Definition of done for PoC 0.1

- Runs on a physical Raspberry Pi 3B.
- Exactly one core is reserved for inference in the test configuration.
- Bellatrix/Emu68 remains active during a complete inference request.
- Model identity and checksum are recorded.
- At least one generated token is checked against a deterministic host reference.
- Memory map, runtime footprint, test steps, known limitations and failures are documented.

## Explicit non-goals

Useful conversation speed, a full chatbot, GGUF support, model quantization, production-grade error recovery, AROS GUI, GPU acceleration and dynamic CPU scheduling.
