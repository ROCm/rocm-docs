#### **AMD SMI** (27.0.0)

##### Changed

- Bumped the library major version to 27.0.0 (breaking).
  - The shared library SONAME is now `libamd_smi.so.27`. Consumers linked against `libamd_smi.so.26` must relink; no source changes are required beyond the API changes listed elsewhere in this release.

- Restructured AMD SMI C++ tests into unit and functional suites.
  - The `amdsmitst` source tree now separates unit tests from hardware-backed functional tests under `tests/amd_smi_test/unit/` and `tests/amd_smi_test/functional/`.
  - GTest suite names now follow a `<Component><Type>[<Operation>]` scheme: functional tests are `<Component>FunctionalReadOnly`/`<Component>FunctionalReadWrite` (e.g. `GpuFunctionalReadOnly`) and unit tests are `<Component>Unit` (e.g. `GpuUnit`). This replaces the old `amdsmitstReadOnly`/`amdsmitstReadWrite` and `AmdSmiDynamicMetricTest` names.
  - Consumers that pass explicit `--gtest_filter` values should update those filters to the new suite names.
  - See the [AMD SMI test design](https://rocm.docs.amd.com/projects/amdsmi/en/docs-10.0.0/conceptual/test-design.html#naming-conventions) for the suite naming convention and `--gtest_filter` usage.

##### Optimized

- Optimized `amdsmi_get_gpu_process_list()` to skip redundant KFD topology discovery.
  - The per-process KFD lookup rebuilt the entire KFD node topology (an expensive sysfs walk) on every call just to translate the device BDF into its KFD GPU id.
  - The caller already knows this value, so it is now passed through to `gpuvsmi_get_pid_info()`, eliminating one full topology discovery per process per refresh. Falls back to the original discovery path when the id is unavailable.

##### Resolved issues

- Fixed `amd-smi ras --cper --json` emitting nothing when there are no CPER entries.
  - The common no-entries case printed empty output, so consumers feeding stdout to `json.loads` failed with `Expecting value: line 1 column 1 (char 0)`. The command now always emits exactly one valid JSON document: `[]` when there are no entries, or a single aggregated array across all GPUs when there are. `--follow` mode stays silent until entries appear. The human-readable primary-partition warning is also suppressed in JSON mode so it no longer corrupts the output.

- Fixed `amd-smi set --ptl-status` silently failing to change PTL state.
  - The set path wrote `"1"`/`"0"` to the `ptl/ptl_enable` sysfs node, which only accepts `"enabled"`/`"disabled"`; the driver ignored the numeric write while the API still reported success. The state now changes as expected, and a rejected write returns a real error instead of a generic success.

- Fixed `amd-smi process` hiding compute processes owned by other users.
  - A caller without permission to read another process's `/proc/<pid>/fd` was misdetected as running in a separate PID namespace, which caused the whole compute-process list to come back empty. Such processes are now listed with a redacted (`N/A`) name instead of being dropped.

- Fixed CU%/SDMA column alignment in the `amd-smi` process table.
  - The `SDMA` header no longer sits a column left of its values, and valid `CU %`/`SDMA` values are no longer truncated.

- Fixed compute processes being reported on every GPU.
  - A process was attributed to a GPU whenever it had a KFD context on that GPU, so a job with queues on a single GPU appeared under every GPU. Attribution now uses the process's active KFD queues plus any GPU where it holds a non-zero VRAM allocation, so a process is listed only against the GPUs it actually uses.

- Fixed `amd-smi` hanging in `amdsmi_init()` on UALink systems when the IFoE driver is unresponsive.
  - `amdsmi_init()` (and every CLI command) opened a per-GPU IFoE/UALoE fabric session up front, so it blocked indefinitely when the Broadcom IFoE driver was unresponsive, even for queries that never use fabric data.
  - The fabric session is now opened only on the first fabric query, so initialization and non-fabric queries no longer touch the IFoE driver.

- Fixed ctypes `DeprecationWarning` from `amdsmi_wrapper.py` on Python 3.14.
  - Python 3.14 deprecates the implicit ctypes structure layout when `_pack_` is set (slated to become an error in 3.19). Each packed structure/union in the generated wrapper now sets `_layout_ = 'ms'`, preserving the existing MSVC-compatible layout (no ABI change) while silencing the warning.


#### **Composable Kernel** (1.2.0)

##### Added

* Multiple D (bias) and large tensor support to the CK Tile quantized GEMM kernel for row-column quantization.

##### Changed

* Improved performance of row-column quantized a8w8 GEMM through better instruction scheduling in the eight-waves pipeline, wider epilogue stores, and nontemporal C/D memory access.

#### **HIP** (7.15.0)

##### Added
* New HIP APIs
    - Stream Ordered Memory Allocator: support for API parity with corresponding CUDA API.
      * `hipMemGetDefaultMemPool` returns the default memory pool for the specified location and allocation type
    - Cooperative Groups scan functions are now supported, providing feature parity with CUDA.
      * `cooperative_groups::exclusive_scan` performs an exclusive prefix scan across the threads in a cooperative group. For each thread, the result is computed from the values of all preceding threads using a binary operation (addition by default), excluding the current thread's own value.
      * `cooperative_groups::inclusive_scan` performs an inclusive prefix scan across the threads in a cooperative group. For each thread, the result includes the current thread's value in addition to the values of all preceding threads.
* Stream capture support for the following APIs, enabling `BatchMemOp` operations to be captured as graph nodes instead of executing immediately. Also improved `BatchMemOp` graph replay reliability through fixes to parameter handling and operation ordering, aligning behavior more closely with CUDA.
    - `hipStreamWaitValue32`
    - `hipStreamWaitValue64`
    - `hipStreamWriteValue32`
    - `hipStreamWriteValue64`
    - `hipStreamBatchMemOp`
* Support Non-Uniform Memory Access (NUMA) in `hipMemCreate` related APIs. HIP runtime added virtual memory support for `hipMemLocationTypeHostNuma` and `hipMemLocationTypeHostNumaCurrent` APIs. This enables NUMA-aware memory allocations backed by host CPU NUMA pools and aligns HIP virtual memory management behavior with CUDA host and host-NUMA VMM expectations.

##### Optimized

* Improved `hipMemcpy2D()` and `hipMemcpy2DAsync()` performance for copy operations with very small row widths and large row counts.
Previously, non-4-byte-aligned row or slice pitches could cause the runtime to issue a separate copy for each row, resulting in significant
performance degradation for workloads such as 1-byte-wide transfers with millions of rows.
These transfers are now handled using a single shader-based copy operation, significantly reducing transfer times.
Copy operations at or below the 256-row threshold are unchanged.
* Improved `hipEventRecord` performance by using the `hipEventDisableTiming` flag to avoid unnecessary profiling when timing information is not required. Event operations are now coalesced to eliminate redundant barrier submissions, reducing runtime overhead and improving execution efficiency.
* Improved batch copy performance: optimized `hipMemcpyBatchAsync` by splitting batch operations into per-device commands.
    - Simplified `rocrCopyBufferBatch` by using a single `src_agent` per engine group (H2D, D2H, and D2D).
    - Streamlined batch grouping:
      * Removed the `AgentGroup/src_agent` mapping for D2D broadcasts.
      * Processed `H2D` and `D2H` LINEAR operations directly, bypassing the broadcast map.

##### Resolved issues

* Resolved library loading error messages thrown by `rocminfo` during driver initialization in WSL (Windows Subsystem for Linux) environment due to failure in loading the HSA runtime library `libhsa-runtime64.so`
since it is not available in the dynamic linker search path. Since `rocminfo` already links against `libhsa-runtime64.so`, the runtime now correctly locates and loads the HSA runtime library using `RTLD_NOLOAD` option, enabling successful ROCm initialization, HSA agent discovery, and subsequent ROCm operations.
* Fixed a segmentation fault in HIP queue idle detection caused by referencing a recycled completion signal. Idle state is now derived from a queue-owned signal with a safe lifetime.
* Resolved incorrect NaN handling in the ordered not-equal comparison intrinsics `__hne` (for `__half`) and `__hne` (for `__hip_bfloat16`), along with their vector forms. Being *ordered* predicates, they now correctly return `false` when either operand is NaN.
* Resolved memory-safety issues in the ROCm code object and ELF loader by adding validation checks during code object module loading, preventing segmentation faults and improving runtime stability.
* Resolved a memory leak affecting mipmapped arrays when using `hipMemcpy2DToArray` with levels obtained via `hipGetMipmappedArrayLevel`. Mipmap level references are now properly released, ensuring that memory is correctly freed when `hipFreeMipmappedArray` is called.
* Fixed a deadlock that could occur when using ROCprofiler-sdk with ROCm-aware MVAPICH and MPICH. HIP runtime now performs profiler registration after dispatch table initialization, ensuring proper initialization ordering and guard release. This prevents hangs caused by reentrant initialization during profiler startup.
* Fixed a deadlock caused by `hipMemMap`/`hipMemUnmap` operations on the null stream that could lead to hangs. The HIP runtime now implements proper synchronization to all devices with access to a mapped pointer before unmapping it.
* Fixed an issue in `cooperative_groups::reduce()` that could cause incorrect results or kernel launch failures when block dimensions had .y or .z components not equal to 1.

#### **hipBLAS** (3.6.0)

##### Added

* Per-batch `alpha`/`beta` support for Level 2 batched and strided-batched forms of `symv`, `hemv`, `sbmv` and `spmv` via `hipblasSetBatchAlphaStride` and/or `hipblasSetBatchBetaStride` (device pointer mode).
* Per-batch `alpha` support for Level 2 batched and strided-batched forms of `syr` via `hipblasSetBatchAlphaStride` (device pointer mode).
* Per-batch `alpha` (scalar vector) API support for Level 1 batched and strided-batched forms of `scal` and the `_ex` forms through `hipblasSetBatchAlphaStride` when `hipblasHandle_t` is in mode `HIPBLAS_POINTER_MODE_DEVICE`.

##### Resolved issues

* PyTorch users can avoid user-constraint-based memory allocation failures (`HIPBLAS_STATUS_ALLOC_FAILED`) by exporting `HIPBLAS_WORKSPACE_CONFIG=:0:0` to allow rocBLAS managed memory to grow automatically.

#### **hipCUB** (4.6.0)

##### Added

* Feature parity with CCCL/CUB 3.0.0.
* `::hip::std` support.

##### Changed

* Changed `CCCL_MINIMUM_VERSION` to `3.0.0` to align with CUB.
* Add support for large num_items `DeviceMerge` and `DeviceSegmentedSort`.
* Replace `#pragma unroll` by `_CCCL_PRAGMA_UNROLL_FULL()` and `_CCCL_PRAGMA_NOUNROLL()` by `_CCCL_PRAGMA_NOUNROLL()`.
* Add `_CCCL_SORT_MAYBE_UNROLL()` in block merge sort and thread sort.
* Update `WarpExchange` template parameters for CUB compatibility.

##### Removed

* hipCUB compatibility with PyTorch v2.9 and v2.10 has been removed in this release. Use PyTorch v2.11 or later.
* Removed `hipcub::BaseTraits::CATEGORY`, `hipcub::BaseTraits::nullptr_TYPE` and `hipcub::BaseTraits::PRIMITIVE`.
* Removed `ConstantInputIterator`, `CountingInputIterator`, `DiscardOutputIterator` and `TransformInputIterator`, which were deprecated in hipCUB-4.1.0.
* Removed `DeviceSpmv`, which was removed from CUB after CCCL's 2.8.0 release. Use `hipSPARSE` or `rocSPARSE` libraries instead.
* Removed `GridBarrier`.
* Removed `HIPCUB_MIN`, `HIPCUB_MAX`, `HIPCUB_QUOTIENT_FLOOR`, `HIPCUB_QUOTIENT_CEILING`, `HIPCUB_ROUND_UP_NEAREST` and `HIPCUB_ROUND_DOWN_NEAREST` which were deprecated in hipCUB-4.1.0.
* Removed `LEGACY_PTX_ARCH`.
* Removed `hipcub:max` and `hipcub:min`, which were deprecated. Use `hip::std::max` and `hip::std::min` instead.
* Deprecated `hipcub::Swap`, use `rocprim::swap` instead.
* Deprecated `HIPCUB_IS_INT128_ENABLED`, use `_CCCL_HAS_INT128()` instead.
* Deprecated `hipcub::Equality`, `hipcub::Inequality`, `hipcub::InequalityWrapper`, `hipcub::Sum`, `hipcub::Difference`, `hipcub::Division`, `hipcub::Max` and `hipcub::Min` operators. Use `hip::std::equal_to`, `hip::std::not_equal_to`, `hip::std::plus`, `hip::std::minus`, `hip::std::divides`, `hip::maximum`, and `hip::minimum` operators instead.

#### **hipFFT** (1.0.25)

##### Changed

* Minor internal changes.

#### **hipFile** (0.4.0)

##### Added

* A KFD-based alternative check for P2P DMA support was added to `ais-check`. This inspects the `capability` property under `/sys/class/kfd/kfd/topology/nodes/*/properties`.
* Guides for setting up storage targets to the documentation.

##### Changed

* `ais-check` now lists the AIS-capable file system mounts detected on the system and fails if none are found.
* Fastpath-only tests are now automatically skipped on systems that do not support the AIS fastpath instead of failing. Running ctest in verbose mode (`ctest -V`) will provide the reason the test was skipped.
* Updated INSTALL.md to point to official install docs.

#### **hipSOLVER** (3.6.0)

##### Changed

* Minor internal changes.

#### **hipSPARSE** (4.7.0)

##### Added
* Blocked ELL format support to the `hipsparseDenseToSparse` routine, along with the new `hipsparseBlockedEllSetPointers` function.
* The `HIPSPARSE_SPMV_CSR_ALG3` algorithm to `hipsparseSpMV`, which exposes the rocSPARSE CSR nnz split algorithm (`rocsparse_spmv_alg_csr_nnzsplit`).
* CSC format support to `hipsparseSpSV` and `hipsparseSpSM`.

##### Resolved issues
* Fixed an issue with `hipsparseSpMM`, which produced incorrect results for the Blocked ELL sparse format.

#### **MIOpen** (3.6.0)

##### Added
* Gfx950 (MI350X/MI355X) 7x7 depthwise forward and backward-data convolution support (fp16/bf16), fixing a slow fallback-to-naive-kernel regression in ConvNeXt-style depthwise convolutions.

##### Changed
* Restored gfx12x support in the Winograd Rage solver, recovering performance that regressed when earlier gfx12 support was reverted.
* Refreshed the gfx1100, gfx1102, and gfx1201 (Navi) SystemDBs with updated tuned find/perf-database entries.
* Refreshed the gfx950 SystemDB with additional tuned entries to cover more models.

##### Removed
* Removed the OpenCL (OCL) backend; MIOpen now supports the HIP backend only.

##### Resolved Issues
* Fixed an off-by-stride indexing bug in the backward CalcStats mean/variance remainder loop that caused a ~1% systematic bias in NCHW batch normalization backward results.
* Fixed an integer overflow in tensor operation kernels for large allocations that could cause memory access faults.
* Fixed a naive convolution solver failure mode where a global work size of 2^32 or more work-items silently failed to launch and left a stale HIP error visible after Find returned success; such launches are now rejected up front.

#### **RCCL** (2.30.7)

##### Added
* Compatibility with NCCL 2.30.7.
* Scalable AllGatherV pattern: grouped `ncclBroadcast` calls with distinct roots are fused into a single ring kernel, improving performance at large scale. Gated by `NCCL_ALLGATHERV_ENABLE` (default off).
* GPU-only multi-segment registration for symmetric memory windows, enabling contiguous VA ranges backed by multiple physical segments (single-node validated).
* Elastic Buffer support for symmetric windows spanning device and host/`HOST_NUMA` memory segments (`NCCL_ELASTIC_BUFFER_REGISTER`, `NCCL_SYM_REUSE_SYSMEM_HANDLES`). Single-node path validated; multi-node registration remains limited pending HIP/HSA multi-segment DMA-BUF export support.

##### Changed
* Adapted the device-initiated GIN backends (Anvil SDMA and rocSHMEM GDA) to the NCCL 2.30.7 GIN API v14:
  * Added the new `getGinProperties` host op.
  * Dropped the data-path ops (`iput`/`iputSignal`/`iget`/`iflush`/`test`) that moved out of GIN under the GIN/RMA split.
  * Switched `createContext` to `ncclGinConfig_v14_t`.
  * Updated the device dispatch signatures, and matched the GIN type renumbering (`ROCSHMEM_GDA` and `ANVIL_SDMA` shifted after the new `GIN_GPI` type).
  * The plugins now use the generic (unversioned) `ncclGin_t` / `ncclGinConfig_t` / `ncclGinProperties_t` typedefs so future ABI bumps do not require touching call sites.
* Updated the ROCSHMEM GIN plugin registration to the v14 layout (corrected struct field names and the conditional that previously only compiled without ROCSHMEM GIN).
* Adapted the InfiniBand transports (`net_ib` and `net_ib_cast`) to the v14 GIN/RMA split: the host/proxy backend is now registered as an `ncclRma_t` vtable (`RMA_IB_PROXY`) that owns the `iput`/`iputSignal`/`iget`/`iflush`/`test` data-path ops, with GIN layered on top through the generic `ncclGinProxy`.

##### Known issues
* The improved AllGatherV support breaks the NCCL profiler support for ncclBroadcast operations, limiting visibility to API events. `NCCL_ALLGATHERV_ENABLE=0` can be used as a workaround until it is fixed in a future release.
* Multi-node multi-segment and Elastic Buffer symmetric-window registration is not yet enabled; NET and LSA+GIN multi-segment paths depend on runtime support for exporting contiguous DMA-BUF handles across all physical segments.

#### **rocBLAS** (5.6.0)

##### Added

* Per-batch `alpha`/`beta` support for Level 2 batched and strided-batched `symv`, `hemv`, `sbmv`, and `spmv` via `rocblas_set_batch_alpha_stride` and `rocblas_set_batch_beta_stride` (device pointer mode).
* Per-batch `alpha` support for Level 2 batched and strided-batched `syr` via `rocblas_set_batch_alpha_stride` (device pointer mode).
* Per-batch `alpha` (scalar vector) API support for Level 1 `scal_batched`, `scal_strided_batched`, and their `_ex` forms through `rocblas_set_batch_alpha_stride` when `rocblas_handle` is in `rocblas_pointer_mode_device`.
* Support custom build with CMake arguments `BUILD_WITH_HIPBLASLT_ONLY=ON` that bypasses legacy Tensile.

##### Upcoming changes

* Deprecated the `ROCBLAS_USE_HIPBLASLT_BATCHED` environment variable. Recent optimizations mean hipBLASLt no longer needs to be disabled for batched operations only. This environment variable is planned for removal in a future release.

#### **rocDecode** (1.9.0)

##### Added

* Invalid video size handling for AVC/HEVC.

##### Resolved issues

* Fixed decode errors of some AVC interlaced container streams by adding support for the picture data packet from the demuxer which contains multiple pictures.
* Corrected fake CTest passes.

#### **rocFFT** (1.0.39)

##### Added

* Optional ROCm Communication Collectives Library (RCCL) backend for single-node multi-GPU communication, enabled via `-DROCFFT_RCCL_ENABLE=ON`.

##### Changed

* Relaxed the usage requirements for `rocfft_setup` and `rocfft_cleanup`.
* Removed the ROCFFT_RTC_PROCESS_HELPER debug environment variable.

##### Optimized

* Improved performance of unit-strided, interleaved, real-to-complex FFTs on gfx1201, gfx90a, gfx942, and gfx950 for the following lengths:
  * (100,100,100)
  * (192,96,96)
  * (200,96,96)
  * (128,128,256)
  * (160,168,168)
  * (160,168,192)
  * (168,168,192)
  * (168,192,192)
  * (192,192,192)
  * (192,192,200)
  * (192,200,200)
  * (200,200,200)
  * (216,216,216)
  * (216,104,100)
  * (216,104,104)
  * (224,104,104)
  * (224,108,104)
  * (224,108,108)

##### Resolved issues

* Addressed internal issues causing multi-device plans to fall back to the least-performant code path for certain 3D real transforms (e.g., multi-device single-precision real out-of-place 3D of size 320x320x320 using slab decomposition).
* Fixed a thread-safety issue that could cause `rocfft_plan_create` to crash when called concurrently from many threads.

#### **ROCgdb** (16.3)

##### Added

* The address space operator `#` is recognized in Fortran programs too. This allows evaluating expressions like `private_lane#0x08` in Fortran applications that offload kernels to an AMD GPU.

#### **rocJPEG** (1.7.0)

##### Added

* `rocJpegDecodeAsync` and `rocJpegDecodeSync` APIs to support asynchronous single-image JPEG decoding, allowing decode submission and completion to be separated across threads for improved pipeline throughput.

#### **ROCm Compute Profiler** (3.8.0)

##### Added

* ``--pc-sampling-rows`` analyze option to cap the PC sampling table at the top N rows (default 10); set ``0`` to show all. Must be non-negative.

* ``--overwrite`` profile mode option to explicitly allow replacing existing workload output.

* Experimental Triton support to ML API tracing. Profile with `--experimental --triton-trace` to emit a ROCTX marker per Triton/Inductor kernel launch attributed to the user call site, and analyze with `--experimental --list-triton-operators` or `--experimental --triton-operator <pattern>` to list or filter Triton operators independently of Torch.

* Support for GPU metrics on gfx1153 hardware.

##### Changed

* Split Python version requirements by mode. Profile mode now runs on Python 3.8+ (standard library only). Analyze mode requires Python 3.9+ and exits with a clear message on older interpreters instead of failing with an import error.

* `--pc-sampling-sorting-type` now defaults to `count` (was `offset`), so the PC sampling table shows the most-sampled instructions first.

* Renamed the `Pct of Peak` / `PoP` analysis column to `Percent of Peak` in analysis output.

* `--torch-trace` now wraps the tensor methods `to`, `cpu`, `cuda`, and `contiguous` by default. Previously these wraps were enabled by setting `ROCPROFCOMPUTE_ROCTX_DEEP_TENSOR_WRAPS=1`. Set `ROCPROFCOMPUTE_ROCTX_DEEP_TENSOR_WRAPS=0` (or `false`, `no`, `off`) to disable them.

* Renamed the torch-trace output files and directory from `torch_trace_*` to `ml_api_trace_*`.

* Profile mode now errors when the target workload directory is non-empty unless `--overwrite` is passed. `--bench-only` likewise requires `--overwrite` before replacing an existing `roofline.csv`.

* Renamed `num_hbm_channels` to `num_memory_channels` in machine specifications to unify memory channel reporting across GPU families.

##### Removed

* Removed the multi-node analysis options ``--nodes``, ``--list-nodes`` (analyze mode) and the experimental ``--spatial-multiplexing`` option (profile and analyze modes). These features did not work as expected and will be redesigned in a future release.

##### Optimized

* Improved GPU Benchmarking and Roofline profiling/analysis support for gfx1150/gfx1151/gfx1152 architectures.
  * gfx11xx supports Wave Matrix Multiply Accumulate (WMMA), replacing MFMA operations.

##### Resolved issues

* The Dual VALU (VOPD) instruction mix metric is now reported for gfx115x in the WGP panel.

* Fixed multi-user roofline benchmarking on shared systems: the per-GPU lock file under `/tmp/rocprof-compute-benchmark/` is now created world-readable/writable (0666) so any user can acquire it, regardless of which user created it first or the active umask. Stale unreadable lock files left by older versions in a sticky `/tmp` cannot be repaired automatically and must be removed manually by their owner or an administrator.

* Fixed CDNA memory chart CLI output to show the numbered `3. Memory Chart` header without repeating the default per-kernel normalization label.

##### Known issues

* Workloads profiled with earlier versions must be re-profiled before analysis. The sysinfo schema changed and older workload directories are not compatible.

* CLI mode block 4 Roofline plot's legend will not appear if there are too many kernels to list, in relation to the user's terminal size. Same per-kernel roofline rate metrics and AI plot point details can be read in block 4's preceding tables.

#### **ROCm Systems Profiler** (1.8.0)

##### Added

- hipFile (GPU-direct storage) API tracing. Add `hipfile_api` to
  `ROCPROFSYS_ROCM_DOMAINS` (shorthand: `hipfile`) to capture hipFile API traces. Requires ROCprofiler-SDK version 1.3.5 or later.

- `--exe-only` flag for `rocprof-sys-instrument`: shorthand for excluding every shared
  library from instrumentation, leaving only the main executable.

- `--exclude-internal-lib-paths` flag for `rocprof-sys-instrument`: by default, each
  internal library is excluded only at the path linked at startup; when enabled, every
  on-disk path matching an internal library's filename is excluded.

- `--max-library-functions` option for `rocprof-sys-instrument`: skips shared libraries
  whose procedure count exceeds the given threshold, keeping instrumentation overhead
  manageable. The target executable is never gated by this, and the check is bypassed by
  the module include/restrict (`--module-include`/`-MI`, `--module-restrict`/`-MR`) and
  function include/restrict (`--function-include`/`-I`, `--function-restrict`/`-R`)
  regexes.

- rocSHMEM host-stream API tracing via `ROCPROFSYS_ROCM_DOMAINS=rocshmem_api`.
  ROCm Systems Profiler now captures the nine host-stream rocSHMEM API calls
  (`putmem_on_stream`, `getmem_on_stream`, `putmem_signal_on_stream`,
  `signal_wait_until_on_stream`, `broadcastmem_on_stream`, `alltoallmem_on_stream`,
  `barrier_all_on_stream`, `sync_all_on_stream`, `quiet_on_stream`) as
  `rocm_rocshmem_api` spans in Perfetto traces and rocpd databases. Requires
  rocprofiler-sdk >= 1.3.4 and rocSHMEM >= 3.6.0 (included in ROCm 10.0.0).
  As of rocSHMEM 3.6.0, `USE_ROCPROFILER_REGISTER` defaults to `ON`, so
  package installations automatically include this support. A `rocshmem` example
  demonstrating two-PE usage of all nine APIs is included under `examples/rocshmem`.

##### Changed

- `ROCPROFSYS_BUILD_TESTING` no longer implies `ROCPROFSYS_BUILD_EXAMPLES`.

- Introduced the new `profiler-hub` writer backend for trace persistence, as a
  replacement for the existing SQLite3/rocpd backend.

##### Removed

- Removed the `-p` / `--pid` option from `rocprof-sys-instrument` for attaching to
  an already running process. Use the `rocprof-sys-attach` executable instead, which
  attaches to and profiles running processes via the ROCprofiler-SDK `rocattach` API.

- Removed `--parse-all-modules` from `rocprof-sys-instrument`. The tool iterates through objects and modules to extract the functions by default.

#### **rocPRIM** (4.6.0)

##### Added

* A parallel `device_topk`, which finds the largest/smallest K elements from an input array of keys.
* A parallel `device_segmented_topk`, which finds the largest/smallest K elements from segmented groups.
* `device_topk` and `device_segmented_topk` are now controlled by the CMake flag `ROCPRIM_ENABLE_TOPK`. Set `-DROCPRIM_ENABLE_TOPK=ON` to enable these features.

##### Changed

* Combined and simplified separate assertion templates using `std::is_floating_point`, `rocprim::half`, and `rocprim::bfloat16` to use `rocprim::is_floating_point`.

#### **ROCprofiler-SDK** (1.3.5)

##### Added

**API:**

  - rocSHMEM host-stream API interception for the rocSHMEM tracing domain introduced in 1.3.0:
    - `rocshmem_putmem_on_stream`, `rocshmem_getmem_on_stream`, and `rocshmem_alltoallmem_on_stream` are intercepted and emitted as per-call trace records.
    - Records are interleaved with HIP, HSA, RCCL, and other runtime traces so rocSHMEM communication activity can be viewed on the same timeline as GPU compute.
  - hipFile API tracing as a first-class tracing domain:
    - hipFile API calls are intercepted through dispatch-table wrapping and emitted as per-call trace records alongside HIP, HSA, and other runtime activity.
    - Enables file I/O operations to be correlated with GPU kernels and memory copies in a single profiling timeline.
  - Streaming Performance Monitor (SPM) counter data in the rocpd output format:
    - SPM records are stored as `rocpd_track` rows labelled `SPM`, with counter values grouped by timestamp into `rocpd_sample` rows and per-dimension data in `rocpd_pmc_event` rows.
    - The rocpd schema gains the `sample_id`, `xcc`, `shader_engine`, and `instance` columns.
    - SPM data is consumable by any tool that reads the rocpd database and is convertible to CSV via `rocpd convert`. Conversion to the other output formats, such as Perfetto and OTF2, is not yet supported.

**rocprofv3 (CLI):**

  - OpenMP (OMPT) tracing via the new `--ompt-trace` flag:
    - Accepts a bare boolean or a space-separated category list (`all` `thread` `parallel` `task` `sync` `mutex` `target` `device` `error`), following the same style as `--pmc` and `--output-format`; for example, `--ompt-trace parallel task target sync`. Categories must be space-separated; comma-separated tokens are rejected. Also folded into `--sys-trace`/`--runtime-trace`.
    - rocpd-only trace: records go to the rocpd database (the default output format) and are exported via `rocpd convert`.
    - The OMPT callback layer is already supported by ROCprofiler-SDK; this flag makes it accessible without writing a custom tool.
  - hipFile API tracing via the new `--hipfile-trace` flag (or the `ROCPROF_HIPFILE_API_TRACE` environment variable):
    - Automatically included in `--runtime-trace` and `--sys-trace`.
    - Records are emitted across all supported output backends: CSV, JSON, Perfetto, OTF2, and rocpd.
  - Container-aware `rocattach` symbol resolution: attach entry points are resolved directly from the target process mapped ELF, and tool paths are validated from the target's perspective before injection. This allows attaching from a host to a containerized process without manually copying `.so` files.

##### Changed

- Previously, `rocattach` calculated symbol offsets from the host's `librocprofiler-register.so` and applied them to the target's mapping, which failed when the host and container libraries differ in ELF layout or path. Offsets are now resolved from the target process itself.
- Idle inline queues with no active profiling consumers now bypass queue interposition entirely, reducing overhead for applications that create queues but do not immediately dispatch work.
- DWARF information is now parsed lazily, reducing startup overhead for attach and tracing sessions on large binaries.
- Disabled autoflush in thread trace to prevent premature buffer flushes that produced incomplete or corrupted traces.
- Bump rocpd schema to version 3.0.1 which supports NIC agent types.
- Bump rocpd schema to version 3.0.2 for HIP graph per-node attribution (`graph_exec_id`/`graph_node_id` columns on `rocpd_kernel_dispatch`/`rocpd_memory_copy` and the new `rocpd_graph_launch` table). The pre-graph-attribution 3.0.1 schema is now frozen under `versions/3.0.1/` per the rocpd schema versioning scheme.
- Bump rocpd schema to version 3.0.3 for SPM support. The pre-spm-support 3.0.2 schema is now frozen under `versions/3.0.2/` per the rocpd schema versioning scheme.

##### Removed

- Dependency on `libatomic`. The library was previously linked unconditionally through the `rocprofiler-sdk-atomic` interface target, which caused link failures on toolchains and container images where `libatomic1` is not installed. The single `std::atomic` use that required it has been replaced with explicit memory-ordering synchronization; behavior is unchanged.

##### Resolved issues

- A GPU stall in device thread trace that occurred when thread trace was started before `hsa_init()`.
- A counter-collection stall caused by an `InterceptQueue` ordering bug, and fixed an out-of-bounds write in `Submit()`.
- `roctxMark` calls propagating as kernel rename labels, which caused spurious kernel name changes in traces containing ROCTx markers.
- SQ aliasing on harvested WGPs and multi-counter desync on gfx11xx targets in AQLprofile, and corrected the `GcEaSeCounterBlockMaxEvent` value.
- A guard to prevent double-initialization of the PC sampling service.
- `rocprofv3` attach sessions exiting before all buffered output was flushed; attach sessions now block until the flush completes.
- The ordering of code object callbacks during attach, which could race with tools that depend on ordered delivery.
- The `fmt/format.h` include path, the `fpic` flag for samples, OMP lookup in CI, and clang-tidy quickscan enablement.

##### Known issues

- SPM sessions can remain in a stale state after abrupt termination. See [GitHub issue #6489](https://github.com/ROCm/rocm-systems/issues/6489) for details.

#### **rocRAND** (5.0.0)

##### Removed

* Removed `h_scrambled_sobol(32|64)_constants`, `rocrand_h_scrambled_sobol(32|64)_direction_vectors`, `rocrand_h_sobol(32|64)_direction_vectors` from public namespace.

#### **rocSHMEM** (3.6.0)

##### Added

* New APIs:
   * `rocshmem_broadcast_wave`
   * `rocshmem_fcollect_wave`
   * `rocshmem_alltoall_wave`
   * `rocshmem_reduce_wave`
   * `rocshmem_reducescatter_wave`
* Support for some tile-granular collectives for the IPC backend:
   * `rocshmem_tile_broadcast`
   * `rocshmem_tile_broadcast_wave`
   * `rocshmem_tile_broadcast_wg`
   * `rocshmem_ctx_tile_broadcast`
   * `rocshmem_ctx_tile_broadcast_wave`
   * `rocshmem_ctx_tile_broadcast_wg`
   * `rocshmem_tile_allgather`
   * `rocshmem_tile_allgather_wave`
   * `rocshmem_tile_allgather_wg`
   * `rocshmem_ctx_tile_allgather`
   * `rocshmem_ctx_tile_allgather_wave`
   * `rocshmem_ctx_tile_allgather_wg`
* Single node support for gfx1250 / MI455X.
* Support for HIP Fabric Handles.

##### Changed

* Dropped LLC dependency when compiling HSCO objects.

#### **rocSOLVER** (3.36.0)

##### Added

* 64-bit APIs for the symmetric/Hermitian eigensolvers:
    * SYEV_64 and HEEV_64 (with batched and strided_batched versions)
    * SYEVD_64 and HEEVD_64 (with batched and strided_batched versions)
* Support added for the gfx1250 architecture.

##### Changed

* Clarified the `geblttrf_npvt` API documentation to accurately describe the in-place LU block-factorization storage.

##### Known issues

* The 64-bit eigensolver APIs (SYEV_64, HEEV_64, SYEVD_64, HEEVD_64) require the matrix
  dimensions `n` and `lda` to fit within a 32-bit integer, because their internal tridiagonal
  reduction and back-transformation steps remain 32-bit.

#### **rocSPARSE** (5.0.0)

##### Added
* Blocked ELL format support to the `rocsparse_dense_to_sparse` routine, including the new `rocsparse_bell_set_pointers` function to set the Blocked ELL array pointers.
* CSC format support to `rocsparse_spsv` and `rocsparse_sptrsv`.
* CSC format support to `rocsparse_spsm` and `rocsparse_sptrsm`.
* `rocsparse_handle_create` to create a handle associated with a user-provided stream. All internal device memory allocation and initialization are stream-ordered on that stream, so handle creation never blocks the calling thread or other GPU streams.
* `rocsparse_handle_destroy` to destroy a handle created by `rocsparse_handle_create`, with an optional error descriptor argument.

##### Changed
* `rocsparse_spmm` with CSR/CSC and the default algorithm (`rocsparse_spmm_alg_default` or `rocsparse_spmm_alg_csr`) now automatically selects a load-balanced (nnz-split) kernel for strongly skewed matrices (those containing a single very long row for CSR, or column for transposed CSC). Behavior is unchanged for non-skewed matrices and for explicit algorithm choices (`rocsparse_spmm_alg_csr_row_split`, `rocsparse_spmm_alg_csr_nnz_split`, `rocsparse_spmm_alg_csr_merge_path`).

##### Removed
* The deprecated `rocsparse_indextype_u16` enum.

##### Resolved issues
* Fixed an issue with `rocsparse_spmm`, which produced incorrect results for the Blocked ELL sparse format.

##### Upcoming changes

* Deprecated the `rocsparse_spildlt0_input_diag` enum value. It was used to dump the diagonal `D` of the ILDLT(0) factorization, which is now stored in-place on the diagonal entries of the `L` factor. It will be removed in a future release.

#### **rocThrust** (4.6.0)

##### Added

* Largely in feature parity with CCCL/thrust v3.0.3.
  - `thrust::tuple`, `thrust::pair` and `thrust::zip_iterator` fall back to rocThrust 4.4.0 implementations when a libhipcxx counterpart corresponding to CCCL/libcudacxx >= v3.0.3 is unavailable.
    * `thrust::tuple` and `thrust::pair`: some features may differ from CCCL/thrust v3.0.3.
    * `thrust::zip_iterator`: some iterator concepts present in CCCL/thrust v3.0.3 are missing.

##### Removed

* rocThrust compatibility with PyTorch v2.9 and v2.10 has been removed in this release. Use PyTorch v2.11 or later.
