# Xenovectra

Virtual Hardware GPU Emulator that runs as a VM appliance or native desktop application and emulates GPU‑style, data‑parallel compute by composing ordinary hardware into a configurable virtual GPU fabric.

---

### Summary

**Xenovectra** maps shader kernels and data pipelines onto CPU threads, host RAM as VRAM emulation, memory‑mapped files, external HDD/SSD, and USB kernel caches. It provides a research and prototype platform for systems without dedicated GPUs, enabling experimentation with latency, bandwidth, and resource‑aware scheduling. The project includes a small shader DSL, an IR path, a scheduler, component plugins, metrics, and optional visualization.

---

### Key Features

- **VRAM Emulation** using host RAM and memory‑mapped files to simulate limited GPU memory and paging.  
- **Component Abstraction** that maps physical devices to virtual GPU subsystems: CPU threads as ALUs, HDD/SSD as high‑latency compute tiles, USB as persistent kernel cache.  
- **Shader DSL and IR** a compact GLSL/CUDA‑inspired language compiled to a simple intermediate representation for interpretation or JIT compilation.  
- **Configurable Virtual Bus** that models bandwidth, latency, and contention to simulate PCIe and external interconnects.  
- **Plugin System** for adding new device types such as NVMe, network nodes, or remote storage.  
- **Metrics and Tracing** with jobs per second, virtual bandwidth, stage latencies, and component utilization exposed via CLI and a local HTTP endpoint.  
- **Example Workloads** including image filters, tiled matrix multiply, and a toy renderer for validation and benchmarking.

---

### Architecture Overview

**Core Engine**  
A low‑level runtime responsible for scheduling, task dispatch, and component orchestration. The core exposes a Component interface for plugins and a VRAM allocator for buffer management.

**Scheduler and Runtime**  
A resource‑aware scheduler partitions workloads into tiles, places tasks based on data locality and device characteristics, and uses work‑stealing and async I/O to maximize utilization.

**Storage and Buffer Backing**  
Large buffers can be backed by memory‑mapped files to simulate VRAM paging and persistence. Eviction policies such as LRU or LFU manage residency and flushing.

**Virtual Bus**  
Transfers are modeled as timestamped messages with configurable bandwidth and base latency. The bus simulates contention and allows experiments with different interconnect profiles.

**Tooling and UI**  
A CLI for automation and an optional GUI for pipeline visualization, buffer inspection, and live metrics. The GUI consumes a local HTTP API exposed by the core engine.

---

### Shader DSL and Toolchain

**Design Goals**  
A small, data‑parallel language inspired by GLSL and OpenCL that expresses elementwise and tiled operations over arrays, images, and tensors.

**Workflow**  
1. Write kernel in the DSL.  
2. Compile to IR with the `vgpucc` tool.  
3. Load IR into the runtime and dispatch with grid and tile parameters.  
4. Optionally JIT compile IR to native code for performance.

**Runtime API**  
Load kernels, allocate VRAM buffers, dispatch kernels with grid/block parameters, and collect results. The runtime exposes hooks for profiling and tracing.

---

### Plugins and Extensibility

**Plugin Interface**  
Components implement a simple trait or interface with methods for initialization, task submission, polling, and metrics reporting. Plugins register at startup or via configuration.

**Example Plugins**  
- HDD Plugin simulates long‑latency, high‑throughput compute tiles.  
- USB Cache Plugin scans mounted devices for kernel repositories and verifies integrity.  
- NVMe Plugin provides a high‑speed cache tier.  
- Network Plugin exposes remote nodes for distributed compute experiments.

**Extending the System**  
Add new device types by implementing the Component interface and registering the plugin. Plugins can expose custom metrics and configuration options.

---

### Getting Started

**Prerequisites**  
Rust toolchain for the core engine, CMake and a C++ compiler for memory‑mapped buffer support, Python for DSL tooling and examples.

**Quickstart Steps**  
1. Build the core engine.  
2. Build the memory‑mapped buffer module.  
3. Set up the Python virtual environment and install DSL tooling.  
4. Generate an example input, compile a kernel, and run a job using the provided dispatch script.  

**Example Workflows**  
- Run an image blur kernel tiled across CPU threads and HDD tiles.  
- Benchmark matrix multiply with different virtual bus profiles.  
- Experiment with eviction policies and measure effective memory bandwidth.

---

### Development and Contribution

**Repository Layout**  
- `core` for the runtime and scheduler.  
- `buffer` for memory‑mapped backing and eviction.  
- `dsl` for the language, compiler, and interpreter.  
- `plugins` for device adapters.  
- `examples` for workloads and tests.  
- `docs` for architecture and design notes.

**Testing and CI**  
Unit tests for allocator, scheduler, and DSL parser. Integration tests for end‑to‑end jobs. CI pipelines build core components and run basic examples.

**How to Contribute**  
Fork the repository, implement features or fixes on a branch, add tests and documentation, and open a pull request. Follow the coding style and include benchmarks for performance changes.

---

### Use Cases and Limitations

**Use Cases**  
- Educational platform for GPU architecture and scheduling research.  
- Fallback compute on GPU‑less systems for prototyping algorithms.  
- Experimentation with heterogeneous resource orchestration and interconnect modeling.

**Limitations**  
Performance will not match dedicated GPUs. I/O and synchronization overheads are primary bottlenecks. The system is intended for prototyping, research, and education rather than production rendering at scale.

---

### License and Governance
MIT
---

### Contact and Resources

See the docs directory for architecture diagrams, API references, and developer guides. Use the issue tracker for bugs and feature requests and the discussions area for design conversations and experiments.
