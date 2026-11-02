# Bellatrix AI Coprocessor

A bare-metal AI coprocessor for [Bellatrix](https://github.com/JJDSNT/bellatrix), running local LLM inference on a dedicated ARM64 core.

> **Status:** architectural proof of concept (PoC). No working inference or Bellatrix integration is claimed.

## Goal

Demonstrate that a Raspberry Pi 3 Model B can run the Emu68/AROS environment alongside a native ARM64 LLM inference worker on a dedicated Cortex-A53 core. The goal is architectural validation, **not** practical chat performance.

## Initial constraints

- Target: Raspberry Pi 3B, four Cortex-A53 cores, 1 GB shared RAM.
- One core is reserved for AI in the first PoC. Dynamic scheduling and coexistence with all Rigel configurations are future work.
- Bare-metal execution: no Linux, UEFI runtime, or second operating system.
- Avoid blocking Emu68/AROS; requests and results are asynchronous.
- Keep the inference engine independent of Bellatrix-specific boot, storage and IPC details.
- No requirement for GPU acceleration, usable tokens/second, or general-purpose model quality.

## First demonstration

1. Boot Bellatrix with a dedicated ARM64 worker core.
2. Reserve a bounded memory arena for weights, activations and KV cache, outside memory exposed to the emulated system.
3. Run a tiny Transformer forward pass and compare logits/tokens against a host reference.
4. Generate at least one validated token while the Bellatrix environment remains running.
5. Exchange a command and result through a minimal shared-memory protocol.

**First model candidate:** TinyStories 15M in the native [llama2.c](https://github.com/karpathy/llama2.c) checkpoint format. A smaller TinyStories checkpoint may be used for bring-up. GGUF and SmolLM2 135M are possible later experiments, not initial requirements.

## Proposed architecture

```text
Bellatrix / Emu68 / AROS
        |
  Bellatrix-side adapter
        |
 shared-memory command/result queues
        |
 dedicated ARM64 AI worker
        |
 freestanding inference adapter
        |
 llama2.c-inspired Transformer inference
        |
 reserved model + scratch + KV-cache arena
```

The exact secondary-core startup path, physical-memory ownership, cache coherency strategy, linker layout, and Bellatrix build hooks **must be verified against current Bellatrix/Emu68 source** before implementation.

The Bellatrix adapter owns platform-specific startup, storage and memory reservation. This repository owns the inference worker, freestanding runtime interfaces, host reference tests, and a platform-neutral message contract. Do not duplicate SD/FAT or USB drivers.

## Research starting points

| Project | Why it matters | Caveat |
| --- | --- | --- |
| [llama2.c](https://github.com/karpathy/llama2.c) | Compact C Transformer inference; straightforward checkpoint and host reference | Uses hosted C facilities; not bare-metal out of the box |
| [TinyLlamas](https://huggingface.co/karpathy/tinyllamas) | Very small checkpoints for repeatable bring-up | Check model format and RAM usage |
| [NIGHTRUN](https://github.com/hardrave/NIGHTRUN) | ARM64 LLM inference without Linux; NEON and reference validation ideas | Raspberry Pi 5 / UEFI assumptions are not transferable as-is |
| [CLLM](https://github.com/cognisoc/cllm) | Documents a llama.cpp-oriented bare-metal integration approach | Do not assume functional inference is implemented |
| [llama.cpp](https://github.com/ggml-org/llama.cpp) / [GGML](https://github.com/ggml-org/ggml) | Future backend and GGUF compatibility | Larger C++ runtime and threading surface |
| [Llama2.pi study](https://web.stanford.edu/class/archive/cs/cs224n/cs224n.1244/final-projects/MatthewDing.pdf) | Lessons from constrained Raspberry Pi memory | Research experiment, not a proven ready-to-use port |
| [Local Llama](https://wolfpaulus.com/local_llama/) / [Gary Explains Pi tutorial](https://github.com/garyexplains/examples/blob/master/how-to-run-llama-cpp-on-raspberry-pi.md) | Hosted Raspberry Pi baseline | Linux examples, **not** bare-metal references |

## Roadmap

- **BAI-01 — Source audit:** identify existing Bellatrix/Emu68 secondary-core launch, linker sections, memory ownership and synchronization; record exact paths and constraints.
- **BAI-02 — Worker bring-up:** execute a freestanding AArch64 worker on the dedicated core; observe heartbeat without disrupting Bellatrix.
- **BAI-03 — Memory and IPC:** fixed arena and versioned request/result queues; specify endianness, atomic ordering, cache maintenance, buffer ownership and error handling.
- **BAI-04 — Host inference baseline:** pin an upstream llama2.c revision and model checksum; capture deterministic reference logits/tokens.
- **BAI-05 — Freestanding inference:** isolate forward pass from file I/O, mmap, allocation, terminal and hosted runtime; start with scalar math.
- **BAI-06 — Model loading:** load or pre-stage checkpoint using Bellatrix-owned storage facilities, with explicit memory accounting.
- **BAI-07 — Integrated demonstration:** generate at least one validated token concurrently with Bellatrix.
- **BAI-08 — Optional optimization:** investigate AArch64 NEON, quantization, GGML/llama.cpp and alternative scheduling only after correctness.

## Engineering acceptance criteria

- All experimental results identify board, firmware revision, model checksum, memory footprint and build configuration.
- Host and bare-metal runs use the same weights and deterministic decoding; compare logits with stated numerical tolerance and generated token IDs exactly.
- The worker never accesses AROS-owned RAM or peripherals without an explicit contract.
- Cross-core messages use fixed-width types and explicit offsets/sizes rather than raw cross-address-space pointers.
- No unbounded allocations or busy-wait loops on the Emu68/AROS side.
- Failures can be reported without taking down the emulated environment wherever technically possible.

## Scope boundaries

This project is a **software-defined AI coprocessor**, not a physical NPU. A dedicated CPU core is a deliberate PoC simplification, not a permanent scheduling requirement. A complete llama.cpp port, production chatbot, Rigel core remapping, AROS GUI, network API and GPU acceleration are out of scope for the first milestone.

## Licensing

Licenses for any incorporated third-party source and model weights must be checked and preserved before vendoring or redistribution. No license for this repository has been selected yet.
