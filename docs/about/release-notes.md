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

ROCm {{ ROCM_VERSION }} adds support for:

* RHEL 10.3 for AMD Instinct and Radeon GPUs.
* RHEL 9.9 for AMD Instinct and Radeon GPUs.
* SLES 16.1 for AMD Instinct GPUs.

For the full list of supported Linux distributions, see [Operating system support](#operating-system-support).

#### Expanded GPU virtualization support for Instinct and Radeon GPUs

ROCm {{ ROCM_VERSION }} adds support for the following virtualization configurations on AMD Instinct and Radeon GPUs:

**Instinct**

* On AMD Instinct MI350P:
  * KVM SR-IOV with Ubuntu 24.04 host OS and Ubuntu 24.04 guest OS.

**Radeon**

* On AMD Radeon PRO W7800, W7900, and W7900DS:
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

The new `HIP_TARGET_API_VERSION` macro lets you select the target HIP API version at compile time, which helps maintain source compatibility for applications written against older API versions. For example, when targeting HIP API version `800` or later, `hipMemAdvise` aliases `hipMemAdvise_v2`, which takes a `hipMemLocation` structure instead of a device ID. Applications using the legacy signature might need source updates. To keep the previous signature, define `HIP_TARGET_API_VERSION` to a nonzero value less than 800 at compile time, such as `700` for HIP 7.0 or `717` for HIP 7.17.

##### New module, library, and kernel APIs

New APIs provide more control over modules, libraries, and kernels. `hipModuleEnumerateFunctions` returns the function handles defined in a loaded module, so applications no longer need to know kernel names ahead of time. `hipLibraryGetModule` returns the module handle associated with a library, and `hipKernelSetAttributeForDevice` sets a device-wide attribute for a kernel on a specific device.

##### Device-side fault reporting on WSL 2

HIP now reports errors for device-side out-of-bounds writes, null pointer dereferences, and asserts on WSL 2, rather than leaving affected applications hanging.

For more information, see the [HIP section](#hip-10-2-0) in the ROCm component changelogs.

### Profiling and debugging tools

```{note}
ROCm Compute Profiler and ROCm Systems Profiler don't support WSL 2, although the ROCm Core SDK supports WSL 2 for other components. To profile an application, use a native Linux system.
```

#### ROCprofiler-SDK feature highlights

The following are notable enhancements to ROCprofiler-SDK:

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

##### Node power management

Node power management (NPM) now lets you set a node-wide power limit in watts, read the node's current power draw, and choose the node's balancing mode, so you can keep an AMD Instinct MI455X GPU node within its power budget without setting a separate limit for each GPU. This is available on bare-metal Linux through the C API and the `amd-smi` CLI.

- **Set the limit and read node power:** Use `amdsmi_set_npm_limit()` or `amd-smi set --node-power-limit` to apply a limit across the whole node. Setting the limit requires root or `CAP_SYS_ADMIN` privileges. `amdsmi_get_npm_info()` and `amd-smi node -p` report the node's current power, queried once per node rather than once per GPU.
- **Choose a balancing mode:** Use `amdsmi_set_npm_balancing_mode()` or `amd-smi set --node-balancing-mode` to switch between power balancing (the default) and frequency balancing. Setting the mode requires elevated privileges and NPM to be enabled.

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
- **Error messages:** Messages now read `[AMDSMI_STATUS_<name>] <message>`, and `--json` and `--csv` errors gain new fields.
- **Metric output:** `amd-smi metric` omits fields that show N/A in human-readable output. Use `--show-unsupported` to restore them. JSON and CSV output are unchanged.

For more information, see the AMD SMI section in the ROCm component changelogs.

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

## ROCm known issues

ROCm known issues are noted on {fab}`github` [GitHub](https://github.com/ROCm/TheRock/issues?q=is%3Aissue%20state%3Aopen%20label%3A%22Verified%20Issue%22). These issues will be fixed in a future ROCm release. For known issues related to individual components, review the [ROCm component changelogs](#rocm-component-changelogs).

## ROCm resolved issues

The following notable issues have been fixed in ROCm {{ ROCM_VERSION }}.

## ROCm upcoming changes

### LLVM symbolic links

The compiler is enabling `-DLLVM_ENABLE_PER_TARGET_RUNTIME_DIR=On`. This option moves LLVM runtime libraries one level deeper, from `lib/llvm/lib` to `lib/llvm/lib/x86_64-unknown-linux-gnu`, and adds an `x86_64-unknown-linux-gnu` directory to the Clang resource directory containing `compiler_rt` and sanitizer libraries.

To maintain backwards compatibility with older 10.x releases, symbolic links are installed in `lib/llvm/lib` for libraries that moved into `lib/llvm/lib/x86_64-unknown-linux-gnu`. A symbolic link `x86_64-pc-linux-gnu -> x86_64-unknown-linux-gnu` is also added to the Clang resource directory so that projects hardcoding `-target x86_64-pc-linux-gnu` continue to function.

These symbolic links will be removed in the next major version of ROCm.
