#### **AMD SMI** (27.1.0)

##### Added

- **Exposed `BOOT_FIRMWARE` field in `amd-smi static --ifwi` output**.
  - The `boot_firmware` value returned by `amdsmi_get_gpu_vbios_info()` now appears under the `IFWI` section alongside `NAME`, `BUILD_DATE`, `PART_NUMBER` and `VERSION` (`--vbios` remains available as a legacy alias).

- **An experimental, opt-in WSL (WDDM/dxg) GPU backend**.
  - Built only with `-DENABLE_WSL_BACKEND=ON` (off by default); native builds and packages are unchanged.
  - Reads GPU telemetry through `librocdxg`; queries with no WDDM equivalent return `AMDSMI_STATUS_NOT_SUPPORTED`. See [Using AMD SMI under WSL](https://rocm.docs.amd.com/projects/amdsmi/en/latest/how-to/amdsmi-wsl-mode.html).

- **UALoE-backed physical accelerator ID and tray info**.
  - `physical_acc_id` added to `amdsmi_asic_info_t` and `amdsmi_enumeration_info_t`, populated by `amdsmi_get_gpu_asic_info()` and `amdsmi_get_gpu_enumeration_info()`.
  - New node-scoped `amdsmi_get_tray_info()` reports compute tray type and accelerator count via `amdsmi_tray_info_t` and `amdsmi_compute_tray_type_t`.
  - Without an active UALoE session these report `AMDSMI_STATUS_NOT_SUPPORTED`, or `UINT32_MAX` for `physical_acc_id`.
  - CLI: `amd-smi static --asic` and `amd-smi list --enumeration` show `PHYSICAL_ACC_ID`; new `amd-smi node --tray`/`-T` prints tray type and accelerator count.

- **`CACHE_ACRONYM` and `TOTAL_CACHE_SIZE` to `amd-smi metric --cache`**.
  - Each `CACHE_<N>` entry now reports a short type label (`L1D`, `L1I`, `L2`, `L3`) and the total size across all instances at that level.

- **`chip_rev_id` and `external_rev_id` to `amdsmi_get_gpu_asic_info()`**.
  - Reports the amdgpu `chip_rev` and `external_rev` values from the `AMDGPU_INFO_DEV_INFO` DRM query, both distinct from `rev_id`, which is the PCI config-space revision. `external_rev_id` is family-scoped, so interpret it alongside `device_id`.
  - Exposed under the same names in the Python `amdsmi_get_gpu_asic_info()` dictionary and in `amd-smi static --asic`. The C fields report `0xFFFFFFFF` when unsupported; Python and the CLI render that as `N/A`.
  - ABI-preserving: the fields take two `uint32_t` slots from `amdsmi_asic_info_t.reserved`, which shrinks from 17 to 15 entries. The structure size and all other field offsets are unchanged.

##### Changed

- **`amdsmi_get_clock_info()` now returns `AMDSMI_STATUS_INPUT_OUT_OF_BOUNDS` for clock values that exceed `INT_MAX`**.
  - Such values were previously narrowed to a negative number and returned as data.

- **Expanded `amdsmi_gpu_block_t` enum with 20 new RAS IP blocks**.
  - Added blocks: from `AMDSMI_GPU_BLOCK_MMSCH` to `AMDSMI_GPU_BLOCK_UCIE_PCS` at bit positions 19-38.
  - Updated `AMDSMI_GPU_BLOCK_LAST` to `AMDSMI_GPU_BLOCK_UCIE_PCS`.

- **A section with no entries now renders as `SECTION: N/A` in human-readable CLI output**.
  - It previously printed a bare `SECTION:` header with nothing beneath it, which read as truncated output. For `SWITCH_ERRORS` this also hid data: a block the driver reported as `N/A` became an empty header.
  - Affects sections that list a variable number of entries: `PORTS` and `RDMA_DEVICES` for an AI-NIC, the per-block counters under `NIC_ERRORS` and `SWITCH_ERRORS`, and `FREQUENCY_LEVELS` in `amd-smi static --clock`. Sections with a fixed set of fields already print `N/A` per field and are unaffected.
  - `--json`, `--csv`, and the table-based subcommands (`monitor`, `partition`, `topology`, `xgmi`, and the default no-argument output) are unchanged.

- **`container_name` in process info now reports the full container ID**.
  - Previously only the first 16 characters were reported. The value is now the complete 64-character ID that `docker inspect`, `docker ps --no-trunc` and Kubernetes tooling use, so process output can be matched against them directly.
  - Nested LXC containers now report the outer container name rather than `<parent>/<child>`.

- **`amd_smi/impl/amd_smi_cper.h` and `example/amd_smi_cper.cc` are no longer installed in the dev package**.
  - Both functions the header declares are C++ symbols, which the version script's `amdsmi_*` export glob does not match, so including the header only ever led to a link error. It joins the `_test` and WSL impl headers that are already build-only.
  - The example is the one shipped file that included that header, so it went with it rather than being left unbuildable against an install tree.

##### Resolved issues

- **Fixed `rsmi_dev_reg_table_get()` failing on register-state images that contain no SMN entries**.
  - The loop-back test ran before the SMN and instance counters reached zero, so an image with no SMN entries re-entered the loop and read past the end of the image; the call then returned an error for a well-formed file.

- **Fixed an out-of-bounds write when a GPU's NUMA or local CPU list names a CPU beyond the bitmask**.
  - `get_bitmask_from_numa_node()` and `get_bitmask_from_local_cpulist()` indexed the bitmask with the parsed CPU number without checking it against the allocated word count. Ranges are now clamped to the bitmask, and negative entries are skipped.

- **Fixed `vram_bit_width` never being reported as `N/A`**.
  - The unavailable-value check compared a `uint32_t` field against `UINT64_MAX`, which can never match, so an unknown bit width was logged as `4294967295`.

- **Fixed an out-of-bounds read in the DRM example's RAS block listing**.
  - `amd_smi_drm_example.cc` iterated every `amdsmi_gpu_block_t` value but indexed a 14-entry name array, so every block from `AMDSMI_GPU_BLOCK_MCA` onward read past the end of the array and printed a garbage label. The list now covers all 39 blocks and the lookup is bounds-checked.

- **Fixed `amd-smi ras --afid --folder --json` emitting nothing when no CPER files are readable**.
  - When every `.cper` file in the folder was skipped (e.g. rejected as a symlink), the JSON path printed empty output, so consumers feeding stdout to `json.loads` failed with `Expecting value: line 1 column 1 (char 0)`. It now emits `[]` for that case, matching the `--cper --json` contract.

- **Fixed an uninitialized processor index that let the CPU and Core APIs read an unrelated CPU socket**.
  - Passing a GPU, NIC, or switch handle to a CPU or Core API could return another socket's telemetry with `AMDSMI_STATUS_SUCCESS`. Such handles are now rejected. Calls that pass a CPU or Core handle are unaffected.

- **Fixed `amd-smi metric` printing nothing for a section requested by name on APUs**.
  - APUs do not expose the discrete-GPU sensors, so those sections are dropped from the default dump. The same suppression applied to an explicitly named section, leaving `amd-smi metric --energy` printing only the GPU header and exiting 0, with no key at all in `--json` and `--csv` output.
  - `--energy`, `--ecc-blocks`, `--overdrive`, `--xgmi-err`, `--pcie`, `--voltage-curve`, `--voltage` and `--fan` now report `N/A` when named explicitly. The default `amd-smi metric` dump is unchanged.

- **Fixed `amdsmi_get_clock_info()` reporting an unavailable clock as `65535` instead of `N/A`**.
  - `amdsmi_clk_info_t.clk` is a `uint32_t`, but the GFX, MEM, SOC, VCLK, and DCLK domains copied the raw `uint16_t` value straight from the GPU metrics table, so the 16-bit unavailable marker surfaced as the literal reading `65535` MHz. The `DF` domain already returned the 32-bit marker.
  - All domains now report an unavailable clock as `UINT32_MAX`, which the Python interface maps to `N/A`. The field is documented accordingly.

- **Fixed `amd-smi metric --usage` reporting APU IPU read/write bandwidth without a unit**.
  - `APU_AVERAGE_IPU_READS` and `APU_AVERAGE_IPU_WRITES` printed bare numbers while the adjacent DRAM counters carried `MB/s`. All four bandwidth counters now report `MB/s`, and the header documents the unit for each.

- **Fixed `amd-smi static --vram` reporting `GDDR7` for LPDDR5 unified memory on APUs (e.g. gfx117x)**.
  - `AMDSMI_VRAM_TYPE__MAX` aliases the highest real memory type (`LPDDR5`), so a genuine LPDDR5 reading was matched by the `__MAX` special case and mislabeled `GDDR7`. It is now correctly reported as `LPDDR5`.

- **Fixed `container_name` in process info being wrong, missing, or crashing `amd-smi`**.
  - `amd-smi process --sort-by-pid` could abort with `terminate called after throwing an instance of 'std::out_of_range'` when any GPU process ran in a cgroup whose path ended in `docker` or `lxc`.
  - Processes in containerd, CRI-O and Podman containers reported no container at all, including Kubernetes pods. They are now identified under either cgroup driver.
  - Processes that were not in a container could be reported as if they were, for example anything running under the Docker daemon's own `docker.service`.
  - Containers managed by LXC 3 or later, or by LXD, under systemd are now identified.
  - A process started from a path of 256 characters or more could report a corrupted process name.

- **Fixed out-of-bounds reads and writes when parsing malformed CPER records**.
  - The ring reader now drops a record whose `record_length` is smaller than a CPER header; such a record was previously copied and then had a product serial number written past the end of the caller's buffer. `amdsmi_get_afids_from_cper` likewise returns `AMDSMI_STATUS_UNEXPECTED_SIZE` for an undersized `buf_size` instead of reading header fields the caller never supplied.
  - A section descriptor's `fru_id` and `fru_text` are now logged only up to their declared width, and a crashdump section's registers are copied into an aligned buffer before decoding. Neither field carries a terminator, and the section can sit at an unaligned record-supplied offset.
  - A non-standard error section whose `reg_arr_size` exceeds the fixed 128-byte register dump, and a record whose `error_severity` overflows the severity mask, are now rejected. Both previously read or matched outside the record; in boot context (`reg_ctx_type` 9) any AFID derived from the over-long register read is gone.

- **Fixed `amd-smi metric --clock` reporting FCLK `MAX_CLK` as 0 MHz**.
  - The FCLK range came from `pp_od_clk_voltage`, which on some GPUs has no FCLK section, leaving the parsed maximum at 0. It now falls back to `pp_dpm_fclk` (and likewise `pp_dpm_sclk`/`pp_dpm_mclk`) when the overdrive file omits a domain.

- **Fixed `amd-smi set -L/--clk-limit <clk> max <value>` not enforcing caps that fall between clock levels**.
  - For `mclk` and `fclk` ONLY, which expose a discrete DPM table, the requested `max` is now rounded down to the nearest selectable clock level, so the enforced limit never exceeds the requested value.
  - `sclk` supports a continuous frequency range, so its requested `max` is honored exactly (e.g. `600` enforces a limit of 600MHz) and is not snapped.

- **Fixed AI-NICs disappearing from `amd-smi` when the RDMA driver is unavailable**.
  - Discovery treated a missing RDMA device list as a fatal error for the whole NIC, so a host with `ionic_rdma` blacklisted (or otherwise not loaded) dropped the NIC entirely and reported no AI-NIC at all.
  - The NIC is now enumerated with an empty `rdma_dev`.
  - `amdsmi_get_nic_rdma_dev_info()` returns `AMDSMI_STATUS_SUCCESS` with `num_rdma_dev` set to 0.
  - `amd-smi static` reports `RDMA_DEVICES: N/A` instead of omitting the device. All other NIC information is reported as usual.

- **Fixed `amdsmi_get_gpu_asic_info()` reporting `rev_id` as a real revision when it is not available**.
  - The WSL backend returned success with a zeroed structure, so `rev_id` read as `0x0`, and where it did report the not-supported value Python rendered it as the raw `0xffffffff`. Python and the CLI now render it as `N/A`.
  - `amdsmi_asic_info_t` is now reset through one shared initializer used by every backend, so a field a backend cannot supply keeps its not-supported value rather than a plausible zero.

##### Upcoming changes

- **UUIDs will be replaced by CUIDs in an upcoming version**.
  - UUIDs will soon be replaced with Component Unified IDs (CUIDs). These CUIDs will be consistent across various AMD tools and products so users will be able to definitively identify their devices regardless of what tool they're using.
  - `amdsmi_get_gpu_device_cuid` has been added as an API for this upcoming change but will remain disabled until full support from the amdgpu driver is available.
  - The CLI `list` output and GPU selection now report the CUID in place of the UUID when a CUID is available, and fall back to the UUID otherwise.

#### **Composable Kernel** (1.3.0)

##### Added

* Support for building Composable Kernel for the target-agnostic SPIR-V target (`amdgcnspirv`), which is compiled to native code at run time.
* Grouped convolution forward, backward data, and backward weight instances for gfx1250.
* A wavelet GEMM pipeline for convolution forward that specializes waves into separate load and math roles to reduce VALU contention.
* A batched contraction kernel with multiple ABD support to CK Tile.
* CK Tile dispatcher support for more GEMM operators, including weight preshuffle GEMM, batched GEMM, batched contraction, grouped A-quantized and AB-quantized GEMM, grouped row-column and tensor quantized GEMM, and block scale quantized GEMM.
* gfx1250 support to the CK Tile dispatcher GEMM operators, covering universal, grouped, multiple D, multiple ABD, batched, batched contraction, and grouped quantized GEMM.
* gfx1250 WMMA instance support to the CK backend for PyTorch Inductor, covering universal GEMM, batched GEMM, and grouped convolution forward.
* The tanh approximation of GELU to the XDL two-stage MoE GEMM epilogue.
* Double-precision buffer atomic add support on gfx1250.

##### Changed

* Disabled the large tensor XDL grouped convolution backward weight instances on gfx1250, where they could produce intermittent memory access faults.
* Changed the ck4inductor instance enumerators to emit a deduplicated, deterministically ordered instance list.

##### Optimized

* Improved FMHA forward performance for head dimension 128 on gfx11 and gfx12 targets by retuning tile selection.
* Improved FMHA forward performance on gfx1250 for bf16 and fp16 head dimension 128 by padding the LDS layout in the qr_tdm pipeline.
* Improved memory coalescing of microscaling (MX) scale loads on gfx1250 by unifying the scale16 layout with scale32.

##### Resolved issues

* Fixed a 32-bit integer overflow in the tensor descriptor element space size that caused undersized workspace allocation and out-of-bounds writes in grouped convolution backward weight for large tensors.
* Fixed grouped convolution forward rejecting valid problem sizes because the implicit GEMM view was subject to the 2 GB logical GEMM size limit.
* Fixed incorrect results in grouped convolution backward data and XDL GEMM kernels caused by an invalid `__restrict__` qualifier on LDS pointers.
* Fixed incorrect accumulation in atomic and split-K kernels on gfx1250 by issuing buffer atomic adds with device scope coherence.
* Fixed memory faults in FMHA batch prefill with a paged KV cache when the page size is a single token.
* Fixed the softmax sink gradient shape in the FMHA backward kernel and corrected sliding window progression when the attention sink is enabled.
* Fixed incorrect results in microscaling (MX) XDL GEMM and B preshuffle kernels when the B operand is more densely packed than A, such as FP8 by FP4.
* Fixed intermittent incorrect results in microscaling (MX) GEMM on gfx1250 caused by missing LDS read ordering in the double-buffered pipeline.
* Fixed incorrect results in FP8 block scale weight preshuffle GEMM and MoE GEMM on gfx1250 caused by an invalid preshuffle layout and a hardcoded wave size.
* Fixed incorrect results in FP4 weight preshuffle quantized GEMM caused by mismatched packed element granularity between the host B preshuffle and the device B tile distribution.
* Fixed incorrect results in the B preshuffle dequantized GEMM pipeline caused by the C transpose setting being ignored.
* Fixed incorrect results from the CK Tile compute v3 intrawave GEMM pipeline with eight-warp block arrangements on gfx1250.
* Fixed out-of-bounds asynchronous loads in the CK Tile compute async WMMA GEMM pipeline caused by a missing element validity check.
* Fixed accuracy loss in the bf16 fast GELU element-wise operation on gfx1250 by using the non-native implementation.
* Fixed a build failure caused by the compiler change from vector to ext_vector types.
* Fixed a CMake failure when building instance libraries for multiple offload targets.
* Fixed an unspecified minimum blocks per compute unit value being passed to the compiler.
* Fixed the CK Tile dispatcher code generator and its ctypes bindings failing to build.

#### **HIP** (7.16.0)

##### Added
* New HIP APIs:
    - Device Management: Support for the following APIs for parity with corresponding CUDA APIs.
      * `hipDeviceGetLuid` returns the locally unique identifier (LUID) and device node mask for the specified device.
      * `hipInitDevice` initializes the runtime state for the specified device without making it the current device for the calling thread. It also applies the requested flags and ensures the device's default stream is created.
* New HIP device attribute:
    - `hipDeviceAttributeHostAllocDmaBufSupported` is now supported, enabling host-allocated buffer sharing.
* Support for host-NUMA virtual memory management (VMM) in `hipMemCreate()` and related VMM APIs. These APIs now support `hipMemLocationTypeHostNuma` and `hipMemLocationTypeHostNumaCurrent`, enabling allocations backed by physical host memory on the selected NUMA node. Previously, support was limited to GPU VMM pools with deferred host access. This enhancement aligns HIP behavior with the corresponding CUDA APIs and expands support for NUMA-aware memory allocation.
* Support for coarse-grained memory coherency on Windows. In supported Windows configurations, applications can now use unified memory to reduce memory footprint by eliminating unnecessary host-device data copies. Components interacting with the device can directly access host memory pointers and enable coarse-grained memory coherency by registering and pinning the associated host allocations using `hipHostRegister()` with the `hipExtHostRegisterCoarseGrained` flag. This provides behavior on Windows that is consistent with the existing Linux implementation while improving memory efficiency.

##### Resolved issues
* On Windows, HIP runtime now correctly handles non-P2P data transfers between GPUs and coordinates multi-GPU kernel execution. It eliminates deadlocks and invalid values in multi-process workloads and resolves issues observed when running LLMs on multi-GPU Windows configurations.
* Resolved an out-of-memory issue affecting certain AMD APUs, such as Strix Halo, on Windows when loading LLMs that could exceed dedicated graphics memory and spill into shared memory. The HIP runtime now correctly uses the full unified memory pool available on high-memory APUs, enabling system RAM to be dynamically allocated as graphics memory. This enhancement improves memory utilization and supports the execution of larger AI models on affected APU platforms.
* Fixed a memory leak in the HIP/HSA runtime that could occur during stream and signal creation on certain GPUs. The issue was triggered by `hipStreamCreate()`, resulting in allocated signal objects not being properly released. The HIP/HSA runtime now correctly releases allocated signal objects during stream destruction and runtime cleanup, eliminating the memory leak and improving resource management.

##### Known issues

* Under WSL2 (Windows Subsystem for Linux 2), GPU device-side memory faults might not be reported correctly and can result in the process hanging.

#### **hipBLAS** (3.7.0)

##### Added

* Level 3 grouped batched GEMM functions `hipblasSgemmGroupedBatched`, `hipblasDgemmGroupedBatched`, `hipblasGemmGroupedBatchedEx`, and `hipblasGemmGroupedBatchedExWithFlags` for both C and FORTRAN, including ILP64 API (`_64` name suffix).

#### **hipCUB** (4.7.0)

##### Changed

* Benchmarking now uses primbench for its benchmarks instead of Google Benchmark.
  * See [primbench/README.md](https://github.com/ROCm/rocm-libraries/blob/develop/shared/primbench/README.md) for its documentation.

#### **hipFFT** (1.0.26)

##### Added

* Support for the `amdgcnspirv` architecture in client programs, so that they are functional even on gfx architectures for which they have not been explicitly compiled.
* `hipfftXtSetWorkArea` API for setting work areas on single-process, multi-device plans.

* Implemented `hipfftXtSetJITCallback` API, to allow for user-defined device functions to be called when loading
  input or storing output of a transform. These callback functions are specified after a plan is allocated with
  `hipfftCreate` but before the plan is initialized with one of the `MakePlan` functions. The backend FFT library
  will Just-In-Time (JIT) compile the code into its own kernels.

  On AMD platforms, the device function is provided as SPIR-V. On CUDA platforms, the device function is provided as
  LTO-IR fatbin.

  These APIs are not currently compatible with multi-GPU transforms. This
  support will be added in a future release of hipFFT.

##### Changed

* Modified the rocFFT backend's implementation details of hipFFT so that cuFFT backend's
  behavior is matched for single-process, multi-device plans configured via
  `hipfftMakePlan{2,3}d` and `hipfftMakePlanMany`, with respect to data distribution
  within descriptors. Behaviors are now aligned for unbatched multi-dimensional transforms
  (in-place only) and all batched transforms (in-place and out-of-place).
  Multi-device, unbatched one-dimensional transforms remain unimplemented pending
  further analyses of the exact behavior(s) to be matched.

##### Resolved issues

* Fixed a hang when creating a plan with a zero length or zero batch. These now return `HIPFFT_INVALID_SIZE`.
* `hipfftGetSize*` work area estimates now account for the plan's pre-initialization
  state (e.g. multi-device configuration) instead of ignoring it.

##### Upcoming changes

* The `hipfftXtSetCallback` and `hipfftXtClearCallback` APIs are now deprecated and will be removed in a future
  release. They allow for specifying callbacks as device function pointers at plan execution time, but rocFFT cannot
  optimize the combined code. Instead, users should specify JIT callbacks with `hipfftXtSetJITCallback` before
  initializing the plan.

#### **hipFile** (0.5.0)

##### Added

* A Stats API for querying hipFile I/O statistics. `hipFileGetStatsL1()`, `hipFileGetStatsL2()`, and `hipFileGetStatsL3()` return progressively more detailed counters: basic I/O and operation counts (Level 1), I/O size histograms (Level 2), and per-GPU statistics (Level 3).
* `ais-check` now detects SR-IOV virtual function (VF) GPUs via `amd-smi` and warns when one is present. hipFile's fastpath is only supported on GPU physical functions (PFs); on a VF, I/O falls back to the compatibility path. The check is skipped if `amd-smi` is unavailable.
* `hipFileReadAsync()` and `hipFileWriteAsync()` now support the AIS fastpath backend, enabling asynchronous GPU-direct I/O enqueued on a HIP stream. Transparent async backend failover to the slowpath is not currently supported for async fastpath operations.
* Batch operations now execute on an internal thread pool, enabling batch API support on the AMD backend. Together with async fastpath support, this resolves the 0.3.0 limitation where batch and async API calls were unsupported on the AMD backend.
* The `HIPFILE_ASYNC_BUFFER_SIZE` environment variable to control the size of the host bounce buffer used for asynchronous fallback I/O. The default size is 16 MiB; setting it to `0` uses the default.

##### Changed

* The synchronous fallback I/O path now sets the active HIP device to the buffer's GPU before `hipMemcpy` and restores the caller's device afterward, fixing copies that could run against the wrong device context.
* Asynchronous fallback I/O now reuses a single per-stream bounce buffer, splitting large transfers into chunks that fit the buffer, to reduce the memory footprint of asynchronous workloads.

##### Resolved issues

* Corrected CMake ROCm path detection so out-of-tree builds locate the correct ROCm installation.

##### Known issues

* Asynchronous operations that use the fastpath backend will not retry on the fallback backend. If asynchronous operations have proper alignment, they now run on the fastpath backend. If the fastpath device lookup fails or the P2P DMA transfer is not supported between the devices, the asynchronous operation will now fail.

#### **hipSOLVER** (3.7.0)

##### Added

* Compatibility-only functions for larft:
    * `hipsolverDnXlarft_bufferSize`
    * `hipsolverDnXlarft`

##### Resolved issues

* Fixed `hipsolverDnXpotrs` calling 32-bit potrs instead of 64-bit potrs.

#### **hipSPARSE** (4.8.0)

##### Added
* The generic API routines `hipsparseSpGEAM_createDescr`, `hipsparseSpGEAM_destroyDescr`, `hipsparseSpGEAM_bufferSize`, `hipsparseSpGEAM_nnz`, and `hipsparseSpGEAM` for sparse matrix-matrix addition (`C = alpha * op(A) + beta * op(B)`), along with the `hipsparseSpGEAMDescr_t` type and the `hipsparseSpGEAMAlg_t` algorithm enum, to match the cuSPARSE 13.3 generic `SpGEAM` API.
* Batched support to `hipsparseSDDMM` for CSR format.

#### **hipThreads** (1.0.0)

##### Added

* `std::thread`-style concurrency primitives that run inside GPU kernels. `hip::wthread`, `hip::mutex`, `hip::lock_guard`, and `hip::condition_variable`, along with cooperative `pseudo_*` variants, mirror the C++ standard-library concurrency API.
* A persistent scheduler kernel that accepts work from both the host and the device, with multi-fiber (SIMD width) execution so a single unit of work can run across multiple GPU lanes.
* Runtime-tunable scheduling. Scheduler concurrency can be configured at runtime through the `HIPTHREADS_VCORES_PER_WGP` environment variable to match your GPU and workload.
* Cross-platform build and tooling. A CMake build with native HIP language support, a lit-based test suite, and example projects. All are supported on Linux and Windows.

#### **libhipcxx** (3.0.2)

##### Added

* libhipcxx is included in the ROCm Core SDK and is built with TheRock.
* `<cuda/std/span>`, backported to C++14.
* `<cuda/std/mdspan>`, backported to C++14.
* `<cuda/std/concepts>`, backported to C++14. C++20 concepts are usable in C++14 and C++17 through existing SFINAE techniques.
* Structured bindings for `cuda::std::tuple`, `cuda::std::pair`, and `cuda::std::array`.
* Experimental `{async_}resource_ref`.
* Support for Clang 15.
* Support for using `lerp` in device code.

##### Changed

* Advanced the version to 2.1 with no breaking changes, aligning with Thrust and CUB as the three libraries move toward a unified CUDA C++ Core Libraries (CCCL) repository.
* Updated the atomics backend to be compatible with `atomic_ref`.
* Modularized `<type_traits>`, `<iterator>`, `<utility>`, and `<functional>`.

##### Resolved issues

* Fixed errors in `atomic` with small aggregates and enum classes.
* Corrected CMake install rules.
* Fixed several issues discovered through tests added for host-only translation units.

#### **MIOpen** (3.6.1)

##### Added

* Support for building kernels with debug symbols. Set `MIOPEN_DEBUG_SYMBOLS_KERNEL=1` to add debug symbols.
* [BatchNorm] Implements Welford's algorithm for calculating variance in FwdTrainSpatial variant 1
* [RNN] Added a fused single-layer LSTM forward-inference kernel, gated behind `MIOPEN_DEBUG_RNN_FUSED_INFERENCE` (disabled by default).
* [Conv] Added a naive reference kernel enabling 3D INT8 forward convolution.
* [Conv] Added the `ConvHipConv` solver backed by vendored hipconv kernels.
* [Conv] Added float8 vector type support and deduplicated the shared math kernels.
* [Conv] Added validated gfx1151 and gfx1200 (Navi) SystemDBs so immediate-mode lookups use tuned entries instead of generic heuristics.
* [Conv] Added `MIOPEN_DEBUG_DISABLE_SYSTEM_DB` and `MIOPEN_DEBUG_DISABLE_USER_DB` runtime environment variables to disable the system and user databases.
* [Conv] Enabled Composable Kernel (CK) depthwise convolution on RDNA wave32 GPUs.

##### Changed
* [Conv] Pruned ASM-GTC NHWC entries from the gfx908, gfx90a, gfx942, and gfx950 system find-databases for shapes the new large-tensor guard rejects, so immediate-mode lookups no longer resolve to a solver that is gated off at runtime. This removes 12,907 entries across the six databases, 4,668 of them rank-1, and empties 81 keys.
* [Conv] Enabled the Winograd Rage RxS f2x3 solver on gfx950 and updated the gfx942 kernels to v4_6_1/v4_9_1.
* [Conv] Updated the 3D AI heuristics (solver selection) for gfx942 and gfx950.

##### Removed
* Removed disabled convolution solver `ConvCkIgemmFwdV6r1DlopsNchw`, `ConvHipImplicitGemmBwdDataV1R1Xdlops`, and `ConvHipImplicitGemmBwdDataV4R1`.

##### Optimized
* [Conv] Use a single GEMM for point (1x1) output convolutions in all three directions (forward, backward-data, and backward-weights) for both 2D and 3D.
* [Conv] Optimized the naive backward-data convolution kernel by widening its workgroup to 1024 threads.

##### Resolved issues
* [Conv] Fixed silently incorrect results from the grouped backward-weights CK xdlops solver when a tensor's element extent exceeds INT_MAX but its individual lengths and strides still fit int32; such problems now use a large-tensor (int64) CK instance instead of overflowing int32 indexing.
* [Conv] Fixed a HIPRTC compilation failure in the ConvDepthwiseFwd3D (gfx942/gfx950) FP16/BFP16 solver.
* [BatchNorm] Fixed MIOpen#3900 by implementing Welford's algorithm in FwdTrainSpatial variant 1
* [Conv] Fixed a batch-split miscalculation that produced incorrect results for large backward-weights (WrW) tensors.
* [Conv] Fixed a convolution failure on XNACK-unsupported gfx9 APUs (gfx902, gfx909, gfx90c).
* [BatchNorm] Fixed a device heap-buffer-overflow in the forward spatial `FinalMeanVariance` kernel.
* [Conv] Fixed incorrect `SetNextValue` polarity in deterministic mode for the grouped backward-data and backward-weights CK solvers.
* [LRN][LayerNorm] Fixed grid alignment and out-of-bounds thread access on gfx1250.
* [Conv] Removed stale gfx950 grouped backward-weights SystemDB entries that caused suboptimal solver selection.
* [Conv][Windows] Fixed system-database path resolution to use the MIOpen module location.
* [Conv] Fixed system-database fallback so MI308X (gfx942) resolves to a usable tuned database.
* Fixed JSON performance logs (`MIOPEN_PERFORMANCE_LOGS`) dropping the last solver evaluated during Find, so naive convolution solvers are no longer omitted.
* Fixed a build failure when compiling against GCC 15's libstdc++ under C++20.

#### **RCCL** (2.30.7)

##### Added

* Communicator suspend and resume (`ncclCommSuspend`, `ncclCommResume`, `ncclCommMemStats`), which releases the dynamic GPU memory of an idle communicator and reacquires it later without destroying the communicator.
* accl-profiler profiler plugin for per-collective timing decomposition (`ACCL_PROFILER_OUTPUT_DIR`, `ACCL_PROFILER_MIN_SIZE_BYTES`).
* DDA `AllReduce` and `AllGather` on the gfx1250 fabric path in both LL and LL128 protocols. `AllReduce` adds one-shot and two-shot tiers per protocol, AllGather one tier each. Gated by `RCCL_DDA_ENABLE` and `RCCL_DDA_LL`, both default on.
* An experimental gfx1250 Tensor Data Mover path for copy-shaped SIMPLE-protocol transfers. All collectives can reach it, but reduction collectives only qualify on slices that carry no reduction operation. Excluded from the default build: it requires `--enable-tdm-simple` at build time and `RCCL_TDM_SIMPLE_ENABLE=1` at runtime. Reduction into LDS staging buffers and double buffering are not yet implemented.
* `install.sh --all_unrolls` (`-DBUILD_ALL_UNROLLS=ON`) to generate every unroll factor (1, 2, 4, 8, 16, 32) for the targeted GPU architecture(s), for measuring unroll factors that the default per-arch matrix does not build. The flag also drops the per-architecture pin, so such a build accepts every `RCCL_UNROLL_FACTOR` value on any targeted architecture.
* The strict `--enable-full-coverage` install flag for unified host + device LLVM source-based code coverage. It requires `--debug`, the device linker, and ROCm 7.15 or newer. The test runner uses the CMake-level `ENABLE_FULL_COVERAGE=AUTO` mode, which falls back to host-only coverage when device instrumentation is unavailable.
* Enabled the Copy-Engine profiler path (`ncclProfiler_v6`): Copy-Engine events are emitted for CE collectives, and the example profiler plugin reports them.

##### Changed

* Raised the default channel count on single-node gfx1250 to 256 for both collectives and P2P. The count is still clamped by the GPU CU count and by `NCCL_MAX_NCHANNELS` / `NCCL_MAX_CTAS` / `NCCL_MAX_P2P_NCHANNELS`. Multi-node gfx1250 keeps the 64-channel cap on the NET path. `RCCL_SATURATE_P2P_NCHANNELS` now defaults to on for gfx1250 so the per-peer channel count tiles the larger pool; set it to `0` to restore the previous behavior.
* Narrowed unroll-factor kernel generation: gfx1250 local builds now generate only unroll 32, its runtime default, instead of 8/16/32, and a multi-arch build generates 1/2/4/32 instead of all six factors. This cuts the multi-arch kernel count by roughly a third; use `--all_unrolls` to build 8 and 16.
* Capped DDA fabric SIMPLE kernels by each GPU's CU count and use the clique-wide minimum so every rank launches matching barrier blocks. `RCCL_DDA_FABRIC_MAXBLOCKS` can lower this cap but cannot raise it; invalid, nonpositive, and excessive values now produce diagnostics.
* Raised `kDdaMaxNranks` from 72 to 144, extending DDA fabric path support to larger DPX cliques.

##### Resolved issues

* `NCCL_MAX_P2P_NCHANNELS` opt-in is now detected from the environment rather than from the parameter value. The value defaults to `MAXCHANNELS`, so every unset run was treated as an opt-in past the historical `4*CHANNEL_LIMIT` (64) bound. As a result, P2P channels on non-gfx1250 architectures were limited only by the collective channel count, and the gfx950 (MI350) multi-node P2P caps never applied. Set `NCCL_MAX_P2P_NCHANNELS` explicitly to restore a higher bound.
* Fixed `ncclConfig_t::maxP2pPeers` and `NCCL_P2P_MAX_PEERS` having no effect. The field was plumbed through configuration parsing, validation, the version gate, the communicator-size cap and the rank exchange, but the resolved `comm->p2pMaxPeers` was never read: the NCCL consumer sits inside `ncclTopoComputeP2pChannels`, which RCCL rewrites, so the NCCL 2.30.4 sync landed the producer without the consumer. Both P2P per-peer channel heuristics, the `RCCL_SATURATE_P2P_NCHANNELS` tiling and the multi-node per-peer reduction, now divide by `maxP2pPeers` instead of the full rank count, so limiting the number of concurrent P2P peers raises the number of channels each of those peers receives. Library behavior is unchanged when the knob is left unset, where `maxP2pPeers` resolves to the communicator size. Note that rccl-tests `sendrecv_perf` sets `maxP2pPeers = 2` through the communicator config, so its per-peer channel count, and therefore its reported bandwidth, does change. Two `NCCL_ENV` log lines also named the variable `NCCL_MAX_P2P_PEERS`; the variable is, and always was, `NCCL_P2P_MAX_PEERS`.
* Fixed `NCCL_CHECK_MODE` having no effect. `commAlloc` reset `comm->checkMode` from the deprecated `NCCL_CHECK_POINTERS` after `NCCL_CHECK_MODE` had already been parsed, so `DEBUG_LOCAL` was only reachable through the deprecated variable and `DEBUG_GLOBAL`, which validates symmetric buffer registration across ranks, was unreachable entirely.
* `RCCL_UNROLL_FACTOR` is now rejected when the requested unroll factor's device functions were not compiled for the running architecture, instead of being accepted and then dispatching into an empty device function table. Unroll factor 32 is compiled for gfx1250 only, but a multi-arch build reported every unroll factor as available, so requesting it on another GPU crashed on the device. That value now fails communicator initialization with a warning naming the architecture, and the default selection falls back to the highest unroll factor actually compiled for the GPU rather than trusting the heuristic's choice. The WarpSpeed auto-tuner's preference for unroll factor 2 is subject to the same check, so it no longer replaces a validated unroll factor with one this build cannot dispatch. gfx1250 is unaffected and still defaults to 32.
* Fixed the per-unroll device function tables being misaligned in multi-arch builds. The LL128 `SendRecv` kernel was skipped for unrolls 8/16/32, so every function after `SendRecv` in those tables sat one index below the id the host had computed from the unroll-1 ordering, and the last entry of the host lookup table was dropped. `AlltoAllPivot`, `AlltoAllGda`, `AlltoAllvGda` and `AllGatherV` were affected on gfx1250.
* Fixed the AMD SMI fabric ABI guard rejecting the layout amd_smi 27.x introduced, which broke the RCCL build outright. The fabric payload union gained a second member, enlarging `amdsmi_fabric_info_t` without moving the v1 fields RCCL reads. RCCL now recognizes that layout, identifies the loaded runtime by how much of the probe buffer it writes, and falls back to the sysfs fabric backend on a 27.x runtime rather than reading a payload it does not model. Fabric topology on such a runtime therefore comes from sysfs, and RCCL warns once per process when it makes that switch.
* Fixed gfx1250 LL and LL128 comm-FIFO hangs on sibling DPX partitions. The FIFO store now uses system-scope b128 (`RCCL_LL_FIFO_SYS_SCOPE`) alongside the existing system-scope load, preventing hangs at slot reuse starting from the 9th collective operation.
* Fixed DDA fabric AllToAll validation race by staging send data into scratch with a host-launched `cudaMemcpyAsync` before the peer exchange kernel.
* Fixed DDA fabric barrier publication race causing sporadic validation errors by using release-acquire semantics on the prologue barrier.

##### Known issues
* On gfx90a (MI210/MI250/MI250X) with ROCm 7.14 or later, per-launch scratch-memory reclaim in the runtime degrades RCCL performance. Set `HSA_NO_SCRATCH_RECLAIM=1` to restore performance.
* Collectives that select the RCCL DDA path are not traced by the profiler plugins. `RCCL_DDA_ENABLE=0` can be used to route the collectives through the instrumented path while profiling.

#### **rocBLAS** (5.7.0)

##### Added

* Level 3 grouped batched GEMM functions `rocblas_sgemm_grouped_batched`, `rocblas_dgemm_grouped_batched`, and `rocblas_gemm_grouped_batched_ex` for both C and FORTRAN, including ILP64 API (`_64` name suffix).

##### Changed

* On gfx950, Level 3 `gemm`, `gemm_ex`, and the functions that internally use GEMM, for single- and double-precision now default to the hipBLASLt backend instead of Tensile. Complex types on gfx950 still default to Tensile. `ROCBLAS_USE_HIPBLASLT` continues to force or disable the hipBLASLt backend.

##### Optimized

* Improved the performance of Level 3 `gemm` for the problem sizes where `m == 1` or `n == 1` and `batch_count == 1` by using `gemv` kernels, previously applied only in `gemm_ex`. On gfx11 the per-precision heuristics guarding this path are also bypassed, except for the `1x1` case.
* Improved the performance of Level 2 `gemv` non-transposed (`TransA == N`) for the problem sizes where `m` is small and `n` is large by splitting the reduction across the grid, as the transposed case already does.

##### Resolved issues

* Fix incorrect per-batch `alpha`/`beta` values on the ILP64 (`_64`) path for batched and strided-batched `scal` (including `_ex`), `ger`, `geru`, `gerc`, `syr`, `symv`, `hemv`, `sbmv`, and `spmv` when `rocblas_set_batch_alpha_stride` or `rocblas_set_batch_beta_stride` is set in device pointer mode. The 64-bit launchers passed an unoffset scalar pointer into each batch chunk, and `scal` also advanced `alpha` by the vector-length chunk instead of the batch index, so later batches used the wrong scalars.
* Fix out-of-bounds workspace access in Level 3 batched and strided-batched `syrk` and `herk` on gfx90a and gfx942 with `batch_count` greater than 65536, `k` of at least 500, and `n` below an internal per-architecture threshold, where the GEMM-only path advanced its workspace pointer cumulatively on each pass of the batch sweep and so wrote past the end of the workspace. This could corrupt memory past a workspace supplied through `rocblas_set_workspace` or, when the device memory pool was sized to the requirement reported by a size query, fault, or return incorrect results. The ILP64 (`_64`) forms were unaffected.
* Fix incorrect results from Level 1 `dot` and `dotc` batched and strided-batched forms, including their `_ex` forms, when `batch_count` is greater than 65535. Every batch item at index 65535 and beyond reduced an empty range and returned zero. The ILP64 (`_64`) forms were unaffected, as they chunk the batch dimension below that limit.
* Fix incorrect results from Level 3 batched and strided-batched `trsm` on the small left-side device path when `batch_count` is greater than 65535 and each batch item has a distinct `A` pointer. Threads that finished the first grid pass left the kernel, so later passes could not reload `A`. Also fix Level 3 `trmm` out-of-place when `batch_count` is greater than 65535 and a per-batch `alpha` of zero caused the kernel to skip remaining batches. The ILP64 (`_64`) forms were unaffected, as they chunk the batch dimension below that limit.
* Fix the hipBLASLt backend returning `rocblas_status_internal_error` when an explicit GEMM solution index is unsupported. The call now returns `rocblas_status_invalid_value`, matching Tensile.
* Fix `ROCBLAS_TENSILE_GEMM_OVERRIDE_PATH` ignoring the solution indices reported by `rocblas_gemm_ex_get_solutions` and `rocblas-gemm-tune`. Tensile solutions are reported as negative indices and were previously discarded when loading an override file, leaving the default kernel selection in place. Raw positive Tensile indices in existing override files are still honored after this fix. An entry which names no Tensile solution is skipped with a warning instead of failing the other overrides in the file.
* Fix `rocblas-gemm-tune` skipped best solution reporting when it came from the Tensile backend. Problems whose fastest kernel belongs to neither backend, such as the internal gemv kernel, remain unreported as no index can name them.
* Fix a process hang on Windows exit when profile logging is enabled (`ROCBLAS_LAYER` bit 2, for example `ROCBLAS_LAYER=4`). The profile dump waited on a worker thread that the loader had already terminated during `DLL_PROCESS_DETACH`.

#### **rocDecode** (1.10.0)

##### Added

* Support for explicitly loading `librocm_sysdeps_va` via `dlopen`, ensuring complete isolation from the system `libva` library.

##### Resolved issues

* Resolved vendored `libva` link issue in samples without extra env vars.

#### **rocFFT** (1.0.40)

##### Added

* Support for the `amdgcnspirv` architecture in client programs, so that they are functional even on gfx architectures for which they have not been explicitly compiled.

* Implemented `rocfft_plan_description_set_load_callback` and `rocfft_plan_description_set_store_callback` APIs, to
  allow for user-defined device functions to be called when loading input or storing output of a transform. These
  callback functions are specified during plan creation and allow rocFFT to Just-In-Time (JIT) compile the code into
  rocFFT's own kernels.

  Also implemented `rocfft_execution_info_set_load_callback_data` and
  `rocfft_execution_info_set_store_callback_data` APIs to allow
  specifying the data pointers for these callbacks, which may differ
  on each execution.

  These APIs are not currently compatible with transforms that have
  fields or bricks also specified on the same plan description. This
  support will be added in a future release of rocFFT.

* Support for very large FFTs on gfx1250.

##### Resolved issues

* Addressed a cache-reuse issue with RCCL communicators by giving each communicator its own set of streams.
* Fixed multi-dimensional real transforms returning wrong results when a complex
  transform that shares the same internal 2D kernel had been planned earlier in the
  process and its plan was still alive. A real transform's kernel needs extra
  twiddle values for its fused real pre/post-processing step, but the 2D twiddle
  table cache key did not distinguish the two, so the real transform could be handed
  the complex transform's shorter table.
* Fixed `rocfft_plan_create` hanging when given a zero FFT length, zero batch, or zero dimensions; these
  now return `rocfft_status_invalid_dimensions` or `rocfft_status_invalid_arg_value`.
* Fixed `rocfft_execution_info_set_stream` to derive the device from the stream itself instead of assuming the current device.

* Fixed a potential issue where rocFFT could terminate the calling process if a HIP module failed to load during plan
  creation.

##### Known issues

* Function pointer callbacks specified via `rocfft_execution_info_set_load_callback` or
  `rocfft_execution_info_set_store_callback` are not functional on gfx1250 and `rocfft_execute` will fail in this case.

##### Upcoming changes

* The `rocfft_execution_info_set_load_callback` and `rocfft_execution_info_set_store_callback` APIs are now
  deprecated and will be removed in a future release. They allow for specifying callbacks as device function
  pointers at plan execution time, but rocFFT cannot optimize the combined code. Instead, users should specify JIT callbacks on plan descriptions.

#### **rocJPEG** (1.10.0)

##### Added

* `rocJpegDecodeBatchedAsync` and `rocJpegDecodeBatchedSync` APIs to support asynchronous batched JPEG decoding.
* Support for explicitly loading librocm_sysdeps_va via dlopen, ensuring complete isolation from the system `libva` library.

#### **ROCm Compute Profiler** (3.9.0)

##### Added

* GPU benchmarking and roofline profiling and analysis support for gfx1153 hardware.

* Per-kernel PC sampling analysis.
  * `rocprof-compute analyze --output-format csv` writes each kernel's disassembly under `per_kernel_pc_sampling/`, with the sample and stall counts on every instruction and the source it was compiled from.
  * The analysis database records the same per-instruction data, so `--output-format db` can be queried for it.

* Multi-process PC sampling across profile and analyze modes.
  * Profile mode writes one PID-prefixed `<pid>_ps_file_results.json` per process.
  * Analyze mode reports every process in a single run, with a `pid` column
    identifying each one.

* Redesigned the standalone roofline HTML to improve user experience and interactivity.

* A profile-mode warning reporting the active compute and memory partition
  modes on partition-capable accelerators, noting that analysis derives logical
  XCD, L2 channel, and HBM channel counts from them.

* [Profile vLLM workloads](https://rocm.docs.amd.com/projects/rocprofiler-compute/en/docs-10.1.0/how-to/profile/mode.html#profile-vllm-workloads) guide for profiling vLLM workloads and its caveats.

##### Changed

* Renamed the PC sampling analysis output: `pc_sampling.csv` is now `pc_sampling_summary.csv`, and the `compute_pc_sampling_view` view is now `compute_pc_sampling_summary_view`.

* ML API tracing options (`--torch-trace`/`--triton-trace`/`--ml-api-trace`) are no longer allowed with PC-sampling-only profiling; the run now fails with an error telling the user to drop the ML API tracing flag or add a counter block, since without counters there is nothing to correlate the markers against.

##### Removed

* Removed the CSV profile output backend and the `--format-rocprof-output` profile mode option. Profiling now always uses the `rocpd` output format, which was already the default.
  * Removed the `--join-type` profile mode option, which only affected the CSV output format.

* Removed analyze support for workloads produced by the CSV profile backend. Such workloads are now rejected with an error telling you to re-profile with a current release.

##### Optimized

* Reduced profile-mode peak memory when writing counter data on large workloads.

* Profile mode now gzip-compresses large counter CSV artifacts to reduce workload directory size.

##### Resolved issues

* Corrected the VGPR allocation label from `RVGPRseq` to `VGPRs` in gfx9 memory charts

#### **ROCm Data Center Tool** (1.3.1)

##### Removed

- Removed the unused `GetBlockNameStr()` test helper and its GPU block name map. Nothing called it, and its `static_assert` on `AMDSMI_GPU_BLOCK_LAST` broke the RDC build whenever AMD SMI added an IP block.

##### Resolved issues

- `RDC_FI_GPU_MEMORY_CUR_BANDWIDTH` no longer reports zero under DMA or copy-only memory traffic on GPUs whose instantaneous UMC activity does not reflect DMA transfers. It now derives memory activity from the `mem_activity_acc` accumulator over firmware time, and falls back to the instantaneous `umc_activity` reading when the accumulator or firmware timestamp is unavailable.

- `RDC_FI_ECC_CORRECT_TOTAL`, `RDC_FI_ECC_UNCORRECT_TOTAL`, and `RDC_FI_ECC_DEFERRED_TOTAL` no longer hang when AMD SMI defines GPU blocks above bit 31. The block iteration used a 32-bit counter that wrapped to zero instead of terminating.

<a id="rocgdb-16-3"></a>

#### **ROCm Debugger (ROCgdb)** (16.3)

##### Added

* For AMD64 and AMDGPU targets, ROCgdb now displays backtrace information for Clang's `__builtin_verbose_trap`. The debug information emitted by the compiler describes the category and message arguments passed to the built-in as a call to an inlined function, and ROCgdb presents it as the context of frame `#0: #0 ... __clang_trap_msg$<category>$<message> () at ...`

* Support for the gfx1103 GPU architecture.

##### Changed

* Improved `maint print address-spaces` command to display properties of address spaces supported by an architecture. For each address space, print its name, DWARF id, address size, null address, and access class.

* The "catch hiperr" feature is now exposed to MI too, with a new `-catch-hiperr` command and related fields in `*stopped` records.
  See the "HIP Runtime Error" subsection of the "GDB/MI Catchpoint Commands" section in the [ROCgdb manual](https://rocm.docs.amd.com/projects/ROCgdb/en/latest/ROCgdb/gdb/doc/gdb/GDB_002fMI-Catchpoint-Commands.html).

#### **ROCm Systems Profiler** (1.9.0)

##### Changed

- **rocpd is now the default output format.** When no output format is specified,
profiling data is emitted as a rocpd SQLite database (`rocpd.db`). Perfetto (`.proto`)
output must now be explicitly enabled via `--output-format proto`. Requires
ROCProfiler-SDK 1.0.0 or later (ROCm 7.0.0+).
- `ROCPROFSYS_PROFILE` (timemory backend) now defaults to `false`, since rocpd
replaces Perfetto as the primary trace output.
- All built-in presets that perform tracing (`--balanced`, `--detailed`, `--sys-trace`,
  `--runtime-trace`, `--trace-gpu`, `--trace-hpc`, `--trace-hw-counters`, `--trace-openmp`,
  `--workload-trace`) now produce a rocpd database by default, because rocpd is the new
  library default. The `--profile-only` and `--profile-mpi` presets explicitly disable
  rocpd output to preserve their lightweight, flat-profile-only character.
- `ROCPROFSYS_SAMPLING_GPUS` is now restricted by the GPUs the ROCm runtime exposes
  via `ROCR_VISIBLE_DEVICES` / `HIP_VISIBLE_DEVICES`.
- The `trace-hpc` preset now enables flat profiling (`ROCPROFSYS_FLAT_PROFILE`) by
  default. Pass `--profile` to get a call-stack-based profile instead.
- `rocprof-sys-python` no longer accepts abbreviated long options (for example,
  `--conf` for `--config`). Spell out the full option name.

##### Removed

- Removed the `ROCPROFSYS_BUILD_SQLITE3` CMake option and the in-tree SQLite3/rocpd
  storage backend. This is now handled by profiler-hub.

- Removed the deprecated `rocprof-sys-user` library and its C API
  (`rocprofsys_user_*`, `<rocprofiler-systems/user.h>`), including the `user`
  find_package component, the `examples/user-api` example, the Python
  `rocprofsys.user` submodule, and the associated pytest coverage. Use
  ROCTx (`rocprofiler-sdk-roctx`) for general-purpose manual instrumentation
  (starting/stopping tracing, named regions) instead; see `examples/roctx`
  for usage.

  - Causal profiling's `ROCPROFSYS_CAUSAL_PROGRESS`/`ROCPROFSYS_CAUSAL_BEGIN`/
    `ROCPROFSYS_CAUSAL_END` macros are unaffected at the source level: they now
    run on a new, minimal `rocprof-sys-causal-api` library instead of the removed
    general-purpose user API. Existing causal profiling code does not need to be
    edited, but it must be rebuilt against the new headers and library, and
    projects that requested the old component via
    `find_package(rocprofiler-systems COMPONENTS user)` must change that to
    `COMPONENTS causal-api`. The `user` component no longer exists, so requesting
    it now fails at configure time.

##### Resolved issues

- Fixed `rocprof-sys-python` ignoring the `-c`/`--config` flag. The configuration
  file is now applied to `ROCPROFSYS_CONFIG_FILE` before the profiler bindings are
  loaded, so its settings take effect. A configuration file already named by
  `ROCPROFSYS_CONFIG_FILE` is preserved, and the one given on the command line is
  appended to it.
- Pausing sampling now stops the underlying per-thread timers instead of only discarding the samples they produce. Previously a paused sampler kept delivering timer signals, so the profiled application's sleeps were still interrupted throughout a window in which no data was being collected.
- `ROCPROFSYS_TRACE_DELAY`/`ROCPROFSYS_TRACE_DURATION` now actually gate GPU context startup, producing a real gap in cached GPU/rocpd data. Previously they only suppressed downstream category emission. For GPU-only tracing with no marker domain or trace region configured, the configured delay had no effect at all on when GPU data collection actually began.

#### **ROCprofiler-SDK** (1.4.1)

##### Added

**API:**

  - Advanced Thread Trace (ATT) support in the live attach workflow:
    - On attach, the SDK registers for code-object iteration and creation callbacks so thread trace operates correctly on code objects that were loaded before the attach occurred.
    - Makes ATT usable on already-running production workloads without an application restart.
  - Experimental SQTT quick scan mode for thread trace, enabled through a new CMake flag:
    - Collects thread trace data without packet insertion or HSA signal manipulation, removing the queue interception overhead required by the standard ATT path.
    - Individual kernels can be traced without serialization, and the path is independent of the ROCm runtime version.
    - Experimental: intended to validate the new collection path and to enable out-of-process thread trace and long-kernel tracing in future releases.

  - HIP event tracing: GPU-side barrier tracing for `hipEventRecord` and `hipStreamWaitEvent`:
    - New tracing kinds `ROCPROFILER_CALLBACK_TRACING_HIP_EVENT` and `ROCPROFILER_BUFFER_TRACING_HIP_EVENT` with operation enum `rocprofiler_hip_event_operation_t` (RECORD, WAIT).
    - New `--hip-event-trace` CLI flag, automatically enabled by `--hip-trace` and `--hip-runtime-trace`.
    - rocpd schema bumped to 3.0.4 with new `rocpd_hip_event` table and `hip_events` data view.

**rocprof-trace-decoder:**

  - Python API for decoding Advanced Thread Trace (ATT) / SQTT data directly from Python, without writing a C++ consumer:
    - Wraps the decoder library and exposes thread trace decoding as a first-class Python interface, with samples demonstrating common workflows.
    - Useful for analysis scripts, Jupyter notebooks, and custom profiling tools that process ATT output programmatically.
    - Decoder integration tests have been migrated to Python, simplifying test authoring and making it easier for downstream tools to validate their trace-decoding pipelines.

##### Resolved issues

  - Fixed `rocprofv3` crashing during output generation when a second tool subscribed to code object tracing in the same process, which blocked profiling PyTorch and Triton workloads through rocprofiler-compute.
  - Fixed `rocprofv3` hanging instead of exiting when a fatal signal arrives while it is already handling one, for example when output generation aborts. It previously left GPU child processes running and required killing the process manually.

#### **ROCr Debug Agent** (2.2.0)

##### Added
- The `--output` and `--save-code-objects` options now support '%' format
  tokens to produce the output file names. The `%p`, `%h`, `%t`, `%e`, `%u`,
  `%g` and `%%` tokens are supported.

#### **rocSHMEM** (3.7.0)

##### Added
* New APIs:
    * `rocshmem_tile_{min, max, sum}_reduce{_wave}{_wg}` variants for the IPC backend
    * `rocshmem_ctx_tile_{min, max, sum}_reduce{_wave}{_wg}` variants for the IPC backend
* Tile RMA and collectives support for GDA backend.
* `--num_wf` argument to functional test harness for runtime wavefront size detection on architectures with wave size 32 (gfx1100, gfx1201, gfx1250).
* LTO inline-remarks comparison toolset and interactive resource-usage dashboards under `scripts/analysis/`.
* Optimized device API wrappers to remove double-indirection in cross-TU calls, enabling more reliable LTO inlining.
* Reduced IPC RMA latency by caching the symmetric heap base and size in device constant memory, removing hot-path heap metadata loads in `ipcPeerPtr()`.

##### Changed
* **Breaking build-system change**: The five compile-time CMake options
  `USE_HEAP_DEVICE_FINEGRAIN`, `USE_HEAP_DEVICE_UNCACHED`,
  `USE_HEAP_DEVICE_COARSEGRAIN`, `USE_HEAP_DEVICE_VMM_POSIX`, and
  `USE_HEAP_DEVICE_VMM_FABRIC` have been removed. Heap allocator
  selection is now a runtime decision via the environment variable:
  ```
  ROCSHMEM_HEAP_ALLOCATOR_TYPE=<uncached|finegrained|coarsegrained|vmm_posix|vmm_fabric>
  ```
  The default is architecture-dependent: `finegrained` for gfx1100/gfx1201,
  `uncached` for all other targets. The VMM allocator sources are now compiled
  unconditionally (when ROCm ≥ 7.2 / AMD SMI fabric handle support is
  detected) rather than behind build flags.

##### Resolved issues
* Fixed `reduce_wg` and `reduce_wave` operations producing incorrect results for some scenarios
* Fixed compilation of rocSHMEM when using clang++ instead of hipcc
* Fixed team-relative PE rank computation in GDA alltoallv
* Fixed `USE_SDMA=ON` builds failing for GDA+IPC with messages > 128 B
* Fixed libverbs/libmlx5 discovery using `.so.1` versioned symlinks in GDA backend
* Fixed Broadcom NIC (bnxt) dmabuf CQ/SQ creation
* Fixed ASan build: correct string size for socket, exclude libhsa

#### **rocSOLVER** (3.37.0)

##### Added

* 2-stage reduction to tridiagonal in the Hermitian eigensolver (SYEVD/HEEVD) and generalized Hermitian eigensolver (SYGVD/HEGVD).
* Cholesky QR methods for computing the QR factorization of a tall rectangular matrix
    - CHOLQR (with batched and strided\_batched versions)
    - CHOLQR_64 (with batched and strided\_batched versions)
* Hessenberg reduction auxiliary routine
    * LAHR2
* 64-bit APIs for existing functions:
    - LARFT_64

##### Optimized

* Improved the performance of sygst/hegst.

#### **rocSPARSE** (5.1.0)

##### Added
* The `rocsparse_spmat_scale` generic routine for sparse matrix scaling (`C = alpha * A`). It writes to `C` `alpha` times the values of `A` and does not copy the sparsity pattern (`C` is assumed to already have the same sparsity pattern as `A`). `alpha` is passed as a self-describing scalar dense vector descriptor that can reside in host or device memory, so no temporary storage buffer is required. In-place operation (`C == A`) is supported. COO, COO AoS, CSR, CSC, BSR, ELL, Blocked ELL, and SELL formats are supported.
* The `rocsparse_dnvec_descr_create_scalar` auxiliary routine, which creates a size-one dense vector descriptor for a host or device scalar.
* Batched support to the SpMM algorithm `rocsparse_spmm_alg_csr_nnz_split` and `rocsparse_spmm_alg_csr_merge_path`.
* `rocsparse_sddmm` batched support to CSR, CSC, COO, COO AoS, and ELL formats.
* ELL format support to `rocsparse_spsv` and `rocsparse_sptrsv`.
* The `rocsparse_solve_mode` enum (`triangular`, `diagonal`) and the `rocsparse_diagonal_modifier` enum (`none`, `absolute`) to enable diagonal-only solves in `rocsparse_sptrsv` and `rocsparse_sptrsm`, together with the `rocsparse_sptrsv_input_solve_mode` / `rocsparse_sptrsm_input_solve_mode` and `rocsparse_sptrsv_input_diagonal_modifier` / `rocsparse_sptrsm_input_diagonal_modifier` set-input values. The modifier selects the function applied to each diagonal value (`d` or `|d|`). CSR and CSC formats are supported.

##### Optimized
* Optimized architecture-aware launch configurations for RDNA (wave32) and CDNA (wave64) GPUs, improving performance and performance portability for several sparse level 2 and level 3 routines without algorithmic or numerical changes. Affected routines include `rocsparse_spmv` for the CSR adaptive, nnz-split, and LRB algorithms, the COO (SoA and AoS) formats, and the ELL format (`rocsparse_Xellmv`); `rocsparse_Xbsrmv`; `rocsparse_Xbsrxmv`; `rocsparse_Xgemvi`; `rocsparse_Xgemmi`; and `rocsparse_spmm` with the blocked-ELL format.

##### Resolved issues
* Fixed an integer overflow in `rocsparse_prune_dense2csr_by_percentage` and `rocsparse_prune_csr2csr_by_percentage`, which computed the matrix element count in 32-bit arithmetic. For matrices with more than `INT32_MAX` (~2.1 billion) elements the count overflowed to a negative value, resulting in out-of-bounds pointer construction and an invalid kernel launch grid. The element count is now computed in 64-bit arithmetic.
* Fixed `rocsparse_spmm` with the segmented COO, atomic COO, segmented-atomic COO, and row-split CSR algorithms, which failed with `hipErrorInvalidConfiguration` for batch counts exceeding 65535 because the batch dimension of the kernel launch grid exceeded the maximum supported grid dimension.
* Fixed an out-of-bounds write caused by an incorrectly sized temporary buffer in the CSR-to-CSC conversion performed by `rocsparse_sparse_to_sparse` when the row pointer and column index types differ (for example, 64-bit row pointers with 32-bit column indices).
* Fixed an issue with `rocsparse_spmm` when using the nnz-split algorithm with the CSR or CSC format. The operation produced incorrect results because the segmented-block-reduction helper had shared-memory pointer parameters marked `__restrict__`, while threads in the block must read values written by other threads. The `__restrict__` attribute has now been removed.

#### **rocThrust** (4.7.0)

##### Changed

* rocThrust now searches for an existing SQLite3 system library first by default. SQLITE_USE_SYSTEM_PACKAGE can be set to OFF to force a local download of SQLite3. The minimum required version of SQLite3 is 3.51.3.
* Updated the mechanism in which rocThrust looks for and includes libhipcxx to be compliant with libhipcxx packaging changes in ROCm 10.1.0.

#### **RPP** (3.2.0)

##### Added

* RPP is included in the ROCm Core SDK and is built with TheRock.
* Single-image processing support for 8 kernels (Brightness, Blend, Box Filter, Crop, Flip, Gaussian Filter, Median Filter, Resize Nearest Neighbor) to match performance with OpenCV.
* Runtime backend selection parameter (`RppBackend executionBackend`) for all RPP tensor API functions.
* Backend tracking in `rppHandle_t` to store backend type (HOST or HIP).
* `RPP_ERROR_HIP_LAUNCH` error type for reporting HIP kernel launch errors.
* `rpp_BACKEND_TYPE` and `rpp_AUDIO_AUGMENTATIONS_SUPPORT` variables to the RPP CMake package configuration.

##### Changed

* All RPP tensor API functions now use a single function signature.
* Updated all test suite calls to use the unified API with a backend parameter.
* Enhanced layout validation for image augmentations within the unified API.
* Test suite dependency on TurboJPEG removed. The test suite now uses `.rgb` files as input.
* OpenCV and pandas are now optional dependencies for the test suite.
* `CMakeLists.txt` updated to remove batch PD references.
* Updated the test suite to use `rpp_BACKEND_TYPE` and `rpp_AUDIO_AUGMENTATIONS_SUPPORT` from the RPP CMake package configuration instead of header parsing.
* `find_package(rpp)` now automatically passes public include directories to the target link interface.

##### Removed

* BatchPD legacy support.
* `LEGACY_SUPPORT` compilation flag and all code enclosed within it.
* OpenCL backend support.
* Batch PD test suite and installation.
