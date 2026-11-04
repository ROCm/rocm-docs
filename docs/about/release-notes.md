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

This release focuses on profiling, runtime enhancements, and AI-assisted developer workflows across AMD Instinct™, Radeon™, and Ryzen™ AI platforms. Highlights include support for additional AMD Ryzen APUs and new virtualization configurations, updated PyTorch, JAX, vLLM, and SGLang versions, and new HIP APIs. The profiling tools gain WSL 2 profiling and Agent Skills. AMD System Management Interface (AMD SMI) adds power profile management, and the libraries add complex GEMM in hipBLASLt and new sparse routines in rocSPARSE.

### Platform and hardware support

This release adds support for additional AMD Ryzen APUs and new GPU virtualization configurations.

#### Expanded AMD GPU support

ROCm {{ ROCM_VERSION }} adds support for the following AMD APUs:

- [AMD Ryzen 7 PRO 253](https://www.amd.com/en/products/processors/laptop/ryzen-pro/200-series/amd-ryzen-7-pro-253.html)
- [AMD Ryzen 7 PRO 249](https://www.amd.com/en/products/processors/laptop/ryzen-pro/200-series/amd-ryzen-7-pro-249.html)
- [AMD Ryzen 7 PRO 217](https://www.amd.com/en/products/processors/laptop/ryzen-pro/200-series/amd-ryzen-7-pro-217.html)
- [AMD Ryzen 5 PRO 225](https://www.amd.com/en/products/processors/laptop/ryzen-pro/200-series/amd-ryzen-5-pro-225.html)
- [AMD Ryzen 5 PRO 224](https://www.amd.com/en/products/processors/laptop/ryzen-pro/200-series/amd-ryzen-5-pro-224.html)
- [AMD Ryzen 5 PRO 216](https://www.amd.com/en/products/processors/laptop/ryzen-pro/200-series/amd-ryzen-5-pro-216.html)
- [AMD Ryzen 3 PRO 205](https://www.amd.com/en/products/processors/laptop/ryzen-pro/200-series/amd-ryzen-3-pro-205.html)
- [AMD Ryzen 7 253](https://www.amd.com/en/products/processors/laptop/ryzen/200-series/amd-ryzen-7-253.html)
- [AMD Ryzen 7 249](https://www.amd.com/en/products/processors/laptop/ryzen/200-series/amd-ryzen-7-249.html)
- [AMD Ryzen 7 217](https://www.amd.com/en/products/processors/laptop/ryzen/200-series/amd-ryzen-7-217.html)
- [AMD Ryzen 5 225](https://www.amd.com/en/products/processors/laptop/ryzen/200-series/amd-ryzen-5-225.html)
- [AMD Ryzen 5 224](https://www.amd.com/en/products/processors/laptop/ryzen/200-series/amd-ryzen-5-224.html)
- [AMD Ryzen 5 216](https://www.amd.com/en/products/processors/laptop/ryzen/200-series/amd-ryzen-5-216.html)
- [AMD Ryzen 3 205](https://www.amd.com/en/products/processors/laptop/ryzen/200-series/amd-ryzen-3-205.html)
- [AMD Ryzen 9 180](https://www.amd.com/en/products/processors/laptop/ryzen/100-series/amd-ryzen-9-180.html)
- [AMD Ryzen 7 165](https://www.amd.com/en/products/processors/laptop/ryzen/100-series/amd-ryzen-7-165.html)
- [AMD Ryzen 7 155](https://www.amd.com/en/products/processors/laptop/ryzen/100-series/amd-ryzen-7-155.html)
- [AMD Ryzen 5 125](https://www.amd.com/en/products/processors/laptop/ryzen/100-series/amd-ryzen-7-125.html)

For the complete list of supported AMD hardware, see [AMD hardware support](#amd-hardware-support).

#### Operating system support update

Operating system support remains unchanged in this release. For the full list of supported Linux distributions, see [Operating system support](#operating-system-support).

#### Expanded GPU virtualization support for Instinct and Radeon GPUs

ROCm {{ ROCM_VERSION }} adds support for the following virtualization configurations on AMD Instinct and Radeon GPUs:

**Instinct**

* On AMD Instinct MI350P:
  * KVM SR-IOV with Ubuntu 24.04 host OS and Ubuntu 24.04 guest OS.

**Radeon**

* On AMD Radeon PRO W7800, W7900, and W7900 Dual Slot:
  * KVM Passthrough with Ubuntu 24.04 or Ubuntu 26.04 host OS and Ubuntu 24.04 or Ubuntu 26.04 guest OS.

Supported Single Root I/O Virtualization (SR-IOV) configurations require the [AMD GPU Virtualization Driver (GIM) 9.4.0.K](https://mkmartifactory.amd.com:8443/artifactory/sw-virt-prod-local/KVM_OpenSource_Releases/9.4.0.K/). For details, see [GPU virtualization support](#gpu-virtualization-support).

#### GPU partitioning support update

GPU partitioning support remains unchanged in this release. For details, see [GPU partitioning support](#gpu-partitioning-support).

### AI inference and frameworks

This release enables support for the following frameworks:

* PyTorch 2.15.0
* JAX 0.11.2
* vLLM 0.32.0
* SGLang 0.5.21

The updated framework support replaces the previous PyTorch 2.12.0, JAX 0.11.0, vLLM 0.29.0, and SGLang 0.5.18 support.

For details, see [AI ecosystem support](#ai-ecosystem-support).

### Runtime and compilers

#### HIP feature highlights

The following are notable enhancements to HIP:

##### Select the target HIP API version with `HIP_TARGET_API_VERSION`

The new `HIP_TARGET_API_VERSION` macro lets you select the target HIP API version at compile time, which helps maintain source compatibility for applications written against an earlier API version. The default is `700`, which matches the current HIP 7 API, and changes only at a major ROCm release. Although setting the macro has no effect today, define it explicitly so that applications that state their target API version continue to compile unchanged when the default moves.

When the HIP 8 API is introduced in a future release, this macro will initially be the only way to access it: you opt in by defining `HIP_TARGET_API_VERSION` as `800`, while the default remains `700`. Because the macro is set per translation unit, you can preview the HIP 8 API side by side with HIP 7 in the same codebase. Build one target against `800` while the rest of your build stays on `700`. For example, in the HIP 8 API, `hipMemAdvise` maps to `hipMemAdvise_v2`, which takes a `hipMemLocation` structure instead of a device ID.

##### New module, library, and kernel APIs

New APIs provide more control over modules, libraries, and kernels. `hipModuleEnumerateFunctions` returns the function handles defined in a loaded module, so applications no longer need to know kernel names ahead of time. `hipLibraryGetModule` returns the module handle associated with a library, and `hipKernelSetAttributeForDevice` sets a device-wide attribute for a kernel on a specific device.

##### Device-side fault reporting on WSL 2

HIP now reports errors for device-side out-of-bounds writes, null pointer dereferences, and asserts on WSL 2, rather than leaving affected applications hanging.

For more information, see the [HIP section](#hip-10-2-0) in the ROCm component changelogs.

### Profiling and debugging tools

#### ROCprofiler-SDK feature highlights

The following are notable enhancements to ROCprofiler-SDK:

##### KFD dispatch log tracing (CP tracing) for kernel tracing

ROCprofiler-SDK adds a new way to collect kernel tracing information, called KFD dispatch log tracing, also known as CP tracing. The GPU command processor firmware already writes an end-of-pipe record for every dispatch it retires. On supported hardware, KFD exposes those records to user space as a dispatch log stream, and ROCprofiler-SDK consumes them directly instead of deriving completion from HSA signals. This capability is enabled on gfx1250 systems running amdgpu driver 31.70 and ROCm 10.2.0.

The SDK opens a dispatch log stream per supported GPU through the KFD profiler ioctl and maps the kernel-owned GTT backing read-only, so firmware records are consumed zero-copy with no intermediate buffer. A dispatch batch registers a pending entry before its packets are published, publishes those packets untouched, and is completed later when the reader confirms retirement from the corresponding firmware record. Dispatch batches that aren't eligible keep the existing signal-based path, so both mechanisms work in the same run.

Signal-less dispatch completion is opt-in. It activates only when `ROCPROFILER_KFD_DISPATCH_LOG_SIGNAL_LESS` is explicitly set to a true value:

```shell
ROCPROFILER_KFD_DISPATCH_LOG_SIGNAL_LESS=true rocprofv3 --kernel-trace -- <application>
```

When the path is active, ROCprofiler-SDK logs that signal-less kernel dispatch completion is in effect. Unsetting the variable returns the process to signal-based completion.

The SDK validates the GPU and the dispatch log ABI at startup (stream ABI version 4, profiler ioctl ABI version 6) and enables the path only on gfx1250 with amdgpu driver 31.70 or later. If that check fails, the path is skipped and dispatch timing falls back to the existing signal-based mechanism, so kernel tracing still works and the output is unchanged. The default ring size and in-flight dispatch budget suit most workloads. For graph-launch-heavy runs that lose dispatch records, set the ring size with `ROCPROFILER_KFD_DISPATCH_LOG_SIZE_KB` and the in-flight budget with `ROCPROFILER_KFD_DISPATCH_LOG_MAX_PENDING_PER_GPU` (default 2,000,000 per GPU).

Signal-less completion changes dispatch completion semantics process-wide, which is why it's opt-in. Because records are recovered from firmware rather than from a signal the profiler owns, a dispatch whose record is lost, through a ring overrun or a queue torn down mid-flight, is counted and reported as a loss rather than given an incorrect timestamp.

##### Streaming Performance Monitor device counting mode and gfx12 support

* Device counting mode samples SPM counters on an agent for as long as the context is active, independent of any kernel dispatch. Use `rocprofiler_configure_spm_device_counting_service` to collect samples as `rocprofiler_spm_counter_record_t` records that carry no dispatch information, so you can observe device-level counter behavior across the whole run, including intervals in which nothing is dispatched.

* SPM is now supported on gfx12 and gfx1250. On gfx1250, samples also report the shader array and WGP, encoded in the counter instance ID alongside XCC, shader engine, and instance.

* The new `ROCPROFILER_SPM_PARAMETER_TYPE_SAMPLE_INTERVAL_REFCLK_CYCLES` parameter sets the SPM sample interval in reference-clock cycles rather than shader-clock cycles, giving a sampling rate that doesn't move with the shader clock. It's supported on gfx1200, gfx1201, and gfx1250; other agents return `ROCPROFILER_STATUS_ERROR_INVALID_ARGUMENT`. Agent configuration queries now report shader-clock and reference-clock intervals separately, and `rocprofv3 --spm-sample-interval-unit` accepts the new `refclk_cycles` value.

* The experimental `rocprofiler_spm_dispatch_counting_service_set_agents()` scopes dispatch SPM to a selected set of GPU agents. Two SPM contexts can now be active at the same time when their agent sets are disjoint.

* SPM counter location is now stored on `rocpd_info_pmc` in rocpd schema 3.0.5. Each combination of agent, counter, XCC, shader engine, shader array, WGP, and instance has its own pmc id, and `rocpd_pmc_event` stores the sample value and points at that id. Location dimensions that don't apply to a counter are stored as `NULL` and left blank in CSV output. The instance index is always stored.

##### Thread trace improvements

* Thread trace now allocates its memory and signals through KFD and copies trace data with SDMA, falling back to HSA with a warning when an SDMA copy isn't possible. This removes thread trace's dependence on HSA resource lifetime and fixes crashes and hangs that occurred when thread trace ran after `main()` returned.

* The new `ROCPROFILER_THREAD_TRACE_PARAMETER_RESOURCE_MODE` parameter and `rocprofiler_thread_trace_resource_mode_t` enum control when thread trace queues are allocated. Thread trace previously created a queue on every visible GPU at HSA initialization, which produced a large number of unused queues for applications spanning many processes or jobs that expose dozens of devices but use few of them. `ROCPROFILER_THREAD_TRACE_PARAMETER_RESOURCE_MODE_CODE_OBJECT` creates the queue for a GPU only when the first code object is loaded on it, so GPUs the application never uses get no queue. This is the new default. `ROCPROFILER_THREAD_TRACE_PARAMETER_RESOURCE_MODE_HSA` allocates at HSA initialization, matching previous releases. The `HIP`, `ALL`, and `DISPATCH` modes are reserved and return `ROCPROFILER_STATUS_ERROR_NOT_IMPLEMENTED`. In `rocprofv3`, select the mode with `--att-resource-mode` (`default`, `hsa`, or `code-object`). Use `--att-resource-mode hsa` to restore the previous allocation behavior.

* `rocprofiler_thread_trace_shader_data_t` gains `start_timestamp` and `end_timestamp` fields of type `rocprofiler_thread_trace_timestamp_t`, each carrying a GPU clock value and a system clock value. Thread trace data can now be lined up directly with API traces on a common timeline.

* SQTT markers previously required building the compiler from source. They now ship as part of the ROCm compiler package, so they're present in a standard ROCm installation.

* The `att-no-detail` parameter, which collects occupancy data without instruction-level detail for lower-volume traces, is now supported on gfx12.

* The thread trace decoder is now resolved by versioned library name (`librocprof-trace-decoder.so.<major>`, then `.so.<major>.<minor>`, then `.so.<major>.<minor>.<patch>`) before falling back to the unversioned name, so `rocprofv3 --att` works on installations that ship only the versioned library, such as TheRock runtime wheels.

* The `rocprof-trace-decoder` Python API, introduced in ROCm 10.0.1, was previously available only from the `rocm-systems` sources. It now ships with ROCm, so decoding ATT/SQTT data from Python works against a standard installation.

##### WSL 2 profiling support for RDNA 3.5

ROCprofiler-SDK and `rocprofv3` can now profile HIP and HSA workloads on WSL 2, including counter collection and thread trace (`--att`). Compute support itself is provided by the ROCm runtime; these changes add the profiling path on top of it. Support is validated on RDNA 3.5 (gfx1150, Strix Point); other gfx115x SKUs weren't explicitly validated.

On WSL the KMT topology is read up front through `librocdxg`, using the same thunk interface as the HSA runtime, and an agent is published only once all of its required topology information has been collected. `rocprofiler_query_available_agents` therefore returns consistent agent information whether or not the HSA runtime has initialized, which `rocprofv3-avail` and counter metric lookup both require. Native Linux continues to use the existing KFD/sysfs path for agent enumeration, though a few shared error-handling fixes from this work apply to Linux as well.

Because WSL 2 provides GPU access through `/dev/dxg` rather than `/dev/kfd`, paths that assume a native KFD device node now degrade cleanly: `rocprofv3 --pmc` no longer aborts when `/dev/kfd` is absent, and PC sampling and KFD-only controls are reported as unavailable. For thread trace, basic ATT capture and decoding are validated on gfx1150. Combining `--att` with `--selected-regions` isn't yet supported on WSL 2 and might produce an empty trace.

##### Dispatch path optimization

Counter collection, Advanced Thread Trace, SPM, and PC sampling used to add themselves to a shared list of callbacks that the HSA queue controller walked on every kernel dispatch. Each of these services now has its dispatch hooks called directly, and each call first checks whether that service is actually active on the GPU the dispatch is going to.

Queue interposition now issues a memory fence when writing packets to the ring buffer, so a packet can no longer be read reordered or half-written. A hang is also fixed that occurred when more kernels were linked by barriers than `ROCPROFILER_ASYNC_SIGNAL_HANDLER_THREADS` allowed, such as a HIP graph fork/join or an all-reduce. A single thread now waits for any completion signal, which also works with older ROCr versions.

##### High-precision timestamp correlation

ROCprofiler-SDK now requests ROCr high-precision GPU/CPU timestamp correlation during HSA registration when GPU timestamp collection is configured and the runtime provides the tools API. This tightens the alignment between GPU-side and host-side timestamps on a shared timeline. Older runtimes, and tools that trace only CPU APIs or markers, retain the existing behavior.

##### PC sampling sample validation

Stochastic PC sampling can return a sample whose program counter doesn't correspond to a real instruction boundary. `rocprofv3` now checks each stochastic sample against the instruction at its PC; samples that fail the check are dropped and counted as invalid, while samples from unknown code objects are kept as is. The same check is available to tool authors as `rocprofiler::sdk::pc_sampling::verify_sample` in the header-only `rocprofiler-sdk/cxx/pc_sampling.hpp`.

The PC sampling buffer size is also increased, from 4 MB to 64 MB, on the multi-chiplet targets gfx94x, gfx95x, and gfx125x. ROCr allocates one PC sampling buffer per XCC, so these multi-XCC parts need a larger overall buffer; at the previous size many samples were dropped or invalid at reasonable sampling rates. Single-XCC targets keep the 4 MB buffer.

##### Process attachment improvements

* Two attach defects are fixed: an HSA signal handle leak when reusing pooled signals, and the incorrect association of attach callbacks with tools when `rocprofiler_configure` and `rocprofiler_configure_attach` resolve from different shared objects.

##### OMPT tool chaining

ROCprofiler-SDK now takes the OpenMP Tools (OMPT) interface only when a registered client actually subscribes to OMPT tracing. Otherwise it passes the interface on to the next OMPT tool in the chain. A user's own OMPT tool can therefore run alongside ROCprofiler-SDK instead of being displaced by it.

##### Agent Skills

ROCprofiler-SDK ships a set of Agent Skills under `skills/`, packaging profiling workflows as instructions that AI coding assistants can follow: `rocprofv3` for general tracing, counter collection, and rocpd post-processing; `thread-trace` for ATT collection and interpretation; and `pc-sampling` for stochastic and host-trap sampling including stall-reason analysis. Each skill ships with evaluation cases and tests.

##### Expanded tracing coverage

* Memory copy tracing now covers `hsa_amd_memory_async_batch_copy`. Each copy in a batch is reported as its own memory copy record through the existing callback and buffer tracing kinds.

* Added tracing support for the HIP APIs `hipKernelSetAttributeForDevice`, `hipLibraryGetModule`, `hipModuleEnumerateFunctions`, and `hipDeviceFlushGPUDirectRDMAWrites`.

* Added tracing support for the HSA APIs `hsa_amd_enable_high_precision_timestamps`, `hsa_amd_agent_set_attribute`, `hsa_amd_vmem_get_vmem_info`, and `hsa_amd_svm_discard_and_prefetch_batch_async`.

##### Platform and target support

ROCprofiler-SDK now supports the gfx117x, gfx1250-strict, and gfx1250 targets.

##### Usability and diagnostics

* Extra counter YAML files passed with `-E`/`--extra-counters` are validated before they're loaded. A malformed definition now produces an error naming the field that's wrong, instead of a crash or a hang.

* When collecting PMC counters on GPUs from gfx1100 up to but not including gfx1250, ROCprofiler-SDK warns if `power_dpm_force_performance_level` isn't `profile_standard` or can't be read, because dynamic power and clock scaling skew counter values. Set the level with, for example, `amd-smi set --perf-level STABLE_STD`.

* `rocprofv3` now rejects multi-pass counter collection from an input file when combined with `--pid` or `--collection-period`, matching the existing behavior for multiple `--pmc` flags. The error message names what made the run multi-pass.

* Environment variables are no longer written to the rocpd database when their name contains any of `api_key`, `auth`, `bearer`, `cert`, `credential`, `header`, `key`, `password`, `private`, `_pwd`, `secret`, or `token`. Matching is a case-insensitive substring test, so the filter is deliberately broad and excludes some variables that hold nothing sensitive. Each excluded variable is named in a log message, so it's clear what was dropped and why.

* The four tool lifecycle traces in `registration.cpp` are logged at info level instead of warning, so processes that load a ROCprofiler-SDK tool with logging disabled no longer write them to stderr. They remain visible with `ROCPROFILER_LOG_LEVEL=info`.

* `ROCPROF_ABORT_FLUSH_TIMEOUT_SECONDS` is replaced by `ROCPROF_FINALIZE_WAIT_TIMEOUT_SECONDS` (default 10 seconds), which now covers both the abort wait and the wait for child processes during finalization. Scripts that set the old variable must be updated; the old name is no longer read.

* New documentation covers using `rocprofv3` with LLM inference frameworks.

##### Quality and stability improvements

This release includes a range of quality and stability improvements across ROCprofiler-SDK and `rocprofv3`.

* When a context that needs packet interception replaces a queue created by `hsa_amd_queue_create` with a system-memory intercept queue, the device-memory placement flags are now cleared in the caller's descriptor. HIP reads these flags back and previously assumed a device-memory ring buffer that profiling had replaced.

* Fixed `rocprofv3` hanging at normal exit when the application left a long-running child process. Child processes are now waited for only when exiting on a signal, and that wait is time-limited.

* Fixed kernel replay dropping later `--pmc` groups when combined with `--kernel-iteration-range`. The range is evaluated once per logical dispatch and later replay passes reuse that decision.

* Fixed `roctxProfilerPause` and `roctxProfilerResume` with `rocprofv3` process attachment. A pause issued before attaching no longer affects the attached session, and with `--selected-regions` collection starts at the first `roctxProfilerResume` after attaching.

* Fixed `rocprofiler-sdk-rocattach` failing to find symbols such as `rocprofiler_register_detach` when the target library's memory mappings didn't line up one-to-one, in address order, with its loadable segments.

* Fixed attachment failing for framework workloads and Python wheels. Attachment libraries now load on runtime-only installs lacking unversioned symlinks, loading the same library through different paths is no longer treated as a conflict, and a compatible library next to `rocprofiler-register` is preferred over an unrelated installation.

* Fixed output path formatting hanging when a placeholder's value contains that same placeholder, and fixed `%q{NAME}` not expanding, which turned names such as `results_%q{MPI_RANK}` into `results_` on every rank.

* Fixed a second copy of ROCprofiler-SDK bundled with an application such as PyTorch initializing itself under `rocprofv3`, even though all calls go to the globally preloaded library.

* Fixed time-based (microsecond) PC sampling intervals being limited by the maximum intended for cycle-based intervals.

* Fixed `rocprofv3 --att` ignoring `ROCPROF_ATT_LIBRARY_PATH` and searching the whole filesystem from `/`.

* Replaced `__hip_atomic` builtins.

* The synchronous device counting sample no longer crashes at exit, and a HIP event tracing sample was added.

##### ROCm Compute Viewer

`rocprof-compute-viewer` is developed in its own repository and versioned separately from ROCprofiler-SDK. The following changes landed in the same timeframe as this release.

* Disassembly in the ISA view is now colored by instruction category (vector, matrix, scalar ALU, scalar memory, memory, LDS, scratch, branch, control, and wait) in both light and dark themes, making the composition of a kernel readable without reading every mnemonic. Instructions that generate no thread trace token are deliberately left unstyled. Classification looks only at the mnemonic, so an operand or comment containing a category word doesn't change how a line is colored.

* The ISA panel gains a proper column header. Columns can be resized by dragging a divider, restored to automatic sizing by double-clicking it, and hidden individually. Selectors in the header choose the draw type and the annotation category displayed alongside the disassembly.

* Line numbers can be shown next to the disassembly and are remembered in the application configuration across sessions.

* The viewer parses Streaming Performance Monitor data from gfx12 and NAVI4 traces, including the associated derived counters.

* Fixed the global view displaying nothing when a trace was nearly empty except for events.

The project also added `SECURITY.md` and `CONTRIBUTING.md`, adopted the shared ROCm security scanning workflows, and updated its documentation toolchain.

#### ROCm Compute Profiler feature highlights

The following are notable enhancements to the ROCm Compute Profiler (`rocprofiler-compute`):

##### Command-line options update

Profile mode options now match `rocprofv3`, so the same commands work in both tools. Select dispatches with `--kernel-iteration-range`, and set the output directory with `-d`. Dispatch IDs now start at 1, so they line up with the IDs `rocprofv3` reports. Scripts that use the old `-d` or `--dispatch` option need updating. For details, see [Kernel iteration filtering](https://rocm.docs.amd.com/projects/rocprofiler-compute/en/develop/how-to/profile/mode.html#profiling-kernel-iteration-range).

##### Installation improvements

* Analyze mode dependencies now install with a single command, `pip install "rocm-profiler[compute-analyze]"`. Installing them into their own virtual environment keeps them from conflicting with packages your application uses, such as `torch` or `numpy`. Run `rocprof-compute analyze --verify-deps` to check that nothing is missing.

* Python 3.14 is supported for both profiling and analysis.

For details, see [Install the analyze mode dependencies](https://rocm.docs.amd.com/projects/rocprofiler-compute/en/develop/install/core-install.html#analyze-deps).

##### Memory chart improvements

* The memory chart for AMD Instinct™ GPUs has a new layout that is easier to read in the terminal. It breaks down read, write, and atomic requests, shows bandwidth between the L2 cache and the memory fabric, and reports how much traffic goes to high-bandwidth memory (HBM) and to remote memory. On AMD Instinct MI350 Series GPUs, it also shows Local Data Share (LDS) activity and per-channel bandwidth for HBM, xGMI, and PCIe.

* The memory chart for AMD Ryzen™ APUs now reports the traffic measured at each interface, adds LDS utilization, and uses clearer labels, arrows, and legend. Missing counter data appears as N/A instead of a misleading 0.

For details, see [Analyzing with the CLI](https://rocm.docs.amd.com/projects/rocprofiler-compute/en/develop/how-to/analyze/cli.html#cli-memory-chart-viewing).

##### Memory bandwidth analysis (experimental)

On AMD Instinct MI350 Series GPUs, memory bandwidth analysis now shows where memory traffic stalls: at the L1 cache, the L2 cache, or the memory fabric interface. It marks each bottleneck on the memory chart and adds a guidance panel below the chart that explains what was measured and how it affects performance. Collect the data with `--experimental --membw-analysis` at profile time. Analyze mode picks it up automatically. For details, see [Using memory bandwidth analysis](https://rocm.docs.amd.com/projects/rocprofiler-compute/en/develop/how-to/membw_analysis.html).

##### PC sampling improvements

* PC sampling reports two new metrics for each instruction. Active thread percent shows how many threads in a wave were active, so a low value points to branch divergence. Wave occupancy shows how full the GPU was with waves when the sample was taken, and requires stochastic sampling. Both appear in the terminal output, the per-kernel CSV files, and the analysis database.

* Each kernel's results folder is now named after the kernel, so you can find a kernel's results without looking up its ID.

For details, see [PC sampling](https://rocm.docs.amd.com/projects/rocprofiler-compute/en/develop/how-to/pc_sampling.html).

##### PyTorch profiling improvements

* Torch trace (experimental) now ships a prebuilt operator collector for PyTorch 2.13 and 2.14, so it no longer needs to build a tracing library on your machine. The collector records operators on every thread, including those that run during the backward pass. Other PyTorch versions use a reduced trace that misses backward-pass operators.

* If PyTorch and the profiler load different ROCm installations, profiling now stops with a clear message that points to the PyTorch and ROCm installation instructions.

For details, see [Torch trace](https://rocm.docs.amd.com/projects/rocprofiler-compute/en/develop/how-to/profile/mode.html#torch-operator-mapping).

##### Roofline enhancements

* Roofline now supports the MXFP8 datatype on AMD Instinct MI350 Series GPUs, both when measuring peak performance and when analyzing kernels.

* The standalone roofline HTML report includes every datatype the GPU supports, with a switch to move between them, so you no longer need a separate report for each precision. The chart axes are fixed for each GPU, so you can compare reports from two runs side by side.

For details, see [Standalone roofline](https://rocm.docs.amd.com/projects/rocprofiler-compute/en/develop/how-to/profile/mode.html#standalone-roofline) and [Roofline HTML generation](https://rocm.docs.amd.com/projects/rocprofiler-compute/en/develop/how-to/analyze/cli.html#roofline-html-generation).

##### Agent Skills for AI coding assistants

ROCm Compute Profiler now ships Agent Skills for AI coding assistants such as Claude Code, Codex, and Cursor. With the skills installed, your assistant can profile your application, find the slow kernels, and explain why, using speed-of-light, roofline, memory, PC sampling, and Torch trace analysis. Run the included `install-skills.sh` script to add them to your assistant. For details, see [Agent Skills](https://github.com/ROCm/rocm-systems/blob/develop/projects/rocprofiler-compute/skills/README.md).

##### Analyze mode improvements

Analyze mode uses less disk space because it reads the profiling output directly instead of writing a merged copy of all counter data. It prints fewer repeated warnings, so the messages that matter stand out. It also stops right away when profiling data is unreadable, instead of reporting results from incomplete data. For details, see [Analyze mode](https://rocm.docs.amd.com/projects/rocprofiler-compute/en/develop/how-to/analyze/mode.html).

##### Deprecations

The analyze mode `--gui` and `--tui` options are deprecated and will be removed in a future release. Use command-line analysis instead, which prints results to the terminal, saves them as text, CSV files, or an analysis database with `--output-format`, and generates interactive roofline charts as HTML.

For details, see [Analysis output formats](https://rocm.docs.amd.com/projects/rocprofiler-compute/en/develop/how-to/analyze/cli.html#analysis-output-format) and [Roofline HTML generation](https://rocm.docs.amd.com/projects/rocprofiler-compute/en/develop/how-to/analyze/cli.html#roofline-html-generation).

For more information, see the [ROCm Compute Profiler section](#rocm-compute-profiler-3-10-0) in the ROCm component changelogs.

#### ROCm Systems Profiler feature highlights

The following are notable enhancements to ROCm Systems Profiler:

##### Unified `rocsys` command-line entry point

The `rocsys` command is a unified command-line entry point. Running `rocsys -- ./app` or `rocsys profile -- ./app` produces a full trace profile, and `rocsys rewrite` runs `rocprof-sys-instrument` in binary-rewrite mode (`-o`). The other subcommands (`instrument`, `causal`, `avail`, `python`, and `attach`) forward to the existing tools. The `rocprof-sys-*` binaries are unchanged. For details, see [Using rocsys](https://rocm.docs.amd.com/projects/rocprofiler-systems/en/develop/how-to/using-rocsys.html).

##### AMD Infinity Storage I/O telemetry

ROCm Systems Profiler supports AMD Infinity Storage™ technology I/O telemetry. Per-GPU I/O counters are sampled into the profiler output. Enable collection with `ROCPROFSYS_USE_HIPFILE`, and select the metrics to collect with `ROCPROFSYS_HIPFILE_METRICS`. For details, see [hipFile telemetry](https://rocm.docs.amd.com/projects/rocprofiler-systems/en/develop/how-to/hipfile-telemetry.html).

##### Attaching to processes in containerized environments

`rocprof-sys-attach` can now attach to a process running in a containerized environment. For details, see [Attaching to a running process](https://rocm.docs.amd.com/projects/rocprofiler-systems/en/develop/how-to/attaching-to-running-process.html).

##### Agent Skills for AI coding assistants

ROCm Systems Profiler skills are now available in the [AMD Skills](https://github.com/amd/skills) repository. With the skills installed, your AI coding assistant can help you:

* Choose a profiling preset (`choosing-a-profiling-preset`): pick the built-in preset that fits your workload, such as an AI training job, an MPI or HPC application, or a low-overhead production run, and get the exact command to run.

* Discover profiling options (`discovering-profiling-options`): find out which hardware counters or ROCm APIs are available on your machine and which command to use.

* Instrument binaries (`instrumenting-binaries`): instrument an application at runtime or by rewriting the binary, include or exclude functions and modules, preview with a dry run, and balance detail against overhead.

For more information, see the [ROCm Systems Profiler section](#rocm-systems-profiler-1-10-0) in the ROCm component changelogs.

### Control and monitoring tools

#### AMD SMI feature highlights

The following are notable changes to AMD SMI:

##### Power profile management

You can now discover, activate, and configure the driver's predefined power and performance profiles (AMPP, the AMD SMI power profile) from both the C API and the CLI. Profiles, fields, and units are enumerated at runtime, so your tools pick up whatever the driver exposes without hard-coded profile lists.

- **Inspect:** `amd-smi static --ampp` shows each profile's state and fields.
- **Activate and configure:** `amd-smi set --ampp-activate` activates a profile, and `amd-smi set --ampp-configure` sets its field values.
- **Restore:** `amd-smi set --ampp-configure @<path>` reapplies the settings recorded in a saved `amd-smi static --ampp --json` file. It applies only what the file contains.

The existing `amdsmi_*_gpu_power_profile*` preset-mask API is unchanged.

##### Link topology API on bare metal

The new `amdsmi_get_link_topology()` API on bare-metal systems lets you query the link between two GPUs with one call. It returns link weight, status, type, hop count, and framebuffer-sharing data, and matches the existing host interface. Python and Rust bindings are included.

##### Fabric link events in RAS reporting

`amd-smi ras --cper` now reports fabric link events (link down, link up, and fatal) alongside GPU errors, so you can see fabric faults and GPU errors in the same Common Platform Error Record (CPER) output. The new `amdsmi_get_fabric_cper_entries()` API retrieves the fabric reliability, availability, and serviceability (RAS) CPER records programmatically.

##### Energy counter API now available on AMD Radeon GPUs

`amdsmi_get_energy_count` now returns valid data on supported AMD Radeon GPUs in bare-metal and pass-through environments, so you can track cumulative energy use and calculate average power draw between readings on Radeon platforms. Previously, the API was available only on AMD Instinct GPUs.

The API now reports:

- `energy_accumulator`: the cumulative energy counter, sourced from GPU metrics.
- `counter_resolution`: the unit of the counter, so you can convert raw counts to joules.
- `timestamp`: the time of each reading. Divide the change in energy by the change in time between two calls to get power.

##### CLI exit codes and output changes (breaking changes)

The `amd-smi` CLI now returns more specific exit codes, a structured error format, and less N/A output, which can affect scripts that parse its output or check `$?`:

- **Exit codes:** Exit codes now identify the failure type more precisely and match the `code` printed in the error message. `amd-smi ras` also exits non-zero when a CPER file fails to decode, where it previously exited `0`. `set` and `reset` commands continue past a per-device failure instead of stopping.
- **Error messages:** Messages now read `[AMDSMI_STATUS_<name>] <message>`. `--json` and `--csv` errors gain an `error_type` field, and `--csv` rows gain a third column, so parsers that expect two fields per row need updating.
- **Metric output:** `amd-smi metric` omits fields that show N/A in human-readable output. Use `--show-unsupported` to restore them. JSON and CSV output are unchanged.

For API names, CLI syntax, and exit code values, see the [AMD SMI section](#amd-smi-27-2-0) in the ROCm component changelogs.

### Libraries

#### hipBLASLt feature highlights

The following are notable enhancements to hipBLASLt:

##### Complex GEMM support

Scientific and engineering applications can now run single-precision (CGEMM) and double-precision (ZGEMM) complex matrix multiplications on AMD Instinct MI350 Series and MI300 Series GPUs (gfx950 and gfx942). Support covers the `HIP_C_32F` and `HIP_C_64F` data types and transpose and conjugate-transpose combinations of the A and B matrices. Complex GEMM solutions are also available through rocBLAS. Performance tuning for specific shapes is ongoing.

##### Smaller installed package size

Solution selection catalogs are zlib-compressed and installed as `.dat.zlib` files in Debian and RPM packages. hipSPARSELt also loads catalogs in this format. Catalogs are decompressed on first use, which can add initial overhead; subsequent calls are unaffected.

##### Sub-matrix offsets for batched GEMM

General batched GEMM accepts 64-bit element offsets into sub-matrices, so you can run a batched GEMM on a sub-matrix of a larger pre-allocated buffer without copying data. This is available on gfx942, gfx950, and gfx1250.

- **Matrix layout attribute:** To apply an offset, set `HIPBLASLT_MATRIX_LAYOUT_OFFSET` on the matrix layout.
- **Requirements:** Nonzero offsets require `HIPBLASLT_BATCH_MODE_POINTER_ARRAY`. For strided batched GEMM, use the stride instead. A nonzero offset with the sub-byte MX types `HIP_R_6F_E2M3`, `HIP_R_6F_E3M2`, or `HIP_R_4F_E2M1` returns `HIPBLAS_STATUS_NOT_SUPPORTED`.
- **Benchmarking:** `hipblaslt-bench` supports the new `batch_offset_a`, `batch_offset_b`, `batch_offset_c`, and `batch_offset_d` arguments.

For more information, see the hipBLASLt section in the ROCm component changelogs.

#### rocFFT feature highlights

The following are notable enhancements to rocFFT:

##### Just-in-time (JIT) callbacks work with multi-GPU transforms

JIT load and store callbacks, set with `rocfft_plan_description_set_load_callback` and `rocfft_plan_description_set_store_callback`, can be used on multi-GPU transforms that use fields, removing the earlier restriction. Function pointer callbacks set with the deprecated `rocfft_execution_info_set_load_callback` and `rocfft_execution_info_set_store_callback` APIs remain unsupported with fields, so use the JIT callback APIs instead.

##### Very large transforms run with 64-bit indexing only where needed

64-bit index arithmetic is now selected automatically for transforms large enough to overflow 32-bit addressing, so very large transforms, including lengths greater than 2^32, can run. Smaller transforms continue to use 32-bit arithmetic.

For more information, see the rocFFT section in the ROCm component changelogs.

#### rocALUTION feature highlights

The following are notable enhancements to rocALUTION:

##### Interpolation operators and truncation for Ruge-Stueben algebraic multigrid

Select the new matrix-matrix based interpolation operators `MMExtPI` and `MMExtPE` for Ruge-Stueben algebraic multigrid (AMG) through `RugeStuebenAMG::SetInterpolationType()`. To truncate the interpolation operator, use `RugeStuebenAMG::SetInterpolationTruncationFactor()` to drop entries that are small relative to the largest entry in their row, and `RugeStuebenAMG::SetInterpolationMaxElmts()` to cap the number of entries per row.

##### Depth limit for AMG hierarchies

`BaseAMG::SetMaxLevels()` limits the number of levels in an AMG hierarchy. Combine it with another AMG solver as the coarse grid solver to build hybrid hierarchies, such as unsmoothed aggregation on the finest levels followed by Ruge-Stueben AMG on the coarser levels. The new `cg-uaamg-rsamg` sample demonstrates this configuration.

For more information, see the rocALUTION section in the ROCm component changelogs.

#### rocSPARSE feature highlights

The following are notable enhancements to rocSPARSE:

##### Strided batched sparse triangular solve

`rocsparse_sptrsm` now supports strided batched execution, so you can solve independent sparse triangular systems with a single call instead of looping over the batch on the host.

- **Formats:** Batched solves support CSR, CSC, and COO matrices.
- **Broadcast inputs:** A zero batch stride lets you share the sparse matrix or the dense matrices across the batch.
- **Diagonal solves:** Batched execution also supports diagonal-only solves.

##### Generic sort routine

The new `rocsparse_spsort` generic API sorts COO, CSR, and CSC matrices through one routine instead of the format-specific `rocsparse_csrsort`, `rocsparse_cscsort`, and `rocsparse_coosort_by_*` routines. You can sort in place or into a separate target matrix. The routine also supports uniform strided batches, including broadcasting one source matrix across the target batch.

##### Triangular solve and factorization optimizations

rocSPARSE reduces analysis-phase and solve-phase time for sparse triangular solves and incomplete factorizations on AMD Instinct MI300 Series and MI350 Series GPUs (gfx942 and gfx950). This covers `rocsparse_spsv`, `rocsparse_sptrsv`, `rocsparse_spsm`, `rocsparse_sptrsm`, `rocsparse_spilu0`, `rocsparse_spic0`, and `rocsparse_spildlt0`, as well as affected CSR and BSR routines.

- **BSR block sizes:** Solve-phase improvements apply up to block size 32 for `rocsparse_Xbsrsv_solve`, 8 for `rocsparse_Xbsrilu0`, and 16 for `rocsparse_Xbsric0` with at most 128 blocks per block row. Larger block sizes and `rocsparse_Xbsrsm_solve` are unchanged in the solve phase.
- **Smaller gains:** Matrices with fewer than about 1,000 rows per level, such as those with long dependency chains, see smaller gains.

##### SpGEMM optimization for large output rows

rocSPARSE improves sparse matrix-matrix multiplication (SpGEMM) performance for output rows too large for the shared-memory hash tables. Workloads that produce long, wide output rows, such as algebraic multigrid (AMG) setup in rocALUTION, can particularly benefit. This applies to `rocsparse_spgemm` for CSR and BSR matrices and to `rocsparse_Xcsrgemm` and `rocsparse_Xbsrgemm`, including their stage routines such as `rocsparse_csrgemm_nnz`. For BSR matrices with a block dimension greater than 1, only `rocsparse_bsrgemm_nnzb` is affected.

For more information, see the rocSPARSE section in the ROCm component changelogs.

#### rocPRIM improves device scan throughput

rocPRIM device-wide scan operations run faster on AMD Instinct MI325X and MI300X GPUs, increasing throughput particularly for 1-, 2-, and 4-byte integer types.


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

(release-supported-fw)=

## Kernel driver and firmware bundle support

ROCm requires a coordinated stack of compatible firmware, driver, and user-space components. Maintaining version alignment between these layers ensures correct GPU operation and performance, especially for AMD data center products. While AMD publishes the AMD GPU driver and ROCm user space components, your server OEM (original equipment manufacturer) or infrastructure provider distributes the firmware packages. AMD supplies those firmware images (platform level data model (PLDM) bundles), which the OEM integrates and distributes.

```{datatemplate:yaml} /data/driver-firmware-support.yaml
:template: driver-firmware-support-table.md.jinja
```

(release-virtualization-support)=

## GPU virtualization support

AMD Instinct and Radeon GPUs support virtualization in the following configurations. Supported SR-IOV configurations require the AMD GPU Virtualization Driver (GIM) 9.4.0.K. See the [AMD Instinct Virtualization Driver documentation](https://instinct.docs.amd.com/projects/virt-drv/en/mainline-9.4.0.k/) for more information.

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

ROCm 10.2.0 provides optimized support for popular deep learning frameworks and AI inference engines. The following table lists supported frameworks and libraries, their compatible operating systems, and validated versions.

:::{important}
The following table is a general overview of supported frameworks and AI inference engines. Actual support might vary by AMD GPU or APU. Use the {doc}`Compatibility matrix </compatibility/compatibility-matrix>` to verify support for your specific setup.
:::

```{include} ./include/ai-ecosystem-support-table.html
:parser: myst
```

(release-components)=

## ROCm Core SDK components

The following table lists core tools and libraries included in the ROCm {{ ROCM_VERSION }} release.

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

## ROCm known issues

ROCm known issues are noted on {fab}`github` [GitHub](https://github.com/ROCm/TheRock/issues?q=is%3Aissue%20state%3Aopen%20label%3A%22Verified%20Issue%22). These issues will be fixed in a future ROCm release. For known issues related to individual components, review the [ROCm component changelogs](#rocm-component-changelogs).

### HuggingFace model training throughput might regress on AMD Instinct MI350 Series GPUs

HuggingFace model training workloads might see 6–22% lower training throughput on AMD Instinct MI355X and MI350X (gfx950) GPUs, compared to builds that use AOTriton 0.11.2b. Impacted models include BERT, DistilBERT, RoBERTa-large, and GPT-2. As a workaround, rebuild PyTorch and pin AOTriton to version 0.11.2b.

## ROCm resolved issues

The following notable issues have been fixed in ROCm {{ ROCM_VERSION }}.

## ROCm upcoming changes

### LLVM symbolic links

The compiler enables `-DLLVM_ENABLE_PER_TARGET_RUNTIME_DIR=On`, which moves LLVM runtime libraries one level deeper, from `lib/llvm/lib` to `lib/llvm/lib/x86_64-unknown-linux-gnu`, and adds an `x86_64-unknown-linux-gnu` directory to the Clang resource directory containing `compiler_rt` and sanitizer libraries.

To maintain backwards compatibility with older 10.x releases, the following symbolic links are installed:

* Symbolic links in `lib/llvm/lib` for libraries that moved into `lib/llvm/lib/x86_64-unknown-linux-gnu`.
* `x86_64-pc-linux-gnu -> x86_64-unknown-linux-gnu` in the Clang resource directory, so that projects hardcoding `-target x86_64-pc-linux-gnu` continue to function.
* `libclang_rt*` in `lib/llvm/lib/clang/24/lib/linux`, pointing into the `x86_64-unknown-linux-gnu` directory one level up.

These symbolic links will be removed in the next major version of ROCm.
