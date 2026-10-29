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

ROCm {{ ROCM_VERSION }} adds support for the following AMD Ryzen APUs:

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

Virtualization support remains unchanged in this release. For the full list of supported Linux distributions, see [Operating system support](#operating-system-support).

#### Expanded GPU virtualization support for Instinct and Radeon GPUs

Virtualization support remains unchanged in this release. For details, see [GPU virtualization support](#gpu-virtualization-support).

#### GPU partitioning support update

GPU partitioning support remains unchanged in this release. For details, see [GPU partitioning support](#gpu-partitioning-support).

### AI inference and frameworks

Framework support remains unchanged in this release. For details, see [AI ecosystem support](#ai-ecosystem-support).

### Runtime and compilers

#### HIP feature highlights

The following are notable enhancements to HIP:

##### HIP preserves location of imported virtual memory handles

`hipMemGetAllocationPropertiesFromHandle` now correctly reports `hipMemLocationTypeHost` for host-backed allocations imported with `hipMemImportFromShareableHandle`, so applications can reliably identify whether an imported allocation resides in host or device memory. For device-backed allocations, the API reports the owning device.

##### HIP adds module and library management APIs

HIP adds APIs for working with loaded modules and libraries. `hipModuleEnumerateFunctions` returns the function handles defined in a loaded module, so applications no longer need to know kernel names ahead of time. `hipLibraryGetModule` returns the module handle associated with a library, and `hipKernelSetAttributeForDevice` sets a device-wide attribute for a kernel on a specific device.

##### HIP adds HIP_TARGET_API_VERSION for source compatibility

The new `HIP_TARGET_API_VERSION` macro lets you select the HIP API version at compile time, which helps maintain source compatibility with applications built against older HIP API versions. For example, `hipMemAdvise` now aliases `hipMemAdvise_v2`, which takes a `hipMemLocation` structure instead of a device ID, so applications that use the legacy signature might require source updates. To keep the previous signature, define `HIP_TARGET_API_VERSION` to a nonzero value less than 800 at compile time, such as `700` for HIP 7.0 or `717` for HIP 7.17.

##### HIP reports device-side faults on WSL 2

HIP on WSL 2 now prevents indefinite application hangs and reports errors for device-side faults, including out-of-bounds writes, null pointer dereferences, and device asserts.

For more information, see the [HIP section](#hip-10-2-0) in the ROCm component changelogs.

### Profiling and debugging tools

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

For more information, see the ROCm Systems Profiler section in the ROCm component changelogs.

### Control and monitoring tools

#### AMD SMI feature highlights

The following are notable changes to AMD SMI:

### Libraries

#### rocPRIM improves device scan throughput (ROCM-28668)

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

The following notable issues have been fixed in ROCm 10.1.0.
