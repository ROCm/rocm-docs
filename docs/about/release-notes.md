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
- [ROCm known issues](#rocm-known-issues)
- [ROCm resolved issues](#rocm-resolved-issues)
- [ROCm upcoming changes](#rocm-upcoming-changes)

```{note}
Since ROCm 7.14, ROCm uses [TheRock](https://github.com/ROCm/TheRock) as its build and release system. For more information, see the [transition guide](/about/transition-guide-TheRock).
```

## Release highlights

This release adds platform, framework, runtime, profiling, and library enhancements across AMD Instinct™, Radeon™, and Ryzen™ platforms. It expands support for AMD Ryzen processors with integrated Radeon graphics and AMD Instinct GPU virtualization, and updates support for PyTorch, JAX, and vLLM. Profiling tools add device-wide Streaming Performance Monitor (SPM) counter sampling, a unified `rocsys` command for ROCm Systems Profiler, and improved memory analysis in ROCm Compute Profiler. HIP adds new module, library, and kernel APIs. Library updates include complex GEMM support in hipBLASLt and new and improved sparse routines in rocSPARSE.

### Platform and hardware support

This release adds support for additional AMD Ryzen APUs and a new GPU virtualization configuration.

#### Expanded AMD GPU support

ROCm {{ ROCM_VERSION }} adds support for the following AMD APUs and GPUs:

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
- [AMD Radeon PRO W7900D](https://www.amd.com/zh-cn/products/graphics/workstations/radeon-pro/w7900d.html)

For the complete list of supported AMD hardware, see [AMD hardware support](#amd-hardware-support).

#### Operating system support update

ROCm {{ ROCM_VERSION }} adds support for:

* RHEL 10.3 for AMD Instinct and Radeon GPUs.
  * RHEL 10.3 replaces RHEL 10.2 as the validated RHEL 10 release for AMD Radeon GPUs.
* RHEL 9.9 for AMD Instinct and Radeon GPUs.
  * RHEL 9.9 replaces RHEL 9.8 as the validated RHEL 9 release for AMD Radeon GPUs.
* SLES 16.1 for AMD Instinct GPUs.

For the full list of supported Linux distributions, see [Operating system support](#operating-system-support).

#### Installation updates

ROCm {{ ROCM_VERSION }} introduces several improvements to the Runfile Installer:

* Added support for HPC SDK (hipTensor, rocALUTION) component installation.
* Added support for new Linux distributions: SLES 16.1, RHEL 9.9, and RHEL 10.3.
* Improved installation of installer-specific dependencies.
* Fixed additional dependency installation issues for SLES.
* Added support for various Ryzen APUs.
* Improved multi-GPU detection.
* Minor bug fixes.

#### Expanded GPU virtualization support for Instinct GPUs

ROCm {{ ROCM_VERSION }} adds support for the following virtualization configuration on AMD Instinct GPUs:

* On AMD Instinct MI350P:
  * KVM SR-IOV with Ubuntu 24.04 host OS and Ubuntu 24.04 guest OS.

Supported Single Root I/O Virtualization (SR-IOV) configurations require the [AMD GPU Virtualization Driver (GIM) 9.4.0.K](https://github.com/amd/MxGPU-Virtualization/releases/tag/9.4.0.K). For details, see [GPU virtualization support](#gpu-virtualization-support).

#### GPU partitioning support update

GPU partitioning support remains unchanged in this release. For details, see [GPU partitioning support](#gpu-partitioning-support).

### AI inference and frameworks

This release enables support for the following frameworks:

* PyTorch 2.15.0
* JAX 0.11.2
* vLLM 0.32.0
* SGLang 0.5.21

The updated framework support replaces the previous PyTorch 2.12.0, JAX 0.11.0, and vLLM 0.29.0 support.

For details, see [AI ecosystem support](#ai-ecosystem-support).

### Runtimes and compilers

#### HIP feature highlights

The following are notable enhancements to HIP. For the full list, see the [HIP changelog](#hip-7-17-0).

##### Module, library, and kernel APIs added

HIP adds new APIs to give you more control over modules, libraries, and kernels:

* `hipModuleEnumerateFunctions` returns the function handles defined in a loaded module, so applications no longer need to know kernel names ahead of time.
* `hipLibraryGetModule` returns the module handle associated with a library.
* `hipKernelSetAttributeForDevice` sets a device-wide attribute for a kernel on a specific device.

### Profiling and debugging tools

#### ROCprofiler-SDK feature highlights

The following are notable changes to ROCprofiler-SDK and `rocprofv3`. For the full list, see the [ROCprofiler-SDK changelog](#rocprofiler-sdk-1-4-2).

##### Device-wide counter sampling added

Device-wide sampling lets you observe GPU counter behavior throughout a run, even when no kernels are running. The new device counting mode for the Streaming Performance Monitor (SPM) samples counters on a GPU for as long as profiling is active. SPM now supports gfx1200, gfx1201, and gfx1250, and on those GPUs you can set the sample interval in reference-clock cycles.

##### Thread trace reliability and timestamps improved

Thread trace is more reliable for applications that use many GPUs or processes. It now creates queues only on GPUs that load a code object, and its timestamps line up with API traces. SQTT markers and the `rocprof-trace-decoder` Python API now ship with ROCm, so you no longer need to build them from source.

##### Other enhancements

* Improved PC sampling accuracy so fewer samples are dropped or invalid. PC sampling now uses a larger sample buffer on gfx94x, gfx95x, and gfx125x, and `rocprofv3` filters out invalid stochastic PC samples.
* Added support for running your own OpenMP Tools (OMPT) tool alongside ROCprofiler-SDK.

##### ROCm Compute Viewer updates

ROCm Compute Viewer (`rocprof-compute-viewer`) is developed in its own [repository](https://github.com/ROCm/rocprof-compute-viewer) and versioned separately from ROCprofiler-SDK. This release adds SPM data support for gfx1200, gfx1201, and gfx1250 traces, including derived counters. The ISA view gains color coding by instruction category, resizable and hideable columns, and optional line numbers. For details about the ISA view, see [Views and controls](https://rocm.docs.amd.com/projects/rocprof-compute-viewer/en/latest/reference/views.html).

#### ROCm Compute Profiler feature highlights

The following are notable changes to ROCm Compute Profiler (`rocprofiler-compute`). This release includes a breaking change to command-line options. For the full list, see the [ROCm Compute Profiler changelog](#rocm-compute-profiler-3-10-0).

##### Command-line options aligned with rocprofv3 (breaking change)

The dispatch filter and output directory options in profile mode now match `rocprofv3`. If your scripts select dispatches with `-d` or `--dispatch`, update them to use `--kernel-iteration-range`. `-d` now sets the output directory, and dispatch IDs now start at 1. For details, see [Kernel iteration filtering](https://rocm.docs.amd.com/projects/rocprofiler-compute/en/docs-10.2.0/how-to/profile/mode.html#profiling-kernel-iteration-range).

##### Memory chart redesigned and bandwidth analysis added

The memory chart shows more traffic detail on AMD Instinct GPUs and AMD Ryzen AI APUs (gfx115x), and MI350 Series GPUs get experimental bandwidth bottleneck analysis.

* Redesigned the memory chart for AMD Instinct GPUs. It breaks down read, write, and atomic requests and shows L2-to-fabric bandwidth. On MI300 Series and earlier GPUs, it also shows the percentage of traffic to high-bandwidth memory (HBM) and remote memory. On MI350 Series GPUs, it shows Local Data Share (LDS) activity and per-channel HBM, xGMI, and PCIe bandwidth.
* Improved the memory chart for AMD Ryzen AI APUs (gfx115x) with per-interface traffic, LDS utilization, renamed metrics, and updated arrows, labels, and legend.
* Added experimental memory bandwidth analysis on MI350 Series GPUs. It marks bottlenecks at the L1 cache, L2 cache, or memory fabric on the chart and explains each one. Collect the data with `--experimental --membw-analysis`. For details, see [Using memory bandwidth analysis](https://rocm.docs.amd.com/projects/rocprofiler-compute/en/docs-10.2.0/how-to/membw_analysis.html).

##### Roofline supports MXFP8

Roofline analysis now covers MXFP8 and lets you compare precisions and runs side by side.

* Added MXFP8 roofline support on AMD Instinct MI350 Series GPUs, for both peak measurement and kernel analysis.
* Improved the standalone roofline HTML report to include every datatype the GPU supports in one report, with fixed axes so you can compare runs side by side.

For details, see [Standalone roofline](https://rocm.docs.amd.com/projects/rocprofiler-compute/en/docs-10.2.0/how-to/profile/mode.html#standalone-roofline) and [Roofline HTML generation](https://rocm.docs.amd.com/projects/rocprofiler-compute/en/docs-10.2.0/how-to/analyze/cli.html#roofline-html-generation).

##### Other enhancements

* Added a single install command for analyze mode dependencies, `pip install "rocm-profiler[compute-analyze]"`. Use `rocprof-compute analyze --verify-deps` to check that nothing is missing. For details, see [Install the analyze mode dependencies](https://rocm.docs.amd.com/projects/rocprofiler-compute/en/docs-10.2.0/install/core-install.html#analyze-deps).
* Added two PC sampling metrics for each instruction: active thread percent, which points to branch divergence, and wave occupancy, which requires stochastic sampling. For details, see [PC sampling](https://rocm.docs.amd.com/projects/rocprofiler-compute/en/docs-10.2.0/how-to/pc_sampling.html).

##### Deprecations

* Deprecated the analyze mode `--gui` and `--tui` options. They will be removed in a future release. Use command-line analysis instead, which supports text, CSV, and analysis database output. For details, see [Analysis output formats](https://rocm.docs.amd.com/projects/rocprofiler-compute/en/docs-10.2.0/how-to/analyze/cli.html#analysis-output-format).

#### ROCm Systems Profiler feature highlights

The following are notable changes to ROCm Systems Profiler. For the full list, see the [ROCm Systems Profiler changelog](#rocm-systems-profiler-1-10-0).

##### rocsys unified command added

`rocsys` is a single entry point for all ROCm Systems Profiler tools, so you no longer need to call each `rocprof-sys-*` binary separately. The existing binaries are unchanged.

* Run `rocsys -- ./app` or `rocsys profile -- ./app` for a full trace profile.
* Run `rocsys rewrite` to instrument a binary in binary-rewrite mode.
* Use the `instrument`, `causal`, `avail`, `python`, and `attach` subcommands to run the existing tools.

For details, see [Using rocsys](https://rocm.docs.amd.com/projects/rocprofiler-systems/en/docs-10.2.0/how-to/using-rocsys.html).

##### Other enhancements

* Added AMD Infinity Storage technology I/O telemetry. Per-GPU I/O counters are sampled into the profiler output. Enable it with `ROCPROFSYS_USE_HIPFILE`. For details, see [hipFile telemetry](https://rocm.docs.amd.com/projects/rocprofiler-systems/en/docs-10.2.0/how-to/hipfile-telemetry.html).
* Added support for `rocprof-sys-attach` to attach to processes running in containers. For details, see [Attaching to a running process](https://rocm.docs.amd.com/projects/rocprofiler-systems/en/docs-10.2.0/how-to/attaching-to-running-process.html).

### Control and monitoring tools

#### AMD SMI feature highlights

The following are notable changes to AMD SMI. For the full list, see the [AMD SMI changelog](#amd-smi-27-2-0).

##### AMD SMI Python library on PyPI

The AMD SMI Python library is now on PyPI as `amdsmi` 27.2.0; you can install it
using `pip install amdsmi`. This PyPI package bundles the AMD SMI library, so using
the Python API doesn't require a ROCm installation. It provides the Python library only;
the `amd-smi` command-line tool ships with ROCm. `amdsmi` releases on PyPI follow the
AMD SMI library version (27.2.0 as of ROCm 10.2), not the ROCm SDK version.

Requirements:

- Linux x86_64 with glibc 2.27 or later
- Python 3.6 or later and pip 20.3 or later (older pip versions install the unrelated legacy 7.0.2 release instead)
- [The amdgpu driver](#release-supported-fw)
- `libdrm-amdgpu1` (Ubuntu/Debian), `libdrm` (RHEL), or `libdrm_amdgpu1` (SLES)

See [https://pypi.org/project/amdsmi/](https://pypi.org/project/amdsmi/) for more information.

##### Power profile management added

You can now discover, activate, and configure the driver's predefined power and performance profiles from both the C API and the CLI. AMD SMI calls them AMPP (AMD SMI power profile) and uses `ampp` in API and CLI names, for example `amd-smi static --ampp`. Profiles and fields are enumerated at runtime, so your tools pick up whatever the driver exposes without hard-coded profile lists.

##### Link topology API added for bare metal

The new `amdsmi_get_link_topology()` API lets you query the link between two GPUs on bare-metal systems with one call, matching the existing host interface. Python and Rust bindings are included.

##### Fabric link events added to error reports

`amd-smi ras --cper` now reports fabric link events alongside GPU errors in the same Common Platform Error Record (CPER) output, so you can see fabric faults and GPU errors together. The new `amdsmi_get_fabric_cper_entries()` API retrieves the fabric reliability, availability, and serviceability (RAS) CPER records programmatically.

##### Energy counter API supports AMD Radeon GPUs

`amdsmi_get_energy_count` now returns valid data on supported AMD Radeon GPUs in bare-metal and pass-through environments, so you can track energy use and calculate average power draw. Previously, the API was available only on AMD Instinct GPUs.

##### CLI exit codes and output updated (breaking changes)

The `amd-smi` CLI now returns more specific exit codes, a structured error format, and less N/A output, which can affect scripts that parse its output or check `$?`:

- **Exit codes:** Exit codes now identify the failure type more precisely and match the `code` printed in the error message. `amd-smi ras` also exits non-zero when a CPER file fails to decode, where it previously exited `0`. `set` and `reset` commands continue past a per-device failure instead of stopping.
- **Error messages:** Errors are now structured, and `--json` and `--csv` errors gain an `error_type` field. `--csv` rows gain a third column, so parsers that expect two fields per row need updating.
- **Metric output:** `amd-smi metric` omits fields that the GPU's metrics table version can't report, so human-readable output shows fewer N/A values. On AMD Instinct MI300X, MI325X, and MI355X GPUs, `--power` no longer prints `THROTTLE_STATUS`, and `--usage` no longer prints `MM_ACTIVITY`, `VCN_ACTIVITY`, or `JPEG_ACTIVITY`. Use `--show-unsupported` to restore omitted fields. JSON and CSV output are unchanged.

For API names, CLI syntax, and exit code values, see the [AMD SMI changelog](#amd-smi-27-2-0).

### Libraries

#### hipBLASLt feature highlights

The following are notable enhancements to hipBLASLt. For the full list, see the [hipBLASLt changelog](#hipblaslt-1-5-0).

##### Complex GEMM support added

Scientific and engineering applications can now run single-precision (CGEMM) and double-precision (ZGEMM) complex matrix multiplications on AMD Instinct MI350 Series and MI300 Series GPUs (gfx950 and gfx942). Support covers the `HIP_C_32F` and `HIP_C_64F` data types and transpose and conjugate-transpose combinations of the A and B matrices. Complex GEMM solutions are also available through rocBLAS.

##### Solution catalogs stay compressed on disk

hipBLASLt solution catalogs now stay compressed on disk as `.dat.zlib` files, instead of being compressed only in the package. For example, the gfx942 catalogs use about 194 MB on disk instead of about 3.8 GB. A catalog is decompressed on first use, which can add initial overhead. Subsequent calls are unaffected. hipSPARSELt also loads catalogs in this format.

##### Sub-matrix offsets supported for batched GEMM

General batched GEMM now accepts 64-bit element offsets, so you can run GEMM on a smaller sub-matrix of a larger pre-allocated matrix. This also improves performance because the device pointer array no longer needs to be copied back and forth between the host and the device. This is available on gfx942, gfx950, and gfx1250.

- **Matrix layout attribute:** To apply an offset, set `HIPBLASLT_MATRIX_LAYOUT_OFFSET` on the matrix layout.
- **Requirements:** Nonzero offsets require `HIPBLASLT_BATCH_MODE_POINTER_ARRAY`. For non-batched and strided batched GEMM, apply the offset directly through the strided batch offset instead. A nonzero offset with the sub-byte MX types `HIP_R_6F_E2M3`, `HIP_R_6F_E3M2`, or `HIP_R_4F_E2M1` returns `HIPBLAS_STATUS_NOT_SUPPORTED`.
- **Benchmarking:** `hipblaslt-bench` supports the new `batch_offset_a`, `batch_offset_b`, `batch_offset_c`, and `batch_offset_d` arguments.

#### hipFile feature highlights

The following are notable enhancements to hipFile. For the full list, see the [hipFile changelog](#hipfile-0-6-0).

##### Asynchronous I/O failover and new documentation

hipFile adds failover for asynchronous fastpath I/O, a batch I/O example, and Stats API documentation.

- **Async fastpath failover:** Asynchronous fastpath I/O now switches to the fallback I/O backend when the fastpath reports a device error (`ENODEV` or `EREMOTEIO`), instead of failing the request. Synchronous and batch I/O already behaved this way.
- **Batch round-trip example:** A new `batch-roundtrip` example shows how to use the batch I/O API.
- **Stats API documentation:** The new [Statistics API reference](https://rocm.docs.amd.com/projects/hipFile/en/docs-10.2.0/reference/api-stats.html) covers how to read I/O statistics.

##### PyTorch adds AMD Infinity Storage hipFile support

PyTorch for ROCm now supports the `torch.cuda.gds` file I/O APIs, including `GdsFile`, `gds_register_buffer`, and `gds_deregister_buffer`, through AMD Infinity Storage hipFile. On supported filesystems, data moves directly between storage and GPU memory instead of being staged through host memory. To check whether your PyTorch build includes this support, use `torch.cuda.gds.is_available()`.

##### hipFile support added to fio and IOR

hipFile support is now available in [fio 3.43](https://github.com/axboe/fio/releases/tag/fio-3.43) and in the main branch of [IOR](https://github.com/hpc/ior/pull/540). Both are open source storage benchmarking tools that you can use to evaluate storage performance.

#### rocFFT feature highlights

The following are notable enhancements to rocFFT. For the full list, see the [rocFFT changelog](#rocfft-1-0-41).

##### Just-in-time callbacks supported on multi-GPU transforms

Just-in-time (JIT) load and store callbacks, set with `rocfft_plan_description_set_load_callback` and `rocfft_plan_description_set_store_callback`, can now be used on multi-GPU transforms that use fields.

Function pointer callbacks set with the deprecated `rocfft_execution_info_set_load_callback` and `rocfft_execution_info_set_store_callback` APIs can no longer be used on transforms that use fields. For these transforms, `rocfft_execute` returns `rocfft_status_invalid_arg_value`. Use the JIT callback APIs instead.

##### 64-bit indexing selected automatically for large transforms

64-bit index arithmetic is now selected automatically for transforms large enough to overflow 32-bit addressing, so very large transforms, including lengths greater than 2^32, can run. Smaller transforms continue to use 32-bit arithmetic.

#### rocSPARSE feature highlights

The following are notable enhancements to rocSPARSE. For the full list, see the [rocSPARSE changelog](#rocsparse-5-1-0).

##### Strided batched sparse triangular solve supported

`rocsparse_sptrsm` now supports strided batched execution, so you can solve independent sparse triangular systems with a single call instead of looping over the batch on the host.

- **Formats:** Batched solves support CSR, CSC, and COO matrices.
- **Broadcast inputs:** A zero batch stride lets you share the sparse matrix or the dense matrices across the batch.
- **Diagonal solves:** Batched execution also supports diagonal-only solves.

##### Generic sort routine added

The new `rocsparse_spsort` generic API sorts COO, CSR, and CSC matrices through one routine instead of the format-specific `rocsparse_csrsort`, `rocsparse_cscsort`, and `rocsparse_coosort_by_*` routines. You can sort in place or into a separate target matrix. The routine also supports uniform strided batches, including broadcasting one source matrix across the target batch.

##### Performance improvements

* Improved sparse triangular solve and incomplete factorization performance on AMD Instinct MI300 Series and MI350 Series GPUs (gfx942 and gfx950), in both the analysis and solve phases. This covers routines such as `rocsparse_spsv`, `rocsparse_spsm`, `rocsparse_spilu0`, and `rocsparse_spic0`. Matrices with long dependency chains see smaller gains.
* Improved SpGEMM performance for products with long, wide output rows. Workloads such as algebraic multigrid (AMG) setup in rocALUTION can particularly benefit.

### HPC SDK highlights

For more information about the HPC SDK, see {doc}`/components/hpc-sdk` and {doc}`/install/hpc-sdk`.

#### New documentation

hipTensor and rocALUTION now have standalone documentation sites:

- [hipTensor documentation](https://rocm.docs.amd.com/projects/hipTensor/en/docs-10.2.0/)
- [rocALUTION documentation](https://rocm.docs.amd.com/projects/rocALUTION/en/docs-10.2.0/)

#### rocALUTION feature highlights

The following are notable enhancements to rocALUTION:

##### Ruge-Stueben interpolation and truncation added

Ruge-Stueben algebraic multigrid (AMG) now supports the matrix-matrix-based interpolation operators `MMExtPI` and `MMExtPE`. To select one, use `RugeStuebenAMG::SetInterpolationType()`. To truncate the interpolation operator, use `RugeStuebenAMG::SetInterpolationTruncationFactor()` to drop entries that are small relative to the largest entry in their row, and `RugeStuebenAMG::SetInterpolationMaxElmts()` to cap the number of entries per row.

##### Depth limit for AMG hierarchies

The new `BaseAMG::SetMaxLevels()` limits the number of levels in an AMG hierarchy. Combine it with another AMG solver as the coarse grid solver to build hybrid hierarchies, such as unsmoothed aggregation on the finest levels followed by Ruge-Stueben AMG on the coarser levels. The new `cg-uaamg-rsamg` sample demonstrates this configuration.

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

(release-supported-fw)=

## Kernel driver and firmware bundle support

ROCm requires a coordinated stack of compatible firmware, driver, and user-space components. Maintaining version alignment between these layers ensures correct GPU operation and performance, especially for AMD data center products. While AMD publishes the AMD GPU driver and ROCm user space components, your server OEM (original equipment manufacturer) or infrastructure provider distributes the firmware packages. AMD supplies those firmware images (platform level data model (PLDM) bundles), which the OEM integrates and distributes.

```{datatemplate:yaml} /data/driver-firmware-support.yaml
:template: driver-firmware-support-table.md.jinja
```

(release-virtualization-support)=

## GPU virtualization support

AMD Instinct and Radeon GPUs support virtualization in the following configurations. Supported SR-IOV configurations require the AMD GPU Virtualization Driver (GIM) 9.4.0.K. See the [AMD Instinct Virtualization Driver documentation](https://instinct.docs.amd.com/projects/virt-drv/en/latest/) for more information.

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

ROCm {{ ROCM_VERSION }} provides optimized support for popular deep learning frameworks and AI inference engines. The following table lists supported frameworks and libraries, their compatible operating systems, and validated versions.

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

(wsl2-illegal-memory-access)=
### AI workloads might fail with an illegal memory access error on AMD Ryzen APUs under WSL2

AI workloads running under WSL2 with ROCm 10.2.0 on AMD Ryzen APUs (gfx1150, gfx1151, gfx1152, gfx1103, and gfx1153) might fail with an illegal memory access error. The failure terminates the workload and has been observed with PyTorch and llama.cpp.

Observed error messages include:

```
ROCm error: an illegal memory access was encountered
torch.AcceleratorError: CUDA error:
an illegal memory access was encountered
HSA_STATUS_ERROR_MEMORY_APERTURE_VIOLATION:
The agent attempted to access memory beyond the largest legal address.
GPU fence timeout after 30000ms — possible device fault
```

As a workaround, use ROCm 10.1.0 or the latest ROCm 10.3.0 nightly build. See [GitHub issue #10097](https://github.com/ROCm/TheRock/issues/10097).

### ComfyUI workloads might show lower performance on some AMD Radeon GPUs and Ryzen APUs

ComfyUI-based AI workloads using the AOTriton backend might show lower performance on some AMD Radeon GPUs and Ryzen APUs, such as AMD Radeon RX 7900 series GPUs and Ryzen AI Max 300 Series (gfx1151) APUs. As a workaround, ComfyUI users can use an alternative attention backend, for example `--use-flash-attention`. For more details on choosing the best attention backends for ComfyUI image and video generation workloads, see [Understanding Attention Algorithms and Their Backends for Image and Video Generation](https://rocm.blogs.amd.com/software-tools-optimization/comfyui-fa-backends/README.html). See [GitHub issue #10098](https://github.com/ROCm/TheRock/issues/10098).

### Hugging Face model training throughput might regress on AMD Instinct MI350 Series GPUs

Hugging Face model training workloads might see lower training throughput on AMD Instinct MI355X and MI350X (gfx950) GPUs, compared to builds that use AOTriton 0.11.2b. Impacted models include BERT, DistilBERT, RoBERTa-large, and GPT-2. As a workaround, rebuild PyTorch and pin AOTriton to version 0.11.2b.

```{note}
This issue is partially addressed in ROCm 10.2.0 for some Hugging Face models. See the [ROCm 10.2.0 resolved issues](hugging-face-model-training-throughput-for-specific-model-is-restored-on-amd-instinct-mi350x) entry for details.
```

See [GitHub issue #10099](https://github.com/ROCm/TheRock/issues/10099).

### rocHPL throughput might regress on AMD Instinct MI325X GPUs

rocHPL workloads running on AMD Instinct MI325X GPUs (gfx942) in an 8-GPU configuration might see lower throughput compared to previous ROCm releases, dropping from roughly 570,000 GFLOPS to 463,000 GFLOPS. As a workaround, set `ROCBLAS_USE_HIPBLASLT=0` to use the Tensile backend instead of hipBLASLt for rocHPL:

```
ROCBLAS_USE_HIPBLASLT=0 <rocHPL command>
```

See [GitHub issue #10100](https://github.com/ROCm/TheRock/issues/10100).

### GPU Mode2 reset might time out on AMD Instinct MI300A GPUs

When a workload triggers a hardware hang or RAS error on AMD Instinct MI300A GPUs (gfx942), the amdgpu Mode2 reset used to recover the GPU might time out with `failed to send mode2 reset, error code -62`. After the timeout, the SMU stops responding, and subsequent GPU management calls fail with `amdsmi_init() timed out after 60s. The GPU driver may be unresponsive.` See [GitHub issue #10101](https://github.com/ROCm/TheRock/issues/10101).

### AMD Instinct MI300A systems might reboot spontaneously on RHEL 10.3

AMD Instinct MI300A systems might reboot spontaneously on Linux kernels that enable the `X86_FEATURE_INVLPGB` broadcast TLB flush, including RHEL 10.3. This happens because the `TLBSYNC` instruction used by the broadcast TLB flush sometimes hangs on MI300A, triggering a CPU machine check watchdog that sync-floods the data fabric. As a workaround, add `clearcpuid=419` to the kernel command line. See [GitHub issue #10102](https://github.com/ROCm/TheRock/issues/10102).

### CRIU restore fails on AMD Instinct MI350X GPUs

CRIU restore fails for GPU workloads on AMD Instinct MI350X GPUs. When `criu restore` is called for any workload that creates a GPU queue, the `AMDKFD_IOC_CRIU_OP` restore ioctl returns `Invalid argument` and restore exits with:

```
amdgpu_plugin: Restore ioctl failed: Invalid argument
amdgpu_plugin: Failed to restore (ret:-1)
Restoring FAILED.
```

Checkpoint operations (`criu dump`) complete successfully; only restore is affected. No workaround is available. See [GitHub issue #10103](https://github.com/ROCm/TheRock/issues/10103).

### An in-place amdgpu-dkms upgrade might fail to load the new driver on Linux

When upgrading `amdgpu-dkms` in place without rebooting, `modprobe -r amdgpu` does not unload the full module stack, and the old `amdttm` and `amdkcl` modules stay resident in memory. Loading the new driver then fails with `modprobe: ERROR: could not insert 'amdgpu': Invalid argument` because `amddrm_ttm_helper` rejects stale symbol versions exported by the leftover modules.

As a workaround, unload all remaining amdgpu-related modules before reinstalling the driver:

```
sudo modprobe -r amdgpu amdttm
lsmod | grep -E '^(amdgpu|amdxcp|amddrm|amdttm|amd_sched|amdkcl)'
```

If any modules remain listed, unload each with `sudo modprobe -r <module_name>`, remove the old driver packages, install the new packages, and load the driver with `sudo modprobe amdgpu`. See [GitHub issue #10104](https://github.com/ROCm/TheRock/issues/10104).

### AGFHC rocTwitch and Sprites subtests might fail on AMD Instinct GPUs

The AGFHC Sprites subtest might fail with `TEST_GFX_SPRITES_FAIL`, reporting `NO_FREE_RESOURCE (0x5)` from the embers allocator and terminating `firexs2` with return code 255. This failure occurs only when the rocTwitch and Sprites subtests run together; running either subtest independently completes successfully. See [GitHub issue #10105](https://github.com/ROCm/TheRock/issues/10105).

### RCCL communicator initialization fails when each rank is bound to a single GPU

In RCCL jobs where each rank sees only one GPU — for example, with Slurm's `--gpu-bind=closest`, or per-rank `HIP_VISIBLE_DEVICES` or `ROCR_VISIBLE_DEVICES` settings — communicator initialization fails. `ncclCommInitRank` returns `ncclInternalError`, which applications and rccl-tests report as "internal error - please report this issue to the NCCL developers". Jobs where each rank sees two or more GPUs, or all GPUs on the node, are not affected.

As a workaround, let every rank see all GPUs on the node. The application is then responsible for selecting the GPU closest to its CPU binding; applications that do not will lose CPU–GPU locality. Alternatively, rebuild RCCL from a source revision that contains the fix and preload it ahead of the packaged library, which preserves the existing binding. See [GitHub issue #10106](https://github.com/ROCm/TheRock/issues/10106).

### VCN decode might hang on AMD Instinct MI325X GPUs

Video decode workloads using rocDecode on AMD Instinct MI325X GPUs might hang intermittently when decode bursts are separated by idle gaps of approximately one second or longer. This happens because each idle gap triggers a VCN power-gate cycle; on power-up, the VCN firmware might read the ring-buffer size register before the hardware restores it, reading 0 and preventing submitted commands from completing. Only the affected VCN instance stalls; the remaining VCN instances on the GPU continue operating. In distributed training jobs that decode video in the input pipeline, the stalled rank stops progressing and the job fails with a collective-communication timeout. This issue will be resolved in a VCN firmware update for an upcoming Best Known Configuration (BKC) release. See [GitHub issue #10107](https://github.com/ROCm/TheRock/issues/10107).

### rocdecode VP9 video decoding might cause a VCN ring timeout and GPU reset on AMD Instinct MI350 Series GPUs

Decoding VP9 video clips using `rocdecode` on AMD Instinct MI350 Series (gfx950) GPUs might cause the VCN video-decode engine to hang with a ring timeout. The ring reset fails, triggering a full GPU mode 1 reset. This occurs because of a regression in the VCN firmware. This issue will be resolved in a VCN firmware update for an upcoming Best Known Configuration (BKC) release. See [GitHub issue #10108](https://github.com/ROCm/TheRock/issues/10108).

### Packed FP32 instructions might produce unexpected data on AMD Instinct MI350X GPUs
 
On AMD Instinct MI350X GPU (gfx950), kernels that use `v_pk_add_f32`, `v_pk_mul_f32`, or `v_pk_fma_f32` instructions with certain `op_sel` encodings can produce unexpected results. As a workaround, the LLVM compiler corrects these encodings by commuting operands for packed ADD/MUL and splitting packed FMA into two scalar `v_fma_f32` instructions. The assembler also rejects invalid `op_sel` values in hand-written assembly. Kernels compiled with an earlier ROCm release that contain the affected encodings must be recompiled to benefit from this workaround. See [GitHub issue #10109](https://github.com/ROCm/TheRock/issues/10109).

## ROCm resolved issues

The following notable issues have been fixed in ROCm {{ ROCM_VERSION }}.

### RAS error injection and query are unavailable on AMD Instinct MI350P GPUs

Previously, the RAS error injection and query interfaces (used to validate error-handling paths, for example with the amdgpuras tool) were not supported on AMD Instinct MI350P GPUs. Injection commands failed, and query commands reported that the block didn't support querying. This affected only manual RAS error injection and query tooling. Normal GPU reliability features such as ECC detection and error recovery during regular operation were not affected. See [GitHub issue #8794](https://github.com/ROCm/TheRock/issues/8794).

### Valgrind workloads might stop responding on WSL2 with high system RAM

Previously, running workloads under Valgrind on WSL2 might stop responding during ROCm runtime initialization on AMD Radeon GPUs, such as the AMD Radeon RX 9060, on systems with a large amount of system RAM. This happened because WSL reserved GPU address space upfront in a way that could collide with Valgrind's reserved memory range as system RAM grows. Workloads that don't use Valgrind are unaffected. See [GitHub issue #8793](https://github.com/ROCm/TheRock/issues/8793).

### Concurrent rocprofv3 profiling causes node reset on AMD Instinct MI300A GPUs

Previously, running `rocprofv3 --pmc` or `--kernel-trace` concurrently across multiple GPUs of a single AMD Instinct MI300A node might hard-reset the node, terminating all co-resident jobs and requiring a manual power cycle to recover. See [GitHub issue #8229](https://github.com/ROCm/TheRock/issues/8229).

### Hugging Face model training throughput for specific models is restored on AMD Instinct MI350X

ROCm 10.2.0 restores Hugging Face model training throughput to previous levels for BART, GPT-2, and Llama 2 70B Chat workloads on AMD Instinct MI350X (gfx950) GPUs, recovering the throughput loss reported in the ROCm 10.1.0 known issues. See [GitHub issue #8878](https://github.com/ROCm/TheRock/issues/8878).

```{note}
A residual performance gap remains on some models. See the [ROCm 10.2.0 known issues](hugging-face-model-training-throughput-might-regress-on-amd-instinct-mi350-series-gpus) for current status and workarounds.
```

## ROCm upcoming changes

### LLVM symbolic links

The compiler enables `-DLLVM_ENABLE_PER_TARGET_RUNTIME_DIR=On`, which moves LLVM runtime libraries one level deeper, from `lib/llvm/lib` to `lib/llvm/lib/x86_64-unknown-linux-gnu`, and adds an `x86_64-unknown-linux-gnu` directory to the Clang resource directory containing `compiler_rt` and sanitizer libraries.

To maintain backwards compatibility with older 10.x releases, the following symbolic links are installed:

* Symbolic links in `lib/llvm/lib` for libraries that moved into `lib/llvm/lib/x86_64-unknown-linux-gnu`.
* `x86_64-pc-linux-gnu -> x86_64-unknown-linux-gnu` in the Clang resource directory, so that projects hardcoding `-target x86_64-pc-linux-gnu` continue to function.
* `libclang_rt*` in `lib/llvm/lib/clang/24/lib/linux`, pointing into the `x86_64-unknown-linux-gnu` directory one level up.

These symbolic links will be removed in the next major version of ROCm.
