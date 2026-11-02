# Focused research — bare-metal LLM inference

This document intentionally excludes generic bare-metal operating-system tutorials as primary references. Hosted Raspberry Pi tutorials are useful only as performance/build baselines.

## Directly relevant projects

### llama2.c
https://github.com/karpathy/llama2.c

Minimal C implementation with a well-defined forward pass and compact checkpoint format. Its run.c uses hosted services (such as file access, mmap/allocators and standard math), so extraction of a freestanding inference core is necessary. Use a pinned upstream revision and deterministic host tests.

### NIGHTRUN
https://github.com/hardrave/NIGHTRUN

ARM64 LLM inference without Linux, using a Raspberry Pi 5/UEFI environment. Study model execution, quantized kernels, NEON/scalar parity and deterministic validation. Its firmware, board, memory and model assumptions cannot simply be reused on a Pi 3B.

### CLLM
https://github.com/cognisoc/cllm

Bare-metal llama.cpp integration experiment. Valuable as a warning about the runtime and C++ integration surface; do not mistake an API skeleton for demonstrated inference.

### Llama2.pi research
https://web.stanford.edu/class/archive/cs/cs224n/cs224n.1244/final-projects/MatthewDing.pdf

A constrained Raspberry Pi study useful for memory and implementation lessons. Do not represent it as a successful production port.

### GGML / llama.cpp
https://github.com/ggml-org/ggml
https://github.com/ggml-org/llama.cpp

Possible future engine once the basic freestanding worker is proven. Avoid importing all C++ and threading dependencies into the first milestone.

## Model candidates

- TinyStories 15M (native llama2.c checkpoint): primary initial model.
- Smaller TinyStories checkpoint: bring-up/debugging fallback; verify its availability and export format.
- TinyStories 42M: optional memory growth test.
- SmolLM2 135M GGUF: later experiment requiring additional loader/architecture support.

Models: https://huggingface.co/karpathy/tinyllamas

**Verification needed:** pin exact model URL, hash, license, tensor dimensions, runtime allocation footprint and deterministic host output. Weight file size alone is not peak RAM usage.

## Other background (not bare-metal)

- https://wolfpaulus.com/local_llama/
- https://github.com/garyexplains/examples/blob/master/how-to-run-llama-cpp-on-raspberry-pi.md

These are not evidence of a bare-metal llama.cpp port.

## Research conclusion

The shortest credible path is to separate a small Llama forward pass from hosted facilities, prove it on a dedicated AArch64 worker core, then assess whether GGML/llama.cpp offers enough benefit to justify its larger runtime.
