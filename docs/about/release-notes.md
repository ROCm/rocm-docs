# ROCm Core SDK {{ ROCM_VERSION }} release notes

These release notes describe notable changes since the previous ROCm release.

- [Release highlights](#release-highlights)
- [AMD hardware support](#amd-hardware-support)
- [Operating system support](#operating-system-support)
- [Installation updates](#installation-updates)
- [Kernel driver and firmware bundle support](#kernel-driver-and-firmware-bundle-support)
- [GPU virtualization support](#gpu-virtualization-support)
- [GPU partitioning support](#gpu-partitioning-support)
- [AI ecosystem support](#ai-ecosystem-support)
- [ROCm Core SDK components](#rocm-core-sdk-components)
- [ROCm breaking changes](#rocm-breaking-changes)
- [ROCm known issues](#rocm-known-issues)
- [ROCm resolved issues](#rocm-resolved-issues)

```{note}
Since ROCm 7.14, ROCm uses [TheRock](https://github.com/ROCm/TheRock) as its build and release system. For more information, see the [transition guide](/about/transition-guide-TheRock).
```

## Release highlights

This release focuses on AI inference, developer tooling, and profiling across AMD Instinct™, Radeon™, and Ryzen™ AI platforms. Highlights include expanded GPU, OS, and virtualization support, new AI framework versions, the addition of new components, new HIP APIs and performance improvements, profiler usability and performance improvements, and updates to math, sparse, and storage libraries.

### Platform and hardware support

This release expands GPU, operating system, and virtualization support.

#### Expanded AMD GPU support

ROCm 10.1.0 adds support for [AMD Radeon AI PRO R9600](https://www.amd.com/en/products/graphics/workstations/radeon-ai-pro/ai-9000-series/amd-radeon-ai-pro-r9600.html) GPUs.

For the complete list of supported AMD hardware, see [AMD hardware support](#amd-hardware-support).

#### Operating system support update

ROCm 10.1.0 adds support for:
* Ubuntu 26.04.1 (kernel: 7.0 [GA]) for AMD Instinct, Radeon, and Ryzen GPUs.
* Ubuntu 24.04.5:
  * kernel: 6.8 [GA] for AMD Instinct GPUs.
  * kernel: 7.0 [HWE] for AMD Radeon and Ryzen GPUs.

The updated support replaces the support for Ubuntu 26.04 and Ubuntu 24.04.4 respectively.

For the full list of supported Linux distributions, see [Operating system support](#operating-system-support).

#### Expanded GPU virtualization support for Instinct and Radeon GPUs

ROCm 10.1.0 adds support for the following virtualization configurations on AMD Instinct and Radeon GPUs:

**Instinct**

* On AMD Instinct MI355X and MI350X:
  * KVM SR-IOV Ubuntu 26.04 host OS with Ubuntu 26.04 guest OS.

* On AMD Instinct MI350P:
  * ESXi SR-IOV VMware ESXi 9.1 host OS and Ubuntu 24.04 guest OS.

* On AMD Instinct MI325X:
  * KVM SR-IOV Ubuntu 26.04 host OS with Ubuntu 26.04 guest OS.
  * KVM SR-IOV RHEL 9.4 host OS with Ubuntu 24.04 and RHEL 9.4 guest OS.

* On AMD Instinct MI300X:
  * KVM SR-IOV Ubuntu 26.04 host OS with Ubuntu 26.04 guest OS.

* On AMD Instinct MI210:
  * KVM SR-IOV Ubuntu 26.04 host OS with Ubuntu 26.04 guest OS.

**Radeon**

* On AMD Radeon PRO V710:
  * KVM SR-IOV Ubuntu 26.04 host OS with Ubuntu 26.04, Ubuntu 24.04, RHEL 10.2, and RHEL 9.6 guest OS.
  * KVM SR-IOV Ubuntu 24.04 host OS with RHEL 10.2 guest OS.

Supported Single Root I/O Virtualization (SR-IOV) configurations require the [AMD GPU Virtualization Driver (GIM) 9.3.0.K](https://github.com/amd/MxGPU-Virtualization/releases/tag/9.3.0.K). For details, see [GPU virtualization support](#gpu-virtualization-support).

#### GPU partitioning support update

GPU partitioning support remains unchanged in this release. For details, see [GPU partitioning support](#gpu-partitioning-support).

### AI inference and frameworks

This release enables support for the following frameworks:

* PyTorch 2.14.0
* JAX 0.11.1
* vLLM 0.29.0
* SGLang 0.5.18
* MIGraphX 2.18

The updated framework support replaces the previous PyTorch 2.11.0, JAX 0.10.0, vLLM 0.27.0, SGLang 0.5.15, and MIGraphX 2.17 support. This release drops support for TensorFlow 2.19.1.

For details, see [AI ecosystem support](#ai-ecosystem-support).

### Runtime and compilers

#### HIP feature highlights

The following are notable enhancements to HIP:

##### HIP supports hipExtHostRegisterCoarseGrained on Windows

HIP now honors the `hipExtHostRegisterCoarseGrained` flag passed to `hipHostRegister` on Windows for APUs with unified memory, giving device kernels correct results for atomic operations on host-pinned memory. Previously, host-registered memory on Windows was always treated as coarse-grain regardless of the flag; fine-grain (uncached) is now the correct default, with coarse-grain available only when explicitly requested.

##### HIP adds host-NUMA virtual memory support

HIP virtual memory management APIs now support `hipMemLocationTypeHostNuma` and `hipMemLocationTypeHostNumaCurrent` in `hipMemCreate`, allocating pinned memory backed by real host memory on a specified NUMA node instead of a GPU-pool substitute. This benefits NUMA-aware workloads, including collective-communication libraries such as RCCL, that need pinned host memory to stay local to a specific NUMA node. The full VMM lifecycle (create, reserve, map, set access, and teardown) is supported for these allocations on Linux systems with NUMA-capable, VMM-capable GPUs.

##### ROCr Runtime extends virtual memory management to host memory

ROCr Runtime virtual memory management APIs (`hsa_amd_vmem_handle_create`, `hsa_amd_vmem_map`, and `hsa_amd_vmem_set_access`) now support CPU memory pools in addition to GPU agents, enabling the full create, reserve, map, set-access, and teardown lifecycle for host-pool virtual memory handles. This includes inter-process sharing of both host-pool and device-pool handles, letting multiple processes map the same GPU-accessible host memory without falling back to workarounds like `/dev/shm`.

For more information, see the [HIP section](#hip-7-16-0) in the ROCm component changelogs.

### Profiling and debugging tools

#### ROCgdb __builtin_verbose_trap improvement

ROCgdb now automatically presents the diagnostic inline frame generated by Clang's `__builtin_verbose_trap` in backtraces, for both AMD64 and AMDGPU targets. Previously, after the trap fired, the reported frame sat outside that inline frame, hiding the category and message strings the built-in carried. Recovering them required manually rewinding the program counter. Backtraces following a verbose trap now show `__clang_trap_msg$<category>$<message>` as frame #0 automatically.

#### ROCprofiler-SDK feature highlights

The following are notable enhancements to ROCprofiler-SDK:

##### Kernel replay feature (Beta)

ROCprofiler-SDK introduces kernel replay, which re-executes each individual dispatch once per counter group within a single application run. Before replaying, the SDK snapshots every tracked coarse-grained device allocation owned by the dispatching agent, plus discovered module-scope device variables, and restores that snapshot between passes so every group observes identical captured inputs.

GPU hardware limits how many performance counters can be collected in a single kernel execution. Previously, application replay worked around this by re-launching the entire application once per counter group, multiplying startup overhead and producing metrics derived from non-simultaneous samples.
With kernel replay, the SDK snapshots tracked device allocations and module-scope variables before each replay and restores them between passes, so every group sees identical inputs. Dispatches that require no replay opt out at no cost.
Kernel replay is exposed as its own callback tracing domain (`ROCPROFILER_CALLBACK_TRACING_KERNEL_REPLAY`) and is available to custom ROCprofiler-SDK tools without going through `rocprofv3`.
In `rocprofv3`, use `--replay-mode kernel` with the `--kernel-replay-beta-enabled` opt-in flag:

```bash
rocprofv3 --pmc SQ_WAVES GRBM_COUNT --pmc GRBM_GUI_ACTIVE --replay-mode kernel --kernel-replay-beta-enabled -- <application>
```
:::{note}
* **Known limitations:** `--replay-mode kernel` requires `--pmc` and the beta opt-in flag, and collects counters only. Cannot be combined with `--att`, PC sampling, or `--spm`, and `rocprofv3` rejects those combinations — counter groups are the only thing that changes between passes, so any other service would stay enabled across all passes and report every kernel once per pass. There is no pass-count flag so the number of passes is the number of `--pmc` groups collectable on the dispatch’s GPU agent.

* **Beta Risk:** Applications whose kernels depend on state outside tracked device allocations and module-scope device variables might not replay faithfully; prefer application replay in those cases.
:::

##### HIP event tracing

ROCprofiler-SDK and `rocprofv3` add GPU-side barrier tracing for HIP events. `hipEventRecord` and `hipStreamWaitEvent` are now traced as first-class activity, making it possible to see where a stream is actually blocked waiting on an event rather than inferring it from surrounding API calls.
The SDK exposes new tracing kinds `ROCPROFILER_CALLBACK_TRACING_HIP_EVENT` and `ROCPROFILER_BUFFER_TRACING_HIP_EVENT`, with the operation enum `rocprofiler_hip_event_operation_t` (`RECORD`, `WAIT`).

In `rocprofv3`, HIP event tracing is enabled with the `--hip-event-trace` flag, and is automatically enabled by `--hip-trace` and `--hip-runtime-trace`. Records are emitted to the JSON and rocpd output formats; use `rocpd convert` to produce CSV or Perfetto output from the database. The rocpd schema is bumped to 3.0.4 with a new `rocpd_hip_event` table and a `hip_events` data view.

##### On-demand tool configuration and initialization

ROCprofiler-SDK now supports enabling and disabling profiler tools at any point during application execution, not just at startup or within a narrow initialization window.
The profilers built into AI frameworks are the primary beneficiaries. PyTorch’s Kineto backend and Triton’s Proton profiler activate mid-session, after the interpreter, HIP, HSA, and often the model itself have already initialized. Previously, late attachment was partially supported but tools could not reliably configure after another tool had already done so, and runtime interception could not be undone. Both restrictions are now lifted. Four changes enable this:

* **Repeated and late configuration:** A tool can now call `rocprofiler_force_configure` after one or more other tools have already configured ROCprofiler-SDK, rather than being rejected because configuration was already complete.
* **Reversible runtime interception:** All intercepted runtime domains (HIP, HSA, ROCTx/marker, RCCL, rocDecode, rocJpeg, rocSHMEM, and hipFile) now support a table restore operation, so interception can be installed, removed, and reinstalled while the application runs.
* **Symbol visibility for re-propagation:** `librocprofiler-register.so` is now opened explicitly with `RTLD_GLOBAL | RTLD_NOLOAD`, and `rocprofiler_register_invoke_all_registrations` is resolved from that handle instead of the global symbol namespace. Late registration therefore works in processes where `rocprofiler-register` was loaded privately. This happens when it arrives as a dependency of a Python extension module, the typical loading path for AI framework profilers.
* **Pointer-stable internal registries:** The buffer and internal task-group registries move to chunked, pointer-stable containers with pre-reserved capacity, so references held by already-configured tools remain valid when a new tool registers mid-run and grows those registries.

:::{note}
**Known limitation:** During another tool’s initialization, there is a brief window where previously configured tools might not receive records from application background threads.
:::

##### Signal handling update

ROCprofiler-SDK now runs signal-triggered finalization on a dedicated worker thread instead of inside the signal handler itself, where the allocation, locking, and I/O it requires are not async-signal-safe. A second signal arriving during the flush is held until the flush completes, and the process then terminates normally. `rocprofv3` also preserves the first signal disposition the application installed, including `SIG_IGN`, so the application sees its own signal behavior as though the profiler were not present, and it re-raises the signal only after child processes are reaped.
`SIGABRT` can’t be deferred, so that path waits for the flush to finish before returning into `abort()`. The wait is bounded by `ROCPROF_ABORT_FLUSH_TIMEOUT_SECONDS` (default 10 seconds; a value ≤ 0 waits indefinitely).
Finalizing inside the signal handler previously caused three failure modes:
* A hang when the signal interrupted a thread already holding a lock the flush needed.
* Truncated output when a second signal arrived mid-flush (a chained application handler re-raising, or a second Ctrl+C).
* An unkillable process that left GPU child processes running and required a manual kill.

Re-raising before reaping also previously deadlocked multi-process applications, such as tensor-parallel inference servers whose workers exit only when told to by the main process.
Opting out: Applications that coordinate their own shutdown can disable the handlers entirely with `disable-signal-handlers` and rely on the normal `atexit` path instead.

##### Dynamic live attach improvements

`rocprofv3` attach (`rocattach`) is now more robust in deployments where the host and target don’t share an identical view of the filesystem:

* Falls back to GNU Build ID validation when device and inode don’t match, so a library that’s the same build but reached through a different mount or path is still recognized.
* Recognizes SONAMEs and equivalent library paths instead of requiring an exact path match.
* No longer propagates the host’s `rocprofiler-register` library path into the target process, which previously pointed the target at a library that might not exist or match on its side.

Combined with the container-aware symbol resolution delivered in ROCm 10.0.0, these changes let host-to-container and mismatched-path attach work without manually copying `.so` files into the target environment.

##### PC sampling improvements

ROCprofiler-SDK has the following enhancements to the PC sampling feature:

* The PC sampling buffer size is increased on AMD Instinct MI300 and MI350 Series GPUs, reducing sample loss on high-rate sampling runs.
* Fixed microsecond sampling intervals being incorrectly clamped by the cycle-interval clamp, so microsecond-denominated intervals are now honored as specified.
* Corrected the set of valid stochastic sampling intervals reported to users, so `rocprofv3-avail` and the SDK no longer advertise intervals the hardware doesn’t accept.

:::{note}
The larger buffer size applies only to AMD Instinct MI300 and MI350 Series GPUs. Other architectures keep the existing buffer size.
:::

##### hipFile statistics API support

ROCprofiler-SDK adds support for the hipFile statistics API, synchronizing the profiler ABI with the hipFile stats dispatch entries. This lets tools report aggregate storage I/O statistics alongside the per-call hipFile trace records already available.

##### Advanced Thread Trace (ATT) improvements

ROCprofiler-SDK has the following enhancements to the ATT feature:

* `rocprofv3` now honors `ROCPROF_ATT_LIBRARY_PATH` when locating the thread trace decoder library, instead of scanning from `/`.
* The decoder library is resolved by its versioned SONAME (`.so.<version>`), falling back to the unversioned name.
* `rocprof-trace-decoder` gains a version query, letting tools check decoder compatibility at runtime.
* Fixed gfx12 occupancy mode, and added a no-detail command-line option for lower-volume ATT collection.
* The ATT completion callback now stays active after a marker-driven pause, so `roctxProfilerPause`/`roctxProfilerResume` cycles no longer drop the trailing portion of a trace.

##### Output format and rocpd improvements

ROCprofiler-SDK has the following improvements for output format and `rocpd`:
* Perfetto event names are now cached for static interning, reducing the cost of Perfetto output generation on large traces.
* Perfetto buffer size limits are now validated up front, producing a clear error instead of a late failure when an out-of-range size is requested.
* `rocpd` Python modules are backward compatible with databases produced by earlier releases.
* `otf2` and `pandas` are now imported locally inside `rocpd`, so the tool no longer requires those packages for unrelated operations.
* `rocpd` marker regions are now named after the ROCTx message rather than a generic label, so ROCTx ranges are identifiable in the database and in converted output.
* `rocprofv3` no longer silently adds `rocpd` to an output format explicitly requested by the caller.

##### Quality and stability improvements

This release includes a range of quality and stability fixes across ROCprofiler-SDK and `rocprofv3`:

* **PyTorch co-existence:** A ROCprofiler-SDK bundled inside PyTorch no longer initializes alongside `rocprofv3`. Previously, two tools subscribing to code object tracing in the same process caused `rocprofv3` to crash during output generation, blocking profiling of PyTorch and Triton workloads through ROCm Compute Profiler.
* **SPM race condition:** Fixed a race in the Streaming Performance Monitor collection path.
* **OMPT trace abort:** Fixed the issue of `--ompt-trace` aborting the profiled application.
* **Code object finalize:** `code_object::finalize()` is now serialized with the destroy mutex, closing a shutdown race.
* **Host function IDs:** Host function IDs are now assigned once per symbol, eliminating duplicate identities for the same function.
* **HIP API coverage:** Added tracing and stringification support for newly introduced HIP APIs, including `hipDeviceGetLuid` and `hipInitDevice`.

For more information, see the [ROCprofiler-SDK section](#rocprofiler-sdk-1-4-1) in the ROCm component changelogs.

#### ROCm Compute Profiler feature highlights

The following are notable enhancements to the ROCm Compute Profiler (rocprofiler-compute):

##### Roofline enhancement

* The standalone roofline report is now a fully interactive HTML page. You can isolate a single kernel or a single roof, select several kernels at once, and switch the arithmetic intensity axis between cache levels, instead of reading one static plot where every kernel shares the same marker.

* Roofline benchmarking and analysis are supported on AMD Ryzen AI APUs with Gorgon Point (gfx1153) integrated graphics. Peak values are now measured on this hardware instead of reported as N/A.

For details, see [Analyzing with the CLI](https://rocm.docs.amd.com/projects/rocprofiler-compute/en/latest/how-to/analyze/cli.html) and [Supported accelerators](https://rocm.docs.amd.com/projects/rocprofiler-compute/en/latest/reference/compatible-accelerators.html).

##### PC sampling improvement

* PC sampling analysis now reports results per kernel. Running `rocprof-compute analyze --output-format csv` writes each kernel's disassembly to its own folder, with the sample and stall counts on every instruction and the source file the kernel was compiled from saved beside it. The same data is available from the analysis database.
* PC sampling supports workloads that span several processes. Each process writes its own results file, so processes no longer overwrite each other, and a single analysis run reports all of them together with the owning process identified.
For details, see [Using PC sampling in ROCm Compute Profiler](https://rocm.docs.amd.com/projects/rocprofiler-compute/en/latest/how-to/pc_sampling.html).

##### Profiling output update

* Profiling always writes the `rocpd` output format, which was already the default. The CSV output backend and the `--format-rocprof-output` option have been removed. Workloads captured with the CSV backend by an earlier release must be profiled again before they can be analyzed.
* Profiling artifacts are considerably smaller. Counter data is highly redundant and is now compressed as it is written. Profile mode also streams counter data to disk instead of holding it in memory, which lowers peak memory use on large workloads.
For details, see [Profile mode](https://rocm.docs.amd.com/projects/rocprofiler-compute/en/latest/how-to/profile/mode.html).

##### Profiler documentation update
Added a guide for profiling vLLM workloads, including the caveats that apply to inference servers. For details, see [Profile vLLM workloads](https://rocm.docs.amd.com/projects/rocprofiler-compute/en/docs-10.1.0/how-to/profile/mode.html#profile-vllm-workloads).

For more information, see the [ROCm Compute Profiler section](#rocm-compute-profiler-3-9-0) in the ROCm component changelogs.

#### ROCm Systems Profiler feature highlights

The following are notable enhancements to ROCm Systems Profiler:

##### rocpd is now the default output format

When no output format is specified, profiling data is emitted as a rocpd SQLite database (`rocpd.db`). Perfetto (`.proto`) output is still available and must now be explicitly enabled via `--output-format proto`. For more information, see the [ROCm Profiling Data (rocpd) output](https://rocm.docs.amd.com/projects/rocprofiler-systems/en/latest/how-to/understanding-rocprof-sys-output.html#rocm-profiling-data-rocpd-output).

##### Removed the deprecated rocprof-sys-user library and its C APIs

`rocprof-sys-user` library and its C APIs (`rocprofsys_user_*`, `<rocprofiler-systems/user.h>`) have been removed, including the `user` find_package component, the `examples/user-api example`, the Python `rocprofsys.user` submodule, and the associated pytest coverage. Use ROCTx (`rocprofiler-sdk-roctx`) for general-purpose manual instrumentation (starting and stopping tracing, named regions) instead. See [examples/roctx](https://github.com/ROCm/rocm-systems/blob/docs/10.1.0/projects/rocprofiler-systems/examples/roctx/roctx.cpp) for usage.

For more information, see the [ROCm Systems Profiler section](#rocm-systems-profiler-1-9-0) in the ROCm component changelogs.

### Control and monitoring tools

#### AMD SMI feature highlights

The following are notable changes to AMD SMI:

##### AMD SMI WSL2 support (Technical preview)

AMD SMI adds a backend as a technical preview feature for reporting GPU telemetry under Windows Subsystem for Linux 2 (WSL2), extending discovery and monitoring to WSL2 environments for the first time.

* **Discovery and monitoring only:** AMD SMI detects AMD GPUs and reports available telemetry, including temperature, power, memory usage, utilization, clock speeds, board, and ASIC identifiers, and driver and firmware versions, to the extent the environment exposes it. Management and configuration actions are not available, matching AMD SMI's existing behavior on Windows.

* **Some categories are unavailable under WSL2:** Because WSL2 reaches the GPU through the Windows display driver rather than native Linux DRM/sysfs, several categories are reported as unsupported rather than failing: CPU/ESMI, NIC, switch, fabric, ECC, and partition queries, along with per-process GPU usage (`amd-smi` process).

* **Graceful fallback:** When a metric can't be read, or no AMD GPU is present, AMD SMI reports it as unavailable and continues rather than crashing or hanging.

* **Documented capability differences:** A published guide covers prerequisites and the full set of capabilities that differ between native Linux and WSL2. For more details, refer to [Using AMD SMI under WSL](https://rocm.docs.amd.com/projects/amdsmi/en/docs-10.1.0/how-to/amdsmi-wsl-mode.html).

### Libraries

#### Component addition to ROCm Core SDK

The following components have now been added to the ROCm Core SDK:

##### hipThreads GPU concurrency library added

ROCm 10.1.0 introduces hipThreads to the ROCm Core SDK under the ROCm Threading libraries. hipThreads helps developers accelerate existing CPU-threaded code on GPUs without a full ROCm or CUDA rewrite. It is designed for applications with parallel CPU workloads that can benefit from GPU execution, but where the effort of moving to a full GPU programming model is not justified. By bringing familiar C++ threading concepts to GPUs, hipThreads provides a lower-friction path to incremental GPU acceleration on Linux and Windows. For more information, see the [hipThreads documentation](https://rocm.docs.amd.com/projects/hipThreads/en/docs-10.1.0/).

##### libhipcxx C++ Standard Library for HIP added

ROCm 10.1.0 introduces libhipcxx to the ROCm Core SDK, joining rocPRIM, rocThrust, and hipCUB in the ROCm math and compute libraries. libhipcxx is the C++ Standard Library for HIP, bringing Standard Library features such as atomics, type traits, and containers to both host and device code, so you can use familiar C++ types directly in HIP kernels without writing your own device-side implementations. See the [libhipcxx documentation](https://rocm.docs.amd.com/projects/libhipcxx/en/docs-10.1.0/index.html) to get started.

##### RPP computer vision library added

ROCm 10.1.0 introduces RPP (ROCm Performance Primitives) to the ROCm Core SDK, joining rocDecode and rocJPEG in the ROCm media and vision libraries. RPP is a high-performance computer vision library that provides GPU-accelerated 2D image and 3D image (voxel) augmentations, as well as other miscellaneous augmentations and primitives, for AI training and inference data pipelines. RPP is supported on Linux with AMD Instinct and Radeon GPUs. See the [RPP documentation](https://rocm.docs.amd.com/projects/rpp/en/docs-10.1.0/) to get started.

#### hipFFT and rocFFT feature highlights

The following are notable enhancements to hipFFT and rocFFT:

##### hipFFT expands single-process multi-device support

hipFFT's single-process, multi-device plans now behave consistently across all supported backends:

* **Batched transforms:** All batched transforms are supported in-place and out-of-place, across all devices.
* **Unbatched multi-dimensional transforms:** In-place unbatched, multi-dimensional transforms are now supported across devices.
* **Consistent data distribution:** Plans created with `hipfftMakePlan2d`, `hipfftMakePlan3d`, and `hipfftMakePlanMany` now use the same data decomposition regardless of backend, removing a source of inconsistent results for multi-device workflows.

Multi-device, unbatched one-dimensional transforms are not yet supported.

##### hipFFT and rocFFT add JIT callbacks

hipFFT and rocFFT now compile load/store callbacks just-in-time (JIT) into their own kernels, letting the FFT library optimize the combined callback and transform code — something the previous function-pointer callback APIs prevented.
* **hipFFT:** Call `hipfftXtSetJITCallback` after `hipfftCreate` and before initializing the plan with a `MakePlan` function. This deprecates `hipfftXtSetCallback` and `hipfftXtClearCallback`, and isn't yet compatible with multi-GPU transforms.
* **rocFFT:** Configure callbacks with `rocfft_plan_description_set_load_callback` / `_store_callback` during plan creation, and supply per-execution data pointers with `rocfft_execution_info_set_load_callback_data` / `_store_callback_data`. This deprecates `rocfft_execution_info_set_load_callback` / `_store_callback`, and isn't yet compatible with transforms that also specify fields or bricks.

#### hipFile adds async fastpath, batch I/O, and stats APIs

hipFile adds three capabilities for AMD Infinity Storage:

* **Fastpath backend for Async I/O:** `hipFileReadAsync()` and `hipFileWriteAsync()` now support the fastpath backend, letting asynchronous storage to GPU I/O requests run on a HIP stream without host bounce-buffer copies that are relevant for frameworks like LMCache. Note that async fastpath operations do not currently retry on the fallback backend if the fastpath fails in this release. Async fastpath fallback support coming in a subsequent release.

* **Batch I/O:** Batch IO API operations are now fully supported using an internal thread pool that is relevant for frameworks like NIXL.

* **Stats API:** `hipFileGetStatsL1/L2/L3()` return progressively detailed I/O statistics. L1 gives basic I/O and operation counts, L2 adds I/O size histograms, L3 adds per-GPU breakdowns, giving ROCprofiler-SDK and other tools visibility into hipFile I/O behavior.

#### hipSPARSE and rocSPARSE feature highlights

The following are notable enhancements to hipSPARSE and rocSPARSE:

##### rocSPARSE and hipSPARSE add batched SDDMM support

rocSPARSE and hipSPARSE now support batched SDDMM (sampled dense-dense matrix multiplication), computing many independent SDDMM problems in a single call instead of one at a time, reducing per-call overhead for workloads that process many small problems together.

* **Broadcast modes:** Four batching patterns are supported, letting either or both dense operands (A, B) be shared across the batch or vary per batch index — configured with `hipsparseDnMatSetStridedBatch` (hipSPARSE) or `rocsparse_dnmat_set_strided_batch` (rocSPARSE).

* **Format coverage:** rocSPARSE supports CSR, CSC, COO, COO AoS, and ELL formats (via `rocsparse_csr_set_strided_batch` and equivalents); hipSPARSE currently supports CSR only (via `hipsparseCsrSetStridedBatch`).

##### hipSPARSE adds generic SpGEAM API

hipSPARSE now provides a generic API for sparse matrix-matrix addition, computing C = alpha × op(A) + beta × op(B) for CSR matrices on both AMD and CUDA backends, backed by the new `rocsparse_spgeam` routine on ROCm.

* **New entry points:**
  * `hipsparseSpGEAM_createDescr` / `hipsparseSpGEAM_destroyDescr` create and destroy an operation descriptor (`hipsparseSpGEAMDescr_t`).
  * `hipsparseSpGEAM_bufferSize` queries the required workspace; `hipsparseSpGEAM_nnz` computes C's row offsets and non-zero count.
  * `hipsparseSpGEAM` computes the final values and sorted column indices.
* **Supported types:** `HIP_R_32F`, `HIP_R_64F`, `HIP_C_32F`, and `HIP_C_64F` compute types, with 32-bit and 64-bit index types (A, B, and C must share the same index type).
* **Constraints and error handling:** Only CSR format and non-transpose operations are supported — transpose operations and non-CSR formats now return a well-defined error (`HIPSPARSE_STATUS_NOT_SUPPORTED`) instead of the silently-accepted-but-incorrect transpose behavior on the ROCm backend in prior releases. A zero alpha or beta collapses C's sparsity pattern to that of the other operand; both zero produce an empty C.

##### SpMM non-zero split batching in rocSPARSE

rocSPARSE SpMM now supports batched computation with the non-zero split algorithm (`rocsparse_spmm_alg_csr_nnz_split`), letting workloads that select this algorithm batch multiple independent SpMM operations into a single call instead of launching them individually. Previously, only the default row split algorithm supported batched computation.

##### rocSPARSE optimizes Level 2/3 routines for RDNA4

rocSPARSE Level 2 (SpMV) and Level 3 (SpMM) routines now use wave32-aware launch and geometry tuning on RDNA4 (gfx1201) GPUs, improving throughput with no change to numerical results and no change to wave64 (CDNA) behavior.

**Tuned routines:** CSR SpMV (adaptive, nnz-split, and long-row-balanced paths), COO and ELL SpMV, BSR/BSRX SpMV, `gemvi`, `gemmi`, and blocked-ELL SpMM (`bellmm`).

##### rocSPARSE ELL format triangular solve support

rocSPARSE now supports the ELL (ELLPACK) sparse matrix format in its triangular solve routines (`rocsparse_spsv` and `rocsparse_sptrsv`), joining the existing CSR and CSC support. Matrices already stored in ELL format can run triangular solve directly, without first converting to CSR or CSC. Transposed and conjugate-transposed operations are not yet supported for the ELL format.

#### hipSOLVER and rocSOLVER feature highlights

The following are notable enhancements to hipSOLVER and rocSOLVER:

##### hipSOLVER fixes and extends 64-bit APIs

hipSOLVER adds a 64-bit-compatible `hipsolverDnXlarft` function (with a corresponding `hipsolverDnXlarft_bufferSize` buffer-size query), backed by a new `LARFT_64` routine in rocSOLVER. Separately, `hipsolverDnXpotrs` is fixed to call its 64-bit `potrs` implementation instead of silently falling back to the 32-bit routine, correcting results for calls made through the 64-bit API on large matrices.

##### rocSOLVER improves Hermitian eigensolver performance

rocSOLVER now offers a 2-stage reduction to tridiagonal form for the Hermitian and symmetric eigensolvers (HEEVD/SYEVD) and their generalized counterparts (HEGVD/SYGVD), moving more of the computation to Level 3 BLAS operations. Improved performance compared to the previous SYEVD implementation was observed for large matrices (n > 11,000) on AMD Instinct MI300X GPUs.

##### rocSOLVER adds Cholesky QR factorization

rocSOLVER now offers Cholesky QR (CHOLQR) as an alternative to Householder-based QR factorization (GEQRF), offering a faster path to computing the QR decomposition of a matrix. CHOLQR and its 64-bit counterpart CHOLQR_64 are available in standard, batched, and strided_batched forms, with selectable algorithm variants that trade some speed for improved numerical stability on ill-conditioned matrices.

(release-supported-hw)=

## AMD hardware support

The following table lists supported AMD Instinct GPUs, Radeon GPUs, and Ryzen APUs. Each supported device is listed with its corresponding GPU microarchitecture and LLVM target.

:::{note}

If your GPU is not listed, it might be community-enabled through TheRock nightly builds. For more information, see [TheRock supported GPUs](https://github.com/ROCm/TheRock/blob/main/SUPPORTED_GPUS.md). For installation guidance, see [TheRock releases](https://github.com/ROCm/TheRock/blob/main/RELEASES.md).
:::

```{datatemplate:yaml} /data/gpus.yaml
:template: hardware-support-table.md.jinja
```

(release-supported-os)=

## Operating system support

ROCm supports the following Linux distributions and Microsoft Windows versions. If you're running ROCm on Linux, ensure your system is using a supported kernel version.

:::{important}
The following table is a general overview of supported operating systems. Actual support might vary by AMD GPU or APU. Use the {doc}`Compatibility matrix </compatibility/compatibility-matrix>` to verify support for your specific setup before installation.
:::

```{datatemplate:yaml} /data/os-support.yaml
:template: os-support-table.md.jinja
```

## Installation updates

### WSL2 support now included in TheRock artifacts (Technical preview)

You can now run ROCm workloads on Windows through Windows Subsystem for Linux 2 (WSL2), for the first time in TheRock builds of ROCm. Previously, enabling WSL2 required building `librocdxg` from source;
TheRock now ships it as part of its artifacts, so the WSL2 driver interface is available out of the box.

The ROCr runtime automatically detects a WSL2 environment by checking for the `/dev/dxg` device and loads `librocdxg` accordingly, so no manual configuration is required in the common case. To disable auto-detection, set the `HSA_ENABLE_DXG_DETECTION` environment variable to 0 (the default is 1). For details, see the [ROCR-Runtime environment variables](https://rocm.docs.amd.com/en/latest/reference/environment-variables/index.html#rocr-runtime-environment-variables) reference.

This technical preview launch targets Ubuntu through `.deb` packages and Python wheels; RPM packages are not in scope. Profiling, debugging, and KFD-dependent tooling are not supported in WSL2 at launch. For details on issues identified with the initial support for WSL2, refer to [ROCm known issues](#rocm-known-issues).

### Runfile installer update

ROCm 10.1.0 fixes minor issues in the Runfile Installer.

(release-supported-fw)=

## Kernel driver and firmware bundle support

ROCm requires a coordinated stack of compatible firmware, driver, and user-space components. Maintaining version alignment between these layers ensures correct GPU operation and performance, especially for AMD data center products. While AMD publishes the AMD GPU driver and ROCm user space components, your server OEM (original equipment manufacturer) or infrastructure provider distributes the firmware packages. AMD supplies those firmware images (platform level data model (PLDM) bundles), which the OEM integrates and distributes.

```{datatemplate:yaml} /data/driver-firmware-support.yaml
:template: driver-firmware-support-table.md.jinja
```

(release-virtualization-support)=

## GPU virtualization support

AMD Instinct and Radeon GPUs support virtualization in the following configurations. Supported SR-IOV configurations require the AMD GPU Virtualization Driver (GIM) 9.3.0.K—see the [AMD Instinct Virtualization Driver documentation](https://instinct.docs.amd.com/projects/virt-drv/en/mainline-9.3.0.k/) for more information.

```{datatemplate:yaml} /data/virtualization-support.yaml
:template: virtualization-support-table.md.jinja
```

(release-gpu-partitioning-support)=

## GPU partitioning support

```{datatemplate:yaml} /data/partitioning-support.yaml
:template: partitioning-support-table.md.jinja
```

See the [AMD GPU partitioning](https://instinct.docs.amd.com/projects/amdgpu-docs/en/latest/gpu-partitioning/index.html) topic in the AMD GPU Driver documentation to learn more.

(release-ai-ecosystem)=

## AI ecosystem support

ROCm 10.1.0 provides optimized support for popular deep learning frameworks and AI inference engines. The following table lists supported frameworks and libraries, their compatible operating systems, and validated versions.

:::{important}
The following table is a general overview of supported frameworks and AI inference engines. Actual support might vary by AMD GPU or APU. Use the {doc}`Compatibility matrix </compatibility/compatibility-matrix>` to verify support for your specific setup.
:::

```{include} ./include/ai-ecosystem-support-table.html
:parser: myst
```

(release-components)=

## ROCm Core SDK components

The following table lists core tools and libraries included in the ROCm 10.1.0 release.

:::{important}
The following table is a general overview of ROCm Core SDK components. Actual support for these libraries and tools can vary by GPU and OS. Use the {doc}`Compatibility matrix </compatibility/compatibility-matrix>` to verify support for your specific setup.
:::

```{datatemplate:yaml} /data/components-current.yaml
:template: core-sdk-components-table.html.jinja
```

### ROCm component changelogs

The following sections describe key changes to ROCm Core SDK components.

```{note}
For a historical overview of ROCm component updates, see the {doc}`ROCm consolidated changelog </release/changelog>`.
```

```{include} ./include/core-sdk-components-aggregated-changelog.md
:parser: myst
```

## ROCm breaking changes

### ROCm SMI is removed from the standard build

ROCm SMI (rocm-smi-lib) is deprecated and, starting with this release, is no longer built or installed as part of the standard `ROCm/TheRock` build. Use the AMD SMI library and its `amd-smi` CLI as the supported replacement.


## ROCm known issues

ROCm known issues are noted on {fab}`github` [GitHub](https://github.com/ROCm/TheRock/issues?q=is%3Aissue%20state%3Aopen%20label%3A%22Verified%20Issue%22). These issues will be fixed in a future ROCm release. For known issues related to individual components, review the [ROCm component changelogs](#rocm-component-changelogs).

### Hugging Face model training throughput might regress for specific models on AMD Instinct MI350X

When you run Hugging Face BERT, RoBERTa-large, DistilBERT-base, or GPT-2 training workloads on AMD Instinct MI350X (gfx950) GPUs,  you might observe 6–22% longer training wall-clock time than expected. BERT and DistilBERT-base training are affected whether or not DeepSpeed ZeRO stage 0 is used. RoBERTa-large and GPT-2 training are affected only when using DeepSpeed ZeRO stage 0. See [GitHub issue #8878](https://github.com/ROCm/TheRock/issues/8878).

```{note}
The impact of this issue was initially reported in ROCm 10.0.0 and is partially addressed in ROCm 10.1.0 for other Hugging Face model configurations. See the [ROCm 10.1.0 resolved issues](hugging-face-model-training-throughput-for-specific-models-is-restored-on-amd-instinct-mi350x) entry for details.
```

### amd-smi event command might stop responding indefinitely on WSL

Running the `amd-smi event` subcommand on Windows Subsystem for Linux (WSL) might cause the process to stop responding, requiring manual intervention to terminate it. Note that the event subcommand is not supported on WSL.

As a workaround, avoid running `amd-smi event` on WSL. If the command is run accidentally, terminate the process manually using `Ctrl+C` (twice) or use:

```bash
kill -9 <pid>
```
See [GitHub issue #8787](https://github.com/ROCm/TheRock/issues/8787).

### Some GPU workloads on WSL2 might stop responding for certain device-side executions

Some GPU workloads running on WSL2 might stop responding when they encounter certain device-side execution conditions, such as HIP device-side assertions or OpenCL device-side enqueues. In these situations, the GPU workload stops making progress, but the host application might remain active while waiting for completion, causing the application to become unresponsive until it is manually terminated. This issue may be observed on AMD Radeon GPUs, such as the Radeon RX 7900 XTX, when running under WSL2. Most HIP and OpenCL workloads are not affected. The issue is limited to workloads that trigger the affected execution paths. As a workaround, you can manually terminate the affected process and restart the application. For OpenCL workloads, avoid using device-side enqueue on WSL2 when possible. See [GitHub issue #8791](https://github.com/ROCm/TheRock/issues/8791).

### RCCL allreduce operations using the LL protocol might produce incorrect results on AMD Instinct MI350 Series GPUs

Multi-GPU workloads relying on RCCL collective communications might produce incorrect results during `allreduce` operations when the Low Latency (LL) protocol is active. The issue has been observed to affect AMD Instinct MI350 Series GPUs. Affected workloads include distributed training and inference jobs where data correctness across GPUs cannot be guaranteed under this protocol.

As a workaround, set the following environment variable to restrict RCCL to protocols that are known to produce correct results:

```bash
export NCCL_PROTO=LL128,Simple
```

See [GitHub issue #8792](https://github.com/ROCm/TheRock/issues/8792).

### Valgrind workloads might stop responding on WSL2 with high system RAM

Running workloads under Valgrind on WSL2 might stop responding during ROCm runtime initialization on AMD Radeon GPUs, such as the AMD Radeon RX 9060, on systems with a large amount of system RAM. This happens because WSL reserves GPU address space upfront in a way that can collide with Valgrind's reserved memory range as system RAM grows. As a workaround, if Valgrind must be used on WSL2, limit the RAM available to the WSL instance to below approximately 36 GB (for example, using the `.wslconfig` `memory=` setting). Try lowering it more if the issue persists. Workloads that don't use Valgrind are unaffected. See [GitHub issue #8793](https://github.com/ROCm/TheRock/issues/8793).

### RAS error injection and query are unavailable on AMD Instinct MI350P GPUs

On AMD Instinct MI350P GPUs, the RAS error injection and query interfaces (used to validate error-handling paths, for example with the amdgpuras tool) are not supported in ROCm 10.1.0. Injection commands fail, and query commands report that the block doesn't support querying. This affects only manual RAS error injection and query tooling. Normal GPU reliability features such as ECC detection and error recovery during regular operation are not affected. See [GitHub issue #8794](https://github.com/ROCm/TheRock/issues/8794).

### ComfyUI workloads might show lower performance on Radeon Pro W7900 GPUs

ComfyUI-based AI workloads running on Radeon Pro W7900 GPUs might experience lower performance compared to previous ROCm releases. Impacted workloads include image and video generation models such as Stable Diffusion, Wan, and LTX. See [GitHub issue #8842](https://github.com/ROCm/TheRock/issues/8842).

### Device-to-device hipMemcpyAsync might fail on AMD Instinct MI200 Series GPUs

On AMD Instinct MI200 Series GPUs, device-to-device `hipMemcpyAsync` copies that use the DMA engine path (for example, with the `hipMemcpyDeviceToDeviceNoCU` flag) might fail without changing the destination buffer and reporting an error. This happens because the copy path selects an SDMA engine without properly validating which engines are actually usable for that GPU pair, so the transfer can target an engine that never completes the write. Although this issue was validated on PCIe-connected systems, the root cause is topology-agnostic and might also affect xGMI-linked systems. As a workaround, use `hipMemcpyDeviceToDevice` (without `NoCU`), which copies through compute units instead of DMA and is not affected. See [GitHub issue #8843](https://github.com/ROCm/TheRock/issues/8843).

### Multi-application workloads might experience an intermittent GPU page fault on some Instinct GPUs

Running multiple concurrent applications with high GPU memory-transfer activity, including SDMA-driven copies and SDMA engine stalls, might intermittently trigger a GPU page fault on some Instinct GPUs. When this happens, the affected job stops responding and must be restarted. In some cases, the GPU queue does not recover until a timeout of about 15 minutes expires, and the kernel log reports a retry page fault. The issue is more likely to occur when GPU memory is already under pressure from a recently run workload. See [GitHub issue #8844](https://github.com/ROCm/TheRock/issues/8844).

## ROCm resolved issues

The following notable issues have been fixed in ROCm 10.1.0.

### PyTorch training and fine-tuning workloads might experience GPU resets or crashes on some Radeon GPUs

Previously, PyTorch training and fine-tuning workloads using Llama-Factory or Unsloth could experience GPU resets or application crashes on some AMD Radeon graphics products, such as the Radeon RX 9070 Series and Radeon AI PRO R9700. See [GitHub issue #7699](https://github.com/ROCm/TheRock/issues/7699).

### Hugging Face model training throughput for specific models is restored on AMD Instinct MI350X

ROCm 10.1.0 restores Hugging Face model training throughput to previous levels for BART, DiT (Diffusion Transformers), GPT-2, and Llama 2 70B Chat workloads on AMD Instinct MI350X (gfx950) GPUs, recovering the 9–25% throughput loss reported in the ROCm 10.0.0 known issues. The regression was caused by AOTriton 0.13b selecting a suboptimal flash-attention backward kernel instead of the faster 3-kernel split implementation used in AOTriton 0.11.2b. This kernel selection is corrected for these models in ROCm 10.1.0. See [GitHub issue #7696](https://github.com/ROCm/TheRock/issues/7696).

```{note}
BERT and DistilBERT-base training continue to be affected in ROCm 10.1.0 with or without DeepSpeed, along with GPT-2 and RoBERTa-large training that use DeepSpeed ZeRO stage 0. See the [ROCm 10.1.0 known issues](hugging-face-model-training-throughput-might-regress-for-specific-models-on-amd-instinct-mi350x) for current status and workarounds.
```
