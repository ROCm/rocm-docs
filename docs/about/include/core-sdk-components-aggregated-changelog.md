#### **AMD SMI** (27.2.0)

##### Added

- **Added unified `amdsmi_get_link_topology()` API on baremetal**.  
  - Returns link weight, status, type, hop count, and framebuffer-sharing data.
  - Matches the existing host interface; includes Python and Rust bindings.
  - Exports `AmdSmiLinkStatus` in Python and `AmdsmiLinkStatusT` in Rust for status interpretation.
  - Self pair (same GPU): `INTERNAL`, `ENABLED`, zero hops and weight, `fb_sharing=1`.
  - Migration from component APIs on baremetal:
    - `amdsmi_topo_get_link_type(gpu, gpu)` reports PCIe and 2 hops instead.
    - `amdsmi_topo_get_link_weight(gpu, gpu)` reports twice the NUMA node weight (typically 40).

- **Added IFoE fabric RAS CPER API**.
  - `amdsmi_get_fabric_cper_entries()` — retrieve IFoE RAS CPER records from UALoE library
  - CLI: `amd-smi ras --cper` now reports fabric link events alongside GPU errors (fabric-linkdown, fabric-linkup, fabric-fatal)

- **Added AMPP (amdsmi power profile) API and CLI support**.
  - `amdsmi_get_ampp_profiles`, `amdsmi_get_ampp_fields`, `amdsmi_activate_ampp_profile`, and `amdsmi_configure_ampp_profile` expose the driver's `app_modes/` sysfs tree of predefined power/performance recipes. Profiles, fields, and units are all dynamically enumerated at runtime — nothing is hardcoded, since the driver does not guarantee a fixed field set, profile count, or naming across SoC generations.
  - `amd-smi static --ampp` displays the tree-wide `app_modes/profile_abi` version plus per-profile active/writable/configured state and per-field name/unit/value/min/max.
  - `amd-smi set --ampp-activate <profile_name>` activates a profile.
  - `amd-smi set --ampp-configure <profile_name> KEY=VALUE...` stages one or more fields on a writable profile and commits them.
  - `amd-smi set --ampp-configure @<path>` restores every writable profile with staged fields from an `amd-smi static --ampp --json`-shaped file in one call.
  - This is unrelated to the legacy `amdsmi_*_gpu_power_profile*` preset-mask API, which is unchanged.

- **Added `AMDSMI_VRAM_TYPE_HBM4` to `amdsmi_vram_type_t`**.  
  - Identifies HBM Generation 4 VRAM, reported by `amdsmi_get_gpu_vram_info()`.
  - Also added the pre-existing `HBM3E` value to the Python `AmdSmiVramType` enum, which had been missing it.

- **Added NPM (Node Power Management) power limit setting**.  
  - New `amd-smi set --node-power-limit`/`-n` to set the node-level power limit in watts (node-wide, not per-GPU).
  - New API: `amdsmi_set_npm_limit()`.
  - New `current_node_power` field in `amdsmi_npm_info_t` (returned by `amdsmi_get_npm_info()`), the current (instantaneous) node power in watts, queried once per node rather than once per GPU; `amd-smi node -p` now displays it when available.
  - New `max_node_power_limit` field in `amdsmi_npm_info_t` (returned by `amdsmi_get_npm_info()`), the platform max bound for `amdsmi_set_npm_limit()` requests.
  - `amdsmi_set_npm_limit()` rejects the request with `AMDSMI_STATUS_INVAL` when NPM is disabled on the node (`amdsmi_npm_info_t::status == AMDSMI_NPM_STATUS_DISABLED`), instead of writing a limit that would have no defined effect. `amd-smi set --node-power-limit` performs the same check up front to fail fast with a friendlier message.

- **Added NPM (Node Power Management) balancing mode get/set**.  
  - New `amdsmi_npm_balancing_mode_t` enum: `AMDSMI_NPM_BALANCING_MODE_POWER_BALANCING` (PB, the default) or `AMDSMI_NPM_BALANCING_MODE_FREQUENCY_BALANCING` (FB).
  - New APIs: `amdsmi_get_npm_balancing_mode()` and `amdsmi_set_npm_balancing_mode()`. Getting always reflects the last-selected mode regardless of NPM enablement, returning `AMDSMI_STATUS_NOT_SUPPORTED` only when the underlying value is missing or unreadable.
  - Setting is AMD-SMI-only (not exposed via BMC Redfish/APML) and requires elevated privileges; it returns `AMDSMI_STATUS_NOT_SUPPORTED` without touching sysfs when NPM is disabled on the node.
  - CLI: `amd-smi node -p` now shows `BALANCING_MODE`; new `amd-smi set --node-balancing-mode {POWER_BALANCING,FREQUENCY_BALANCING}` (node-wide, not per-GPU, requires sudo).
  - New `AMDSMI_NPM_BALANCING_MODE_MAX` enum value marks the end of the defined range, and new `amdsmi_get_npm_supported_balancing_modes()` reports the balancing modes supported by a node's platform/ASIC as a bitmask (bit N for enum value N), independent of NPM enablement. `amdsmi_set_npm_balancing_mode()` now returns `AMDSMI_STATUS_SETTING_UNAVAILABLE` when the requested mode is absent from this bitmask, distinct from the `AMDSMI_STATUS_NOT_SUPPORTED` NPM-disabled case. CLI: `amd-smi node -p` now also shows `SUPPORTED_BALANCING_MODES`, and `amd-smi set --node-balancing-mode` reports a distinct "not supported on this platform" message for a mode absent from the bitmask vs. the generic "NPM is disabled" message.

- **Added a receipt timestamp to `amd-smi event` records**.
  - `AmdSmiEventReader.read()` now attaches a `timestamp` (Unix epoch seconds) to every event record, and the `event` subcommand surfaces it as a `TIMESTAMP` field so events can be correlated across streams.
  - Human-readable output renders one indented block per event (including `TIMESTAMP`); `--csv` writes the header once followed by a row per event, and `--json` emits newline-delimited JSON (one object per line).

##### Changed

- **Use fwupd daemon to read and write UMA carveout information**.  
  - `amdsmi_get_gpu_uma_carveout_info()` / `amdsmi_set_gpu_uma_carveout()` (and therefore `amd-smi static/set --mem-carveout`) now read and write the carveout through the fwupd BIOS-settings interface on integrated GPUs and fallback to amdgpu `.../device/uma/carveout` sysfs node when the fwupd daemon is absent.
  - Implemented in the AMD SMI library over the fwupd daemon's D-Bus BIOS-settings interface; the fwupd path is used only for the integrated (APU) GPU, and PolicyKit brokers authorization instead of requiring explicit root. Reading needs fwupd >= 1.8.4; writing needs fwupd >= 2.1.1 (Ubuntu 26.04+).

- **Reworked `amd-smi` CLI process exit codes.**  
  How exit codes are chosen:
  - Library failures exit with the underlying `AMDSMI_STATUS_*` value (0-56).
  - CLI-only errors use dedicated codes in the reserved 193-253 band.
  - When more than one failure with different codes is recorded in a single run — across devices, or across sub-steps of one command (e.g. `reset --clocks`) — the process exits `204`.

  Multi-device `set`/`reset` behavior:
  - A per-device GPU failure is now recorded, and the command continues to the remaining devices.
  - The exit code is decided once, after every device has been attempted. Previously a single device failure could abort the loop early.

  Notable value changes (many cases previously aliased onto `1`/`2`; each now has a distinct code):

  | Case | Was | Now |
  | --- | --- | --- |
  | Import failure | `1` | `193` |
  | Invalid command | `1` | `194` |
  | Invalid parameter | `2` | `195` |
  | Device not found (bad device index, or no devices present) | `3` / `255` | `196` |
  | Invalid file path | `4` | `197` |
  | Invalid parameter value | `5` | `198` |
  | Missing parameter value | `6` | `199` |
  | Command not supported (CLI, parse-time) | `7` | `200` (distinct from library `NOT_SUPPORTED` = `2`) |
  | Device interface unavailable (CLI, runtime) | — | `201` |
  | Required target/argument missing | `9` | `202` |
  | Invalid subcommand | `10` | `203` |
  | Permission denied | `11` | `10` (real `AMDSMI_STATUS_NO_PERM`) |
  | Mixed device/field failures (differing codes) | — | `204` |
  | `amdsmi_init()` watchdog timeout | `2` | `205` |
  | Drivers not loaded | `255` | `206` |
  | Interactive confirmation declined (answering no, or closing stdin) | `1` | `207` |
  | Library (device) failure | `(1000 + status)` wrapped to a byte | underlying `AMDSMI_STATUS_*` (`0`-`56`) |
  | Unknown/unmapped library error | `231` | `255` (library `UNKNOWN_ERROR` reported as its low byte) |
  | Library status too large to report as an exit code | — | `253` (not reachable with current library versions) |

  The `Was`/`Now` values are process exit codes (`$?`). Previously the printed `code` field showed the *negative* of these (e.g. `-7`) while the process exited with the absolute value (`7`); the rework makes the printed `code` and `$?` agree (both `200`).

- **`amd-smi set --power-cap` now applies per GPU instead of aborting on the first out-of-range device.**  
  - Each GPU is validated against its own reported range.
  - An out-of-range value on one GPU is reported for that GPU, while in-range GPUs still apply.
  - The process exit code reflects the per-device failure.

- **Clearer, more consistent `amd-smi` error messages.**  
  - Errors now read `[AMDSMI_STATUS_<name>] <message>`, so the underlying status is obvious at a glance. The internal class name is no longer prepended, and `--json`/`--csv` errors are now valid JSON/CSV.
    - Before: `amdsmi_cli_exceptions.AmdSmiLibraryErrorException: AMDSMI has returned error '-1002' - 'Command not supported' Error code: -1002`
    - After (human): `[AMDSMI_STATUS_NOT_SUPPORTED] Command not supported Error code: 2 [AMDSMI_STATUS_NOT_SUPPORTED]`
    - After (`--json`): `{"error": "[AMDSMI_STATUS_NOT_SUPPORTED] Command not supported", "code": 2, "error_type": "AMDSMI_STATUS_NOT_SUPPORTED"}`
  - `--json` and `--csv` errors gain an `error_type` field (e.g. `INVALID_PARAMETER_VALUE` or an `AMDSMI_STATUS_*` name) alongside the numeric `code`. `--csv` gains a third column (`error,code,error_type`); a reader that unpacks exactly two fields per row will need updating.
  - `amd-smi ras` reports a readable error when a CPER file can't be read or decoded, instead of printing a Python traceback.

- **`amd-smi ras` now exits non-zero when a CPER file fails to decode.**  
  - Affects `--afid --folder`, `--afid --cper-file`, and `--cper`, which previously exited `0`.
  - The exit code is the underlying `AMDSMI_STATUS_*`, for example `43`. When files fail with different codes, the command exits `205`.
  - The failing file shows `[AMDSMI_STATUS_<name>] <message>` in place of its AFIDs.
  - A record that decodes successfully but carries no AFIDs (e.g. a corrected entry) shows `N/A` in place of its AFIDs.
  - `--json` and `--csv` gain `status`, `message`, and `code` fields. Existing fields are unchanged.
  - `--afid --csv` now emits CSV instead of the human-readable table.

- **`amd-smi set --compute-partition` / `-C` now attempts each GPU individually.**  
  - An unsupported GPU reports `NOT_SUPPORTED` on its own, instead of the whole command aborting up front.
  - Input is validated against the static partition type names when profiles cannot be enumerated.
  - A numeric profile INDEX (e.g. `-C 0`) is also accepted, matched against an estimated `SPX, DPX, TPX, QPX, CPX` ordering.
  - The `-C` help now lists `SPX, DPX, TPX, QPX, CPX` and `0-4` instead of `N/A`.

- **`amdsmi_get_npm_info()` and `amdsmi_set_npm_limit()` now reject `amdsmi_node_handle` values not vended by `amdsmi_get_node_handle()`**.  
  - Previously any non-null handle was dereferenced directly; an unregistered/garbage handle now returns `AMDSMI_STATUS_INVAL` instead.

- **NPM sysfs numeric reads (e.g. `cur_node_power_limit`, `max_node_power_limit`) now reject negative or malformed content**.  
  - Previously a leading `-` (e.g. `"-1"`) parsed successfully as `UINT64_MAX`; such content now fails with `RSMI_STATUS_UNEXPECTED_DATA`.

- **`amd-smi metric` no longer prints fields the GPU's `gpu_metrics` table version cannot carry** (human-readable output only).  
  - These fields previously printed as `N/A` and are now omitted from human-readable output. On a metrics v1.3 GPU this removes 88 of the 110 `N/A` values in `amd-smi metric -g 0` (80%), taking the command from 140 lines to 95.
  - **Fields that stop appearing on current accelerators.** On the metrics v1.6 and later tables, which cover MI300X, MI325X and MI355X, the `POWER` section loses `THROTTLE_STATUS` and the `USAGE` section loses `MM_ACTIVITY`, `VCN_ACTIVITY` and `JPEG_ACTIVITY`. Naming the section does not bring them back, so `amd-smi metric --power` no longer prints `THROTTLE_STATUS` and `amd-smi metric --usage` no longer prints the three activity rows. On a metrics v1.3 GPU plain `amd-smi metric` instead drops the whole `THROTTLE` section, and `THROTTLE_STATUS` is still printed. **`--show-unsupported` restores all of them**, and `--json` and `--csv` never stopped emitting them.
  - **`--json` and `--csv` are unaffected and still emit every field, `N/A` ones included.** They are the machine-readable contract, so their key sets and column sets stay complete and stable: any key or column a script indexed before still resolves. Anything parsing the human-readable output does see fewer fields, and `--show-unsupported` gives it back the previous output.
  - **Pass `--show-unsupported` to restore the full human-readable output.** The flag is accepted with `--json` and `--csv` and has no effect there, since those formats are never filtered.
  - Scoped to the metrics table version, not to ASIC support: fields the CLI can also source from hwmon or sysfs are always printed, namely the `edge`, `hotspot` and `mem` temperature sensors, the fan section, the voltages, and the `gfx_N`, `vclk_N`, `dclk_N`, `mem_N` and `socclk_N` clock slots. Entries sourced only from the metrics blobs are eligible for omission: the `hbm_stacks`, `mid`, `aid` and `xcd` temperature arrays, and the `uclk_aid` and `socclks_mid` clock arrays.
  - A section named on the command line is never emptied. Plain `amd-smi metric` drops a section the version can populate nothing of, which on a metrics v1.3 GPU removes the whole `throttle` section; `amd-smi metric --throttle` prints all 31 rows instead. A named section that is only partly suppressed is still filtered. `--partition` scopes the data rather than naming a section, so `amd-smi metric --partition` protects nothing and filters like plain `amd-smi metric`.
  - A field the version does carry but the ASIC or driver leaves unpopulated still prints `N/A`, a field reporting a real value is never removed, and an unrecognized metrics version or an unreadable header suppresses nothing.

- **`amd-smi static` shows NUMA information on Linux guests**.  
  - `--numa` was previously limited to Linux baremetal and passthrough guests.
  - Default `amd-smi static` output on guests now includes NUMA, in human-readable, JSON, and CSV output.

##### Removed

- **Removed the internal `amd-smi` CLI exception classes `AmdSmiParameterNotSupportedException` and `AmdSmiUnknownErrorException`**.  
  These were CLI-internal (not part of the public `amdsmi` Python library), their exit-code behavior is superseded by the reworked CLI process exit codes described above.

##### Resolved Issues

- **Fixed `amd-smi` printing a Python traceback when an unknown NIC or switch is selected**.  
  - `amd-smi static --nic 999` and `--switch 999` failed while building the "device not found" error, so the command exited `1` with a traceback and no readable message. `--json` and `--csv` produced no parseable output.
  - Both now report `Can not find a device: NIC '999'` (or `SWITCH`) and exit `196`, matching `--gpu`, `--cpu`, and `--core`.

- **Fixed `amd-smi set`/`reset` on a GPU silently exiting `0` after a per-device failure.**  
  - A device error during GPU `set`/`reset` is now recorded, so the process exits with a non-zero code instead of reporting success.
  - Example: `set --memory-partition` on a GPU that does not support it.

- **Fixed the topology APIs reporting uninitialized NUMA data when a GPU has no CPU `io_link`**.  
  - The KFD node left its NUMA node number, weight and type uninitialized, so `amdsmi_topo_get_numa_node_number()`, `amdsmi_topo_get_link_weight()` and `amdsmi_topo_get_link_type()` could return an unrelated value alongside `AMDSMI_STATUS_SUCCESS`. They now return `AMDSMI_STATUS_NOT_SUPPORTED` for that case.
  - A debug build previously aborted on an assertion in the same path instead of reporting a status.

- **Fixed `amdsmi_get_gpu_fabric_info()` failing when only some UALink sysfs files carry data**.  
  - The call required every file in scope to have usable content and otherwise returned `AMDSMI_STATUS_UNEXPECTED_DATA`, so a legitimately empty `vpod_active_accels` or `local_accels` suppressed the rest of the fabric data and left `amd-smi fabric` with nothing to print.
  - It now returns `AMDSMI_STATUS_SUCCESS` when at least one file has usable content, and `AMDSMI_STATUS_NO_DATA` only when none do.

- **Restored `amd-smi` bash autocompletion**.  
  - Pressing Tab completes `amd-smi` subcommands and their options again, whether completion is registered explicitly or through the global `argcomplete` hook.

- **Fixed runtime fatal CPERs reporting no AFIDs**.  
  - `amd-smi ras --cper` showed an empty `list afids` column for fatal records, `amd-smi ras --afid --cper-file` printed `-`, and `amdsmi_get_afids_from_cper()` returned no AFIDs. amdgpu writes fatal crashdump sections 32 bytes shorter than `sizeof(cper_sec_crashdump)`, and the section bounds check required the full struct, so every such section was skipped. The check now requires only the dump member the record type uses.

- **Fixed APU core and skin temperatures showing `0.0 °C` when the sensor is not measured**.  
  - On APUs such as Ryzen AI Max (Strix Halo), `amd-smi metric` and the library now report these readings as not available instead of a false 0 °C.

- **Fixed xGMI read and write data counters reading as unavailable on MI450**.  
  - On MI450 the GPU connects to the CPU over xGMI, and the driver reports that link's traffic as a single counter. It was dropped instead of being stored as the first link, so `xgmi_read_data_acc` and `xgmi_write_data_acc` from `amdsmi_get_gpu_metrics_info()`, and the link `read`/`write` from `amdsmi_get_link_metrics()`, read `UINT64_MAX` (`N/A`).

- **Fixed `amdsmi_get_processor_count_from_handles()` crashing on NULL output pointers**.  
  - It wrote through a NULL `nr_cpusockets`, `nr_cpucores` or `nr_gpus` and crashed. It now returns `AMDSMI_STATUS_INVAL`.

- **Fixed `amdsmi_get_gpu_cper_entries()` failing the page after a large CPER record**.  
  - A record an earlier call had already returned still had to fit the caller's buffers on the next call, so that call failed with `AMDSMI_STATUS_OUT_OF_RESOURCES`. Records already returned are now skipped before the space check.

- **Fixed `amd-smi static --clock` printing a Python traceback for an unknown clock name**.  
  - It now reports the invalid parameter and exits `195` (`INVALID_PARAMETER`).

- **Fixed `amd-smi static --numa --csv` dropping socket affinity**.  
  - When NUMA-node CPU affinity was unavailable, as on guests, the CSV row printed `N/A` for the socket columns that human-readable and JSON output showed. It also no longer fails when only the socket affinity is unavailable.

- **Fixed crashes when several threads start or stop AMD SMI at the same time**.  
  - Programs that initialize and shut down the library from more than one thread could crash, find no GPUs, or leave the library initialized after every thread had shut it down.

- **Fixed a one-byte overrun when reading the memory partition into a small buffer**.  
  - A buffer too small for the partition name had the byte after its end overwritten. The name is now truncated inside the buffer, and the call still reports that the buffer is too small.

- **`amd-smi set --memory-partition` no longer advises a driver reload when the partition is already current**.  
  - Requesting the current partition does not cancel a different partition set earlier, so a reload would apply that one. amd-smi now reports that nothing changed and notes that an earlier request still applies at the next reload.

- **Fixed `amdsmi_get_gpu_driver_info()` reporting `0` as the driver date**.  
  - Kernels that no longer use the DRM driver date report it as `0`. `driver_date` is now empty in the C API and `N/A` in Python.

- **Fixed `amd-smi node --tray` printing an empty section on hardware without UALoE**.  
  - The tray output was guarded on the retrieved dictionary being non-empty, so the `N/A` defaults next to it could never be reached. `NODE:` was emitted with no body, and `--json` returned `[{"node": {}}]`. All three output formats now report `MAX_ACC_PER_TRAY` and `TRAY_TYPE` as `N/A`.

- **Fixed `amdsmi_get_afids_from_cper()` raising `TypeError` when passed a list of CPER records**.  
  - The list branch called `isinstance()` with a subscripted generic, which Python rejects at runtime, so the documented list-of-dicts input always failed. `bytes` input was unaffected. Input that is neither `bytes` nor a list now raises `AmdSmiParameterException`.

- **Fixed a segmentation fault in the DRM example when run without root**.  
  - `amdsmi_get_gpu_accelerator_partition_profile_config()` returns `AMDSMI_STATUS_NO_PERM` without filling its output, and the example read `num_profiles` from that uninitialized structure, so the profile loop ran past the end of the `profiles` array on every GPU. The loop now runs only when the query succeeds.

- **Fixed `amd-smi` using the `amdsmi` Python library of another ROCm install**.  
  - When `ROCM_PATH` or `ROCM_HOME` named a different ROCm install, `amd-smi` loaded that install's library, so it either failed with an `AttributeError` or reported the other install's library and ROCm versions. `ROCM_PATH` and `ROCM_HOME` no longer redirect which `amdsmi` Python package `amd-smi` loads.

- **Fixed `amdsmi_set_clk_freq()` falsely returning `AMDSMI_STATUS_INVAL` on Navi48 (gfx1201)**.
  - The deep-sleep bitmask validation added previously rejected the entire request when any requested bit fell outside a settable range computed from a prior read, so a supported clock set failed on gfx1201 even though the write is valid. The set is now lenient: out-of-range bits are dropped, and `AMDSMI_STATUS_INVAL` is only returned when no requested level is settable. The read-only check runs after the write, and `TestFrequenciesReadWrite` is re-enabled on Aldebaran (gfx90a / MI210).

#### **Composable Kernel** (1.3.0)

##### Added

* Added a CI path-length check that rejects newly added or renamed files whose repository-relative path exceeds 200 characters, keeping the absolute path under the Windows `MAX_PATH` limit of 260.
* Added bf16, fp32, fp8, and bf8 data types and the rrr, crr, and ccr layouts to the CK Tile Tile Engine batched GEMM and its dispatcher bridge.

##### Optimized

* Improved FMHA backward performance on gfx1250 by moving the operand transfers to TDM and the transposed reads to ds_load_tr, keeping the dK and dV accumulators in registers, pairing mirrored tiles under a causal mask, and coalescing the dQ atomic onto a single cache line.

#### **HIP** (7.17.0)

##### Added
* New HIP APIs
    * Module Management: support for API parity with corresponding CUDA API.
      * `hipModuleEnumerateFunctions` returns the function handles defined in a loaded module.
    * Library Management: support for API parity with corresponding CUDA API.
      * `hipLibraryGetModule` returns the module handle associated with a library.
* Disable HRR capture feature
* Support for recovering the allocation properties of an imported virtual memory handle. `hipMemGetAllocationPropertiesFromHandle()` now reports `hipMemLocationTypeHost` for a host-backed allocation obtained from `hipMemImportFromShareableHandle()`, instead of always reporting device memory. For a device-backed allocation it reports the owning device rather than whichever device was current when the handle was imported.

##### Resolved issues

* Fixed a regression where HIP applications on MI300A and other APUs failed at the first stream creation, `hipMemcpy`, `hipMemset`, or kernel launch with out-of-memory or invalid-argument errors after device-memory AQL rings were enabled by default (`DEBUG_CLR_AQL_DEV_QUEUE=1`). Large BAR now requires a real CPU link and a discrete agent, so APUs keep system-memory rings. WSL/Windows still uses CPU access to the GPU pool for discrete GPUs, because that path does not report link hops the way KFD does. Workaround before the fix: `DEBUG_CLR_AQL_DEV_QUEUE=0`.
* A registered `__device__` global that is absent from the loaded code object no longer aborts the process. Symbol lookup now returns `hipErrorInvalidSymbol` from the runtime's variable materialization path (`hipGetSymbolAddress`, `hipLibraryGetGlobal`, and related entry points). `hipModuleGetGlobal` still reports `hipErrorNotFound` for a missing name.
* Fixed `__hip_bfloat162` comparisons that ignored or misread the high lane. `__hbneu2` now returns true only when both lanes are unordered-not-equal, `__hgt2` and `__hisnan2` now return the per-lane result in `.y` instead of always 1.0, and the `<`, `<=`, `>`, `>=` operators now compare `.y` with `.y`. Code that relied on the previous results may see different values.

##### Changed

* Stream priority is now disabled to avoid known queue-priority-related issues. Priority streams are currently not supported.

#### **hipBLAS** (3.8.0)

##### Added

* gfx1170 and gfx1171 support to clients.

##### Changed

* The minimum required CMake version is now 3.17, for the library and the clients. `FindCUDAToolkit`, used by the CUDA backend, was added in that release.
* `hipblasSetWorkspace` chooses whether the rocBLAS backend may grow its device workspace. A non-null workspace with a nonzero size is a fixed limit and is never resized, which caps device memory for the handle. A later call that needs more, such as `trsmBatched` or `trsmStridedBatched` with a large `batchCount`, returns `HIPBLAS_STATUS_ALLOC_FAILED` even when free device memory remains. Passing `nullptr` or a size of 0 returns the handle to rocBLAS-managed memory, which starts from a default size and grows as needed. `HIPBLAS_WORKSPACE_CONFIG=:0:0` selects that same managed mode for callers that configure the workspace through the environment, such as PyTorch. With the rocBLAS backend, set `ROCBLAS_VERBOSE_ERROR=1` to print the required and available sizes when a fixed workspace is too small for `trsm`.

#### **hipCUB** (5.0.0)

##### Added

* Feature parity with CCCL/CUB 3.0.0.
* Added `::hip::std` support.
* Added support for large num_items `DeviceMerge` and `DeviceSegmentedSort`.

##### Changed

* Changed `CCCL_MINIMUM_VERSION` to `3.0.0` to align with CUB.
* Replaced `#pragma unroll` by `_CCCL_PRAGMA_UNROLL_FULL()` and `#pragma nounroll` by `_CCCL_PRAGMA_NOUNROLL()`.
* Updated `WarpExchange` template parameters for CUB compatibility. The third argument, `ARCH`, has been removed.
* The block merge sort and thread sort algorithms now perform optional loop unrolling via `_CCCL_SORT_MAYBE_UNROLL()`.

##### Removed
* The hipCUB migration to cccl3 has resulted in the following removals and deprecations:
  * Removed `hipcub::BaseTraits::CATEGORY`, `hipcub::BaseTraits::nullptr_TYPE` and `hipcub::BaseTraits::PRIMITIVE`.
  * Removed  `ConstantInputIterator`, `CountingInputIterator`, `DiscardOutputIterator` and `TransformInputIterator` which were deprecated in hipCUB-4.1.0.
  * Removed `DeviceSpmv`, which was removed from CUB after CCCL's 2.8.0 release. Use `hipSPARSE` or `rocSPARSE` libraries instead.
  * Removed `GridBarrier`.
  * Removed `HIPCUB_MIN`, `HIPCUB_MAX`, `HIPCUB_QUOTIENT_FLOOR`, `HIPCUB_QUOTIENT_CEILING`, `HIPCUB_ROUND_UP_NEAREST` and `HIPCUB_ROUND_DOWN_NEAREST` which were deprecated in hipCUB-4.1.0.
  * Removed `LEGACY_PTX_ARCH`.
  * Removed `hipcub:max` and `hipcub:min`, which were deprecated. Use `hip::std::max` and `hip::std::min` instead.
  * Deprecated `hipcub::Swap`, use `rocprim::swap` instead.
  * Deprecated `HIPCUB_IS_INT128_ENABLED`, use `_CCCL_HAS_INT128()` instead.
  * Deprecated `hipcub::Equality`, `hipcub::Inequality`, `hipcub::InequalityWrapper`, `hipcub::Sum`, `hipcub::Difference`, `hipcub::Division`, `hipcub::Max` and `hipcub::Min` operators. Use `hip::std::equal_to`, `hip::std::not_equal_to`, `hip::std::plus`, `hip::std::minus`, `hip::std::divides`, `hip::maximum` and `hip:minimum` operators instead.
* hipCUB compatibility with PyTorch v2.9 and v2.10 has been removed in this release.  Use PyTorch v2.11 or later.

#### **hipFFT** (1.0.27)

##### Changed

* Minor internal changes.

#### **hipFile** (0.6.0)

##### Added
* `ais-check` now reports whether an LVM logical volume supports fastpath.

##### Changed

* The hipFile CMake options have been renamed from `AIS_*` to `HIPFILE_*` (e.g., `AIS_CXX_STANDARD` is now `HIPFILE_CXX_STANDARD`). The old `AIS_*` names still work but are deprecated and emit a CMake deprecation warning.
* The `HIPFILE_WARN_UNSAFE_BUFFER_OPS` CMake option now defaults to `OFF` .

##### Resolved issues

* The `HIPFILE_WARN_UNSAFE_BUFFER_OPS` CMake option now applies the clang `-Wunsafe-buffer-usage` and `-fsafe-buffer-usage-suggestions` flags to C++ sources. Previously, the flags were never passed to the compiler.
* hipFile now returns `hipFileGetNewFDFailed` instead of `hipFileInternalError` when the process or system runs out of file descriptors. API calls that need a new file descriptor, such as `hipFileHandleRegister()`, can return this error.

#### **hipSOLVER** (3.8.0)

##### Added

* Added functions:
  * gelsBatched
    * hipsolverXXgelsBatched_bufferSize
    * hipsolverXXgelsBatched
  * geqrfBatched
    * hipsolverSgeqrfBatched_bufferSize, hipsolverDgeqrfBatched_bufferSize, hipsolverCgeqrfBatched_bufferSize, hipsolverZgeqrfBatched_bufferSize
    * hipsolverSgeqrfBatched, hipsolverDgeqrfBatched, hipsolverCgeqrfBatched, hipsolverZgeqrfBatched
  * getrfBatched
    * hipsolverSgetrfBatched_bufferSize, hipsolverDgetrfBatched_bufferSize, hipsolverCgetrfBatched_bufferSize, and hipsolverZgetrfBatched_bufferSize
    * hipsolverSgetrfBatched, hipsolverDgetrfBatched, hipsolverCgetrfBatched, and hipsolverZgetrfBatched
  * getriBatched
    * hipsolverXgetriBatched_bufferSize
    * hipsolverXgetriBatched
  * getrsBatched
    * hipsolverSgetrsBatched_bufferSize, hipsolverDgetrsBatched_bufferSize, hipsolverCgetrsBatched_bufferSize, hipsolverZgetrsBatched_bufferSize
    * hipsolverSgetrsBatched, hipsolverDgetrsBatched, hipsolverCgetrsBatched, hipsolverZgetrsBatched
* Added compatibility-only functions:
  * trtri
    * hipsolverDnXtrtri_bufferSize
    * hipsolverDnXtrtri

##### Changed

* The rocSOLVER backend now allows devIpiv to be passed as a null pointer to hipsolverXgetrs and hipsolverDnXgetrs.

##### Resolved issues

* Fixed hipsolverDnXpotrs calling 32-bit potrs instead of 64-bit potrs.
* Fixed hipsolverSpSetStream not setting the internal rocsparse_handle stream.

#### **MIOpen** (3.6.2)

##### Resolved Issues
* [Reduce] Fixed `miopenReduceTensor` silently truncating tensor lengths, strides, element counts
  and byte spans above INT32_MAX, and single reductions of about 2^30 or more fp16, bf16 or int8
  elements, which produced incorrect results. Such tensors are now rejected with
  `miopenStatusBadParm`.
* [Conv] Fixed a `ConvHipConv` crash on gfx1250 for grouped convolutions with more than
  65535 group-batch tiles, and an LDS race in its gfx1250 direct kernel
  (hipconv v0.3.2).
* [Conv] Fixed `ConvHipConv` losing its tuned config on a perf-db reload, which could fail
  execution with "not enough workspace for direct kernel" after an exhaustive Find. Records
  now carry the hipconv minor version, and records from another version are discarded.
* [Conv] Fixed incorrect backward-weights results from the `ConvHipConv` gfx950 direct kernel
  for configurations with `waves_q == 1`, where the LDS staging tile was reused across filter
  positions without cross-lane ordering.
* [Conv] Fixed up to 8x slower `ConvHipConv` `direct`, `direct_l1` and `direct_wgrad` kernels
  under ROCm 10.1's full AMDGCN link-time optimization default by building the hipconv device
  targets with `-fno-offload-lto`.
* [Conv] Fixed a multi-node data-parallel training throughput regression on gfx942 by excluding
  the backward-pass GEMM solvers for single-group 1x1 stride-1 pad-0 NHWC convolutions with a
  batch size below 4.
* [Conv] Fixed invalid Composable Kernel (CK) perf-config entries in the gfx1201 and gfx1100
  system databases that MIOpen could not resolve, which caused a fallback to untuned defaults.
* [BatchNorm] Fixed just-in-time compilation failures for the batch normalization kernels on
  non-GFX9 targets by restricting the Welford reduction's DPP inline assembly to GFX9 instead
  of a per-architecture denylist.
* Fixed an out-of-bounds write in `miopenLoadSolution` when a serialized solution carries more
  than three kernel work dimensions; such solutions are now rejected.
* Fixed a `uint64_t` typedef conflict between `miopen_cstdint.hpp` and the system `<stdint.h>`
  that broke runtime kernel compilation on LP64 Linux.
* [Conv] Fixed `TransposedConvWinoRageRxS<2,3>` being unreachable: it was registered in the solver
  registry and explicitly instantiated but missing from both Find solver lists, so it could never be
  enumerated or selected for NHWC convolutions.


##### Added
* [Conv] Added gfx950 depthwise backward-weights (fp16/bf16) and gfx1250 depthwise
  kernels to the `ConvHipConv` solver (hipconv v0.3.1).
* [Conv] Added gfx950 patch-embedding kernels (fp16/bf16), gfx950 TF32 kernels for dense
  and depthwise forward/backward-data and dense backward-weights, and gfx1250 depthwise
  backward-weights kernels (fp16/bf16) to the `ConvHipConv` solver (hipconv v0.3.2).
* [Conv] Added a gfx1250 dense backward-weights kernel (fp16/bf16) to the `ConvHipConv`
  solver (hipconv v0.4.0).
* [Conv] Added NHWC (channel-last) support to the GEMM convolution solvers, together with the
  im2col and col2im kernels the layout requires.
* [Conv] Added cross-architecture AI heuristics for immediate mode (`MIOPEN_FIND_MODE=2`): a
  LightGBM ranker selects the solver and a per-solver model selects its performance config,
  extending data-driven selection to the Navi architectures.
* [Conv] Added the missing 3x3 stride-1 pad-1 tile-28 CK depthwise backward-data instance.

##### Changed
* [Reduce] `miopenGetReductionWorkspaceSize` and `miopenGetReductionIndicesSize` now reject tensors
  of rank greater than 6, matching `miopenReduceTensor`.
* [Reduce] Invalid arguments to the reduction API (mismatched descriptors, unsupported types,
  undersized workspace or indices buffers) now return `miopenStatusBadParm` instead of
  `miopenStatusUnknownError`.
* [Conv] `ConvHipConv` is held out of the AI performance-config picker and uses its own default
  config until the model is retrained against the current hipconv config vocabulary.
* [Conv] Large forward and backward-data convolutions are no longer gated off CK by tensor byte
  span. The 32-bit byte-offset overflow that motivated the gate is fixed in CK itself, so these
  problems can again use the faster int32 CK instances.
* [Conv] Relaxed the ASM-GTC NHWC large-tensor guard for forward and backward-data. Those kernels
  address a full 4 GiB window per dispatch slice and advance the base pointer per batch-split, so
  the input and output bounds were redundant; what remains is a bound on the per-group weight size.
  Backward-weights keeps the stricter int32 bound, because 292 of its 900 kernels ignore the
  batch-split dimension (#11805). This restores 8,978 forward and backward-data entries across the
  six system find-databases that the guard's first revision removed.

##### Optimized
* [Conv] Optimized the `ConvDirectNaiveConv` solver with per-direction kernel files, float
  accumulators for fp32/fp16/bf16 inputs, spatial tiling and a bfloat16 path, for 2-35x
  speedups on representative model shapes.
* Optimized the kernel cache by reusing cached database connections instead of opening and
  closing SQLite on every binary load or store.

##### Removed
* [Conv] Removed gfx803 convolution solver `ConvBinWinogradRxSFused` and its kernel sources.
* [Conv] Removed the gfx803 code paths and kernel sources from `ConvBinWinograd3x3U` and `ConvBinWinogradRxS`.
* [Conv] Removed the gfx803 find-database files.

#### **RCCL** (2.32.3)

##### Added
* Compatibility with NCCL 2.32.3.
* `ncclSetEncryption()` and `ncclEncryptionConfig_t` (initialize with `NCCL_ENCRYPTION_CONFIG_INITIALIZER`) to encrypt NCCL-owned TCP socket traffic with TLS using a pre-shared key of at least 32 bytes. Requires a build configured with `-DTLS_BACKEND=OPENSSL3`; `libssl.so.3` is loaded at run time. Encryption is off by default and covers only traffic sent through RCCL's TCP sockets, not RDMA or GPU-to-GPU transfers.
* `launchCompletionEvent` in the per-collective config (`ncclCollConfig_t`, now `ncclCollConfig_v23200`): a caller-owned event that RCCL records for the collective's kernel launch. Every rank must pass an event, or every rank must pass `NULL`. HIP has no launch-completion launch attribute, so on ROCm the event is recorded on the stream immediately before the kernel launch.
* `NCCL_WIN_GIN_ONLY` window registration flag, which registers a window for GIN access only.
* GIN device-API C entry points for the timeout waits (`ncclGinWaitTimeout`, `ncclGinWaitSignalTimeout`, `ncclGinWaitSignalTimeoutVA`, `ncclGinWaitCounterTimeout`, `ncclGinFlushTimeout`), which return `ncclTimeout` after `timeoutCycles` `clock64()` cycles, and for `ncclGinFlushAsync`, `ncclGinWait` and the VA-signal operations (`ncclGinReadSignalVA`, `ncclGinResetSignalVA`, `ncclGinWaitSignalVA`). The equivalent C++ `ncclGin` members were already available.
* ReduceSum and ReduceCopy device APIs (`ncclLsaReduceSum`, `ncclLsaCopy`, `ncclLsaReduceSumCopy`, `ncclLocalReduceSumCopy`) in the LLVM bitcode library (`EMIT_LLVM_IR=ON`, `librccl_device.bc`) for 8-, 32- and 64-bit integer, FP16, BF16, FP32 and FP64 types.
* `NCCL_DEBUG_LEVELS` and the `ATTN` log level. `ATTN` reports non-fatal conditions that need attention, such as configuration fallbacks and plugin initialization failures. `NCCL_DEBUG_LEVELS` adds individual levels (`VERSION`, `WARN`, `ATTN`, `INFO`, `ABORT`, `TRACE`) to the selection made by `NCCL_DEBUG`.
* Device progress counters (`NCCL_PROGRESS_COUNTERS`, default `0`; also requires `NCCL_RAS_ENABLE=1`, the default) mirrored to host memory, and a RAS progress monitor that warns when a communicator's counters stop advancing (`NCCL_PROGRESS_COUNTER_MONITOR_POLL_MS`, `NCCL_PROGRESS_COUNTER_MONITOR_STALE_MS`, `NCCL_PROGRESS_COUNTER_MONITOR_STALE_WARN_SEC`).
* New RAS diagnostics checks (`NCCL_RUN_RAS_DIAGNOSTICS`) for PCI topology (`rdma_topo check`), the IOMMU mode and groups and the ATS state of GPU/NIC pairs, and the path types each communicator uses.
* An IB write-bandwidth check in the communicator init diagnostics (`NCCL_RUN_DIAGNOSTICS`). It runs perftest's `ib_write_bw` between hosts of multi-node communicators, with a per-run timeout set by `NCCL_DIAGNOSTICS_IB_BW_TIMEOUT` (default 5 seconds). It measures from GPU memory only when the installed `ib_write_bw` supports `--use_cuda`; otherwise it measures from host memory.
* Communicator initialization reports, at the `ATTN` level, ranks that run RCCL builds from different git revisions.
* Inspector plugin ring-buffer drop counters: a single warning per process when events are dropped, a `dump_stats` record with operation and drop counts in JSON output, and `NCCL_INSPECTOR_PROM_DUMP_STATS` (default `0`) to add the same counters to Prometheus output.
* `RCCL_BCM_LINKS_PATH`: fallback directory path for Broadcom PEX890xx multi-host switch inter-switch-link topology. On systems where a single physical PEX switch is partitioned across multiple CPU domains (upstream ports), RCCL needs to know which ports share the same switch so it can classify cross-partition PCIe paths as `PXB` instead of `PHB` and enable GDR across them. The `switch_discovery` kernel module exposes this via sysfs (`/sys/kernel/pci_switch_link/virtual_switch_links`); when that path is not available, RCCL reads the same directory layout from `RCCL_BCM_LINKS_PATH`. The companion userspace tool `rccl_bcm_isl_discover.sh` discovers the switch groups by reading the PCIe Device Serial Number (DSN) from extended config space (vendor `1000`, device `c030`) and populates this directory.
* `NCCL_CE_CHUNK_SIZE` (default 8 MiB): chunk size for the round-robin chunking that host RMA Copy Engine batches use when per-peer transfer sizes differ.
* `NCCL_CE_INTRA_GPU_MEMCPY_ENABLE` (default `1`): controls whether batched Copy Engine collective copies pass `hipMemcpyFlagPreferOverlapWithCompute`. RCCL previously always set this flag, so the default keeps the previous behavior; set `0` to omit it.
* `NCCL_GIN_MLOPART`: allows GIN on MLO/CPX partitions. A partition reaches xGMI and the network over the physical device's paths, so GIN is as available to a partition as to the whole GPU. By default a single-node partitioned communicator keeps GIN, and one spanning several nodes does not, because past one node admitting a partition also reaches the hierarchical Copy Engine path. Set `1` to allow GIN on partitions on any number of nodes, or `0` to restore the previous partition exclusion.
* `NCCL_HIER_CE_COLL_AG_RAIL_RING_ENABLE` (default `-1`): a positive value selects a ring for the inter-node rail phase of hierarchical Copy Engine `ncclAllGather`. The default keeps the direct path.
* `NCCL_TREE_COMPACT` (default `0`): builds the inter-node trees with a half-interleave node remapping, for jobs whose nodes are ordered by network domain. Both trees keep cross-domain hops at the leaves only when the nodes split evenly into two domains; with an uneven split, the first tree does so when the first domain holds at least half of the nodes, and the second tree still crosses domains. All ranks in a communicator must use the same value; initialization fails with `ncclInvalidUsage` otherwise.
* `NCCL_IB_SORT_MERGE_NICS`: sorts the sub-devices of a merged IB device by plane ID. RCCL defaults it to `0` (NCCL defaults to `1`), so merged-device order and names are unchanged.
* `RCCL_FABRIC_TELEMETRY_ENABLE` (default `0`): samples every amd-smi fabric telemetry category for the GPUs this process opens communicators on and logs the counters that moved, plus a closing report over the whole sampled span when a device's last communicator is destroyed. Diagnostic only, and off by default. Also needs `RCCL_USE_AMD_SMI_LIB=1` and `render`-group access to the IFoE config character device; without either, RCCL logs why telemetry was skipped and initialization continues.
* `RCCL_FABRIC_TELEMETRY_INTERVAL_MS` (default `1000`): sampling period for `RCCL_FABRIC_TELEMETRY_ENABLE`, clamped to a 100 ms floor and a one-hour ceiling. A non-positive value disables telemetry entirely.
* Hierarchical symmetric `ncclAllGather` for multi-node using GIN and LSA (`AllGather_RailRing_LsaST`): a GIN ring over the rails team moves data between nodes and LSA stores broadcast it to the other GPUs in each node.
* nccl4py: per-call collective configuration (`NCCLCollConfig`, `VendorOption`) including the launch completion event, communicator properties (`NCCLCommProperties`), and the `GIN_ONLY` window flag. On ROCm, the HIP `Event` shim provides only an event handle; `record()`, `sync()` and `query` are not implemented.
* `RCCL_TUNER_EMBEDDED_CONFIG` (default `1`): the built-in CSV tuner configurations in `tuner/*.csv` are now compiled into `librccl.so` and used as the stock tuning defaults, so a cluster no longer has to stage the CSV on every node or on NFS. Set `0` to ignore the embedded configurations. `NCCL_TUNER_CONFIG_FILE` and the on-disk tuner directories still take precedence, and `NCCL_TUNER_PLUGIN=none` still disables the tuner entirely.
* `RCCL_TUNING_PARTITION_MODE` (`auto`, `SPX` or `DPX`; default `auto`): selects the gfx1250 tuning table for the GPU's compute-partition mode. `auto` uses the DPX table when the GPU reports DPX, and the SPX table otherwise. A communicator uses the DPX table only when every rank selects it. The DPX table is currently a copy of the SPX table, so tuning is unchanged.
* `RCCL_CE_AR_2SHOT_MAX_BYTES` (default `-1`): overrides the 2-shot AllReduce size cap from the arch table. Set to a positive value to override `ceNonRegMax[AR]`. Renamed from `RCCL_CE_AR_MAX_MSG_BYTES`.
* `RCCL_CE_AR_2SHOT_MIN_BYTES` (default `-1`): overrides the minimum message size for the CE 2-shot AllReduce path. When `-1`, resolved from `ceNonRegMin[AllReduce]` in the per-arch table.
* `RCCL_CE_AR_REG_MAX_MSG_BYTES` (default `-1`): overrides the registered CE AllReduce size cap from the arch table.
* `RCCL_CE_AR_STAGING_BYTES` (default `-1`): overrides the CE AllReduce staging buffer allocation size; when unset, `NCCL_CE_AR_STAGING_BYTES` is used. CE ReduceScatter shares this buffer.
* `RCCL_CE_REDUCESCATTER` (default `0`): opt-in Copy Engine ReduceScatter on a single-node symmetric communicator with `CTA_POLICY_ZERO`. Always stages through `ceARTmpBuf`; user-buffer registration is not required. No message-size cap (large messages are pipelined).
* `RCCL_CE_COLL_MAX_BYTES` (default `-1`): overrides the CE registered-window and 2-shot size cap for all non-AllReduce collectives (AllGather, AlltoAll, etc.). When `-1`, resolved from `ceRegMax[func]` or `ceNonRegMax[func]` in the per-arch table.
* `RCCL_CE_COLL_MIN_BYTES` (default `-1`): overrides the minimum message size for CE non-AllReduce paths (both registered-window and scratch). When `-1`, resolved from `ceNonRegMin[func]` in the per-arch table.
* `RCCL_SYM_K_MAX_BYTES` (default `-1`): overrides the symmetric-kernel upper bound for all collectives. When `-1`, resolved from `symMaxR2[func]` (eager) or `symMaxR2Graph[func]` (graph-capture) in the per-arch table. `kThreshUnlimited` (SIZE_MAX) in the table means no suppression.
* `RCCL_SYM_K_MIN_BYTES` (default `-1`): overrides the symmetric-kernel lower bound for all collectives. When `-1`, resolved from `symMinR2[func]` in the per-arch table. Below this size, DDA is preferred over symk.
* Per-architecture dispatch table (`rcclArchThresholds`) centralizing algo/proto selection for DDA protocol, CE, and symmetric kernel per collective, replacing scattered hardcoded defaults. Tables are defined for gfx1250, gfx950, and gfx942; the lookup `rcclGetArchThresholds()` maps GCN arch strings to the appropriate entry. The `symMinR2[func]` field sets a per-collective lower bound for registered buffers.
* `RCCL_IGNORE_ARCH_TABLE` (default `0`): when set to `1`, bypasses the `rcclArchThresholds` dispatch table and falls back to compile-time constants for all DDA, CE, and symmetric-kernel thresholds. Has no effect on gfx942 and gfx950, which always use the arch table.
* New `rcclAddonAlgos_t` values: `RCCL_CE_SCRATCH` (CE via DDA scratch, distinct from `RCCL_CE_REGISTERED`), `RCCL_A2A_PIVOT`, `RCCL_A2A_GDA`, `RCCL_A2A_GIN_SDMA`, and `RCCL_DIRECT_ALLTOALL`, with corresponding `rcclGetAlgoName()` labels.
* rccl-tests: AlltoAll now reports algo/protocol in VERSION output, consistent with other collectives.
* `net_ib_cast` refreshes local GIDs after `IBV_EVENT_GID_CHANGE` and applies them to data, flush, probing and port recovery QPs during IB port recovery, matching `net_ib` in NCCL 2.31.2.
* Per-collective configuration APIs (`ncclCollConfig_t` / `nccl*Config()` entry points) and the `ncclConfigExt_t` vendor extension list. Initialize configs with `NCCL_COLLCONFIG_INITIALIZER`.
* Communicator config (`ncclConfig_v23100`) fields for implicit launch ordering (`launchOrderImplicit`), RMA signal count (`numRmaSig`), eager RMA init (`rmaEagerInit`), and host collective fault tolerance (`hostCftMode`).
* NCCL profiler plugin API v7, with per-call user profiler tags and symmetric-kernel phase events.
* RAS diagnostics (`NCCL_RUN_RAS_DIAGNOSTICS`) covering GPU inventory, ROCm runtime versions, ECC counters, XGMI link state, and `NCCL_*` environment consistency (AMDSMI in place of NVML).
* Communicator init diagnostics (`NCCL_RUN_DIAGNOSTICS`) with an active P2P connectivity check.
* Multiple GIN proxy progress threads via `NCCL_GIN_PROXY_NTHREADS`.
* Enabled hierarchical Copy Engine `ncclAllGather`, `ncclAlltoAll`, and `ncclAlltoAllv` on multi-node communicators: the intra-node phase runs over SDMA/CE across the LSA team and the inter-node phase over one-sided RMA through the CPU (GIN) proxy. The path is off by default and takes two opt-ins: `NCCL_CTA_POLICY=2` (equivalently `config.CTAPolicy = NCCL_CTA_POLICY_ZERO`), and symmetric registration of both the send and the receive buffer through `ncclCommWindowRegister`. On architectures other than gfx1250 (MI450) a third is needed, `NCCL_CUMEM_ENABLE=1`, because symmetric memory depends on cuMem and cuMem auto-detection enables itself only on gfx1250. `NCCL_GIN_ENABLE`, `NCCL_NUM_RMA_CTX`, `NCCL_WIN_ENABLE` and `NCCL_DMABUF_ENABLE` need no change, since their defaults already qualify; the inter-node rail draws from the internal RMA context pool (`NCCL_NUM_RMA_INT_CTX`, default 4) rather than the user-addressable contexts, so `numRmaCtx` only has to stay above zero. Selection also requires an available GIN backend and the same number of ranks on every node; communicators that do not qualify keep using the kernel path. `ncclAlltoAllv` walks the gathered per-peer size matrix, skips zero-length peers, and chunks each inter-node put; single-node AlltoAllv stays on the existing CE path. Validated for correctness and path selection from 1 to 32 nodes.
* Added `ncclGinFenceLevel` barrier semantics on the Anvil SDMA GIN backend (`NCCL_GIN_TYPE=7`). `gin.get`, `flushAsync` and `wait` are implemented instead of trapping, and a barrier's signal is now ordered behind the payload on the peer's SDMA queue, so `Put`, `Get` and the default `Put | Get` fences drain what they promise. `Put` covers a rank's puts to itself, and `ncclGinAllContexts` fences every GIN context. Single-node validated on gfx950.
* Added gfx1250 (MI450) Tensor Data Mover staging to the symmetric AllGather, AllReduce and ReduceScatter kernels, where NCCL uses NVIDIA's TMA. Their deep loops move each tile through the warp's shared-memory scratch with the DMA engine instead of per-lane vector loads; on ROCm that path previously compiled out entirely, so the `Tma` kernels were never generated. Launch tuning was also corrected so these kernels can use up to a quarter of a GPU's WGPs, where only 14 were reached before, which is where most of the large-message gain comes from. Reached only when the application opts into symmetric memory; `NCCL_SYM_TMA_ENABLE` now defaults to `1` (see Changed in RCCL 2.32.3). `NCCL_SYM_TMA_ENABLE=2` additionally forces these kernels wherever the collective has one, skipping both the tuner's cost comparison and the deep-loop size bar, which is intended for A/B measurement rather than production. The new kernels are generated only when gfx1250 is in `GPU_TARGETS`, and every other architecture keeps the vector path unchanged.
* Removed GIN rocSHMEM GDA dependency on rocSHMEM GDA bitcode: GIN rocSHMEM GDA is now header-only. The CMake `roc::rccl` target provides the required include directories in its `INSTALL_INTERFACE` property.
* `RCCL_IB_GDR_FLUSH_FLAG` (default `0`): replaces the GPUDirect RDMA flush of Simple-protocol receives in the `IB` network transport. Instead of the proxy fencing each receive with an RDMA read and waiting for it, the sender writes a per-QP flag into GPU memory after the payload without relaxed ordering, and the GPU waits for it. A communicator uses it only when every rank enables it and can use it (gfx942 or gfx950, the `IB` transport, and no adaptive routing, out-of-order receive queues, ID-based receive matching (`NCCL_IB_RECEIVER_SIDE_MATCHING_SCHEME=1`) or port-failover resiliency (`NCCL_IB_RESILIENCY_PORT_FAILOVER`)); otherwise every rank keeps the RDMA read flush. Connections over a fused NIC (NIC Fusion, including a fused vNIC chosen by subnet-aware routing), with more than 8 QPs per request, or that cannot register the flag memory also keep it. On gfx950 with 2 to 8 nodes of 8 GPUs, such a communicator also selects Ring/LL128 from 16 MB per node, Tree/Simple from 32 MB per node and Ring/Simple from 64 MB per node for `ncclAllReduce`, and switches `ncclReduceScatter` from LL128 to Simple above 6 MB per rank instead of 8 MB, unless `NCCL_ALGO` or `NCCL_PROTO` is set. On MI355X it raises large `ncclAllReduce` and `ncclReduceScatter` bandwidth by 10 to 22% and 1 to 8 MB `ncclAlltoAll` by 4 to 12%.
* `RCCL_IB_GDR_FLUSH_FLAG_TUNE` (default `0`): additional tuning for communicators that use `RCCL_IB_GDR_FLUSH_FLAG` on gfx950 with 8 GPUs per node on 2 to 8 or 16 nodes. Ring and tree connections get a 4x LL128 buffer (send/recv keeps the default, and `NCCL_LL128_BUFFSIZE` disables it); `ncclAllReduce` selects Ring/LL128 from 12 MB per node up to what one ring loop carries, then Tree/Simple up to 64 MB per node and Ring/Simple on 48 channels up to 128 MB per node, and Tree/Simple from 8 to 64 MB per node on 16 nodes; `ncclReduceScatter` keeps LL128 up to 7 MB per rank, then Simple, on 16 nodes too. All ranks must use the same value.
* `RCCL_PCIE_ORDERED_RING` (default `1`, also accepted as `NCCL_PCIE_ORDERED_RING`): pins the ring graph search to the order GPUs appear in the topology array, so every channel uses the same ring. Without it the search picks a different permutation per channel, so a device exchanges with many different peers; pinning the order reduces fan-out to one peer per direction per device across all rings, which cuts the connections, FIFO buffers and setup work each device carries. On gfx1250 partitioned (DPX) GPUs the unpinned search also mixes scale-up and scale-in hops between channels; pinning keeps the two partitions of a die adjacent, though ring throughput is bounded by its slowest hop either way. This constrains GPU visitation order only: path validation, bandwidth constraints and backtracking are unchanged. It applies to gfx1250 ring graphs on a single host where every rank is a GPU of that topology, so trees, CollNet, other architectures and multi-host communicators are unaffected, and `NCCL_GRAPH_FILE` and `NCCL_RINGS` still take precedence. Set `0` to restore the unconstrained search, or `2` to fail communicator initialization instead of falling back when no ordered ring passes validation.
* Added `RCCL_TDM_LL128_ENABLE` (default `0`): an experimental gfx1250 (MI450) Tensor Data Mover path for the LL128 protocol's user destination buffer, staging each warp slice in LDS and pushing it out as one bulk transfer instead of per-lane vector stores. Excluded from the default build: it requires `--enable-tdm-prim-ll128` at build time and `RCCL_TDM_LL128_ENABLE=1` at runtime, and compiles only on gfx1250 with an SDK providing the tensor data mover builtins. The runtime flag is captured once in the primitive's constructor rather than read from `ncclShmem` per slice, so the disabled case costs one register test per slice. Only the destination leg is staged: an async source load through the same per-warp window corrupted one slice in roughly 2% of 2-node ReduceScatter runs at 4 GiB and measured no faster, so it was removed rather than fixed. The receive and send FIFO legs are not eligible either, because both interleave protocol flag words into the payload. Measured on gfx1250 under forced RING/LL128 (out-of-place bus bandwidth, median of 3, 4 and 8 ranks): AllGather gains 5-10% above 256 MiB, AllReduce 3-5%, ReduceScatter 1-2%, while messages below roughly 32 MiB lose 2-5% to the staging overhead. With the tuner left to choose algorithm and protocol the path is not selected and throughput is unchanged, because LL128 is not the winning protocol in the size range where the staging pays off. The accumulator load is deliberately kept off this path because it has no spin to overlap; the accumulator-reduced store still goes through it.
* Added host-side and device-side tests for the LL128 Tensor Data Mover store leg (`Ll128TdmStoreTest`): the build flag reaching the device pass, the slot indices tiling the staged window exactly once, and the staged store compared byte-for-byte against the scalar path it replaces. These require one GPU rather than the eight the existing LL128 suites need.
* Added a reduction path to the experimental gfx1250 (MI450) Tensor Data Mover support for the SIMPLE protocol. Slices that carry a reduction, or a pre-op or post-op that is not the identity, now move whole 8 KiB tiles of each source into the warp's LDS staging window with the TDM, reduce them there, and write the result to every destination with the TDM; the rest of each slice stays on the vector path. Only the SIMPLE primitives' main reduce-copy calls can reach it, every source and destination must be 256-byte aligned, and the bias, pipelined and multimem variants are excluded. Like the copy path it requires `--enable-tdm-simple` at build time, and it has its own runtime switch, `RCCL_TDM_SIMPLE_REDUCE_ENABLE=1`, independent of `RCCL_TDM_SIMPLE_ENABLE`. Double buffering is not implemented.

##### Changed
* `NCCL_MULTI_SEGMENT_REGISTER` default changed `1` -> `0` on HIP and stays `1` on other builds. On HIP, multi-segment registration stays off until a job sets the variable. Set `1` to register one DMA-BUF memory region per physical segment where the transport supports it.
* NET registration counts physical segments over the whole registered buffer, not only the range a collective touches. With `NCCL_MULTI_SEGMENT_REGISTER=0`, a collective on any part of a multi-segment registration skips NET registration and stages, instead of retrying a single-region registration on every collective.
* GIN window registration on HIP fails with `ncclInvalidUsage` when the window spans several device-memory segments, instead of registering a memory region that covers only the first segment, and when it contains host VMM memory and `NCCL_DMABUF_ENABLE=0`.
* `install.sh --all_unrolls` (`-DBUILD_ALL_UNROLLS=ON`) now adds the unroll factors the multi-arch default skips (8 and 16) on top of each architecture's existing set. It no longer replaces that set or clears the gfx1250 pin on unroll 32. The flag stays off by default, so a normal build does not compile unroll 8 or 16.
* Host-side device API declarations (`ncclDevCommCreate`, `ncclDevCommDestroy`, `ncclCommQueryProperties`, `ncclGetPeerDevicePointer`, the `*CreateRequirement` helpers, the `ncclDevCommRequirements` and `ncclCommProperties` structs and their initializers) moved to `nccl_device/host.h`. `nccl_device.h` still includes it; code that includes individual `nccl_device/*.h` headers directly must also include `nccl_device/host.h`.
* Profiler `ncclProfileKernelPhase` events are emitted only by symmetric kernels, matching NCCL 2.32. Regular and P2P kernels report only `ncclProfileKernelCh` start and stop.
* Updated the RMA plugin interface to v16, which adds a per-communicator `getRmaProperties` query. Plugins built against v13, v14 or v15 still load.
* `NCCL_DEBUG_TIMESTAMP_LEVELS` now timestamps `ATTN` messages as well as `WARN` messages by default.
* The internal IB RMA proxy chains work requests and rings the doorbell once per batch of up to `NCCL_RMA_IB_WR_BATCHSIZE` (default `64`) requests, which raises the small-message rate.
* Communicator initialization at large scale is faster because the proxy no longer scans inactive poll descriptors.
* Network devices for each GPU are now chosen by rail and plane assignment, replacing the previous start-device scattering.
* `NCCL_MLOPART_RDMA_ENABLE` (default `0`) is kept. NCCL 2.32 removed it and treats buffers on partitioned GPUs as RDMA-capable on all non-ARM hosts; RCCL keeps network buffer registration for partitioned (CPX/DPX) GPUs opt-in.
* Enabled the single-node one-slice optimization for gfx1250, which was previously limited to gfx942 and gfx950. Ring Simple AllGather, AllReduce and ReduceScatter now use one slice per chunk instead of two, halving the per-chunk peer handshakes. The gain is largest below about 128 MB and tapers to a few percent at multi-GB sizes, where the ring is already near the link bandwidth. LL, LL128 and non-ring algorithms are unaffected.
* The CSV tuner configurations are no longer installed to `${ROCM_PATH}/share/rccl/tuner/` or copied into the build tree; they ship inside `librccl.so`. A CSV placed in that directory, or in `<librccl.so dir>/tuner/`, still overrides the built-in defaults. A copy left behind by an earlier build or by `cmake --install` over an existing prefix also still takes precedence, so delete it to pick up the embedded defaults; package upgrades remove it automatically.
* Retuned the symmetric AllReduce, ReduceScatter and AllGather kernels for gfx950. The block width is now selected per collective and message size instead of a fixed 256 threads, and the work partitioning and unroll factors in the load-direct kernels and the AllGather store kernel were refitted to the 64-lane wavefront. AllGather also switches from LL to the store kernel from 1 MB instead of 16 MB. The multi-node GIN kernels and all other architectures are unchanged. To fit the wider LL blocks, the shared LL scratch buffer on gfx950 grows from 4 MiB to 8 MiB per 8-rank communicator.
* gfx1250 no longer forces unregistered `ncclAllGather`, `ncclAlltoAll`, `ncclGather` and `ncclScatter` through the Copy Engine scratch path (CE-Scratch), because it was slower than the kernel paths it replaced. Whether CE-Scratch is forced by default now comes from the per-architecture table: gfx942 and gfx950 still force it. A communicator-level `NCCL_CTA_POLICY_ZERO` still forces it on every architecture; an explicit `RCCL_FORCE_CE=1` still forces it and `RCCL_FORCE_CE=0` still turns it off. gfx1250 `ncclAllGather` now uses DDA LL128 up to 128 MiB.
* gfx1250 `ncclAllReduce` on single-node communicators keeps the LL128 protocol at larger sizes: Tree at every size, Ring up to nRanks x 32 MiB, and the DDA LL128 tier up to 256 MiB. The DDA LL128 two-shot ceiling now follows that tier ceiling (`RCCL_DDA_LL128_TWOSHOT_THRESHOLD` still overrides it); the DDA scratch slot lets two-shot reach 256 MiB from 32 ranks, about 240 MiB at 16 and 128 MiB at 2 to 8. On gfx1250 communicators of 2 to 8 ranks, the DDA fabric scratch buffer grows to about 546 MiB.
* On gfx1250 single-node communicators of 8 or more ranks, the DDA tier, algorithm and protocol crossovers now scale with the rank count. Communicators below 8 ranks keep the previous thresholds.
  * `ncclAllReduce`: DDA LL two-shot up to nRanks x 1 MiB (`RCCL_DDA_LL_TWOSHOT_THRESHOLD` still overrides it); from 32 ranks, DDA VMM between 2 MiB and nRanks x 512 KiB for unregistered buffers unless `RCCL_DDA_THRESHOLD` or `RCCL_DDA_LL_THRESHOLD` is set; Tree instead of Ring up to nRanks x 1 MiB unless `NCCL_ALGO` or `NCCL_PROTO` is set, with LL up to nRanks x 32 KiB.
  * `ncclAllGather`: DDA LL up to nRanks x 16 KiB, DDA LL128 up to 256 MiB from 16 ranks, and Ring LL128 up to 64 MiB per rank.
  * `ncclReduceScatter`: DDA LL up to nRanks x 256 KiB, DDA LL128 up to nRanks x 512 KiB, then DDA VMM up to nRanks x 2 MiB; Ring LL up to 512 KiB per rank and LL128 up to 8 MiB per rank (32 MiB for 1-byte types); Ring Simple on at most 128 channels between 8 and 32 MiB per rank unless `NCCL_MIN_NCHANNELS` or `NCCL_MAX_NCHANNELS` is set.
  * DDA VMM `ncclAllReduce` and `ncclReduceScatter` launch 2048 / nRanks blocks, clamped to 32 to 128 and never more than before.
* `fp8_e4m3` and `fp8_e5m2` sum reductions on gfx1250 convert eight elements per instruction using the packed conversion instructions, instead of two. Results are unchanged. Other architectures and reduction operators are unaffected.
* The rocSHMEM GDA GIN backend (`NCCL_GIN_TYPE=6`) now sets the RoCE traffic class of its queue pairs from `NCCL_GIN_IB_TC`, then `NCCL_IB_TC`, then the communicator traffic class, matching the IB proxy backend. It previously read `ROCSHMEM_GDA_TRAFFIC_CLASS`, which no longer affects these queue pairs. The default remains traffic class 0 when none of these is set.
* `NCCL_SYM_TMA_ENABLE` now defaults to `1`, matching NCCL 2.31.2, so on gfx1250 (MI450) the Tensor Data Mover symmetric kernels (`AllGather_TmaST`, `AllReduce_RSxTmaLD_AGxTmaST`, `ReduceScatter_TmaLD`) are offered to the symmetric cost model whenever an application uses symmetric registered memory. The cost model selects them only where their deep loop engages, and launches them at the widest allowed block count. They require 16-byte aligned buffers and a build with gfx1250 in `GPU_TARGETS` and a ROCm SDK that ships the TDM descriptor header. For in-place AllGather and ReduceScatter the per-rank slice must also be a multiple of 16 bytes, so that every rank selects the same kernel. Other architectures are unaffected. Set `NCCL_SYM_TMA_ENABLE=0` to restore the previous behavior.
* rccl-tests `-A 1` (and `rcclGetCollImplInfo` / `rcclSymKGetInfo`) now report a symmetric collective that selects a Tensor Data Mover kernel as `SymTDM` instead of `SYM`. The symmetric reporting query also accounts for buffer alignment, so it no longer omits the TDM kernels that the live dispatch would pick.
* The AMD Pensando 'ionic' RDMA provider for the rocSHMEM GDA GIN backend (`NCCL_GIN_TYPE=6`) now properly enforces the required support for creation of 'collapsed' single-entry completion queues. If network driver and firmware support for this feature is unavailable, an initialization failure will occur rather than a silent fallback to plain completion queues.
* Vendored `{fmt}` 10.2.1 as a header-only tree under `external/fmt`. RCCL, its tests, and the `topo_expl` tool now always build against this copy instead of looking for a system `fmt` package or downloading one through `FetchContent` or `git clone`.
* `rcclSelectAllGather` signature changed — it now takes a `cudaStream_t stream` parameter (for live dispatch) and a `bool query` mode that uses `graphCapturingHint` instead of probing the stream. The comment block was updated too.
* `RCCL_DDA_THRESHOLD`, `RCCL_DDA_LL_THRESHOLD`, and `RCCL_DDA_LL128_THRESHOLD` now default to `-1` (unset), resolved at runtime from the arch dispatch table instead of being hardcoded to 128 MiB and 64 KiB respectively (RCCL_DDA_LL128_THRESHOLD had no pre-table default; the LL128 tier was off by default). Setting these env vars explicitly still takes precedence over the table.
* `RCCL_DDA_LL128` now defaults to `-1` (auto): the LL128 tier is enabled only for architectures whose arch-table row has a non-zero `ddaLL128Max` for the collective (currently gfx1250 only).
* DDA LL128 eligibility functions (`ddaLL128ArOneShotEligible`, `ddaLL128ArTwoShotEligible`) no longer check `RCCL_DDA_LL`. LL and LL128 are now independently gated: `RCCL_DDA_LL=0` disables only the LL tier; `RCCL_DDA_LL128=0` disables only the LL128 tier. Both checks are enforced via `rcclDdaLLThresholdTab` / `rcclDdaLL128ThresholdTab` at the selector level, which return 0 when the respective param is disabled.
* DDA LL128 is disabled for ReduceScatter and AlltoAll on gfx1250: `ddaLL128Max[RS]` and `ddaLL128Max[A2A]` are 0 in the gfx1250 arch table.
* `RCCL_CE_ALLREDUCE` now defaults to `-1` (auto/enabled) instead of `0` (disabled). CE AllReduce is therefore on by default for gfx1250 communicators that meet all other eligibility criteria.
* CE-2-Shot AllReduce size cap is now resolved at runtime via `rcclCeAr2ShotMax()` (env var wins, then arch table `ceNonRegMax[AR]`, then the 256 MiB `NCCL_CE_AR_TMPBUF_DEFAULT_BYTES` fallback) rather than from a compile-time constant.
* `RCCL_FORCE_CE_ALLREDUCE=1` no longer overrides the CE AllReduce 2-shot selector cap. It bypasses the `CTA_POLICY_ZERO` check only; the selector cap is enforced regardless.
* With `NCCL_CTA_POLICY=2` (`NCCL_CTA_POLICY_ZERO`), AllReduce on registered symmetric windows now takes the registered CE path whenever it is available, even when the symmetric kernel is eligible, matching AllGather and upstream NCCL. A non-zero arch-table `ceRegMax[AllReduce]` no longer caps it under this policy; a zero entry and an explicit `RCCL_CE_AR_REG_MAX_MSG_BYTES` still apply.
* `RCCL_FORCE_CE` (default `1`): acts as a prerequisite gate for CE-scratch selection. When `RCCL_FORCE_CE=0`, CE-scratch is disabled entirely for all collectives regardless of other flags. When non-zero, CE-scratch fires if `CTA_POLICY_ZERO` (`-x 2`) is set (bypassing the size window) or if the message size falls within the arch-table `ceNonRegMax` window. AlltoAll CE-scratch now has a size window (`ceNonRegMax[AlltoAll]` in the per-arch table, currently 0 on all arches).
* `tuning_model_11` registered for gfx1250 single-node; gfx1250 added to the AINIC tuning index map.
* Extended `ncclFunc_t` enum in rccl-tests `common.h` to match the librccl internal ABI: renumbered existing values and added `ncclFuncAlltoAll` (8), `ncclFuncScatter` (9), `ncclFuncGather` (10), `ncclFuncAllToAllPivot` (11), `ncclFuncAlltoAllGda` (12), `ncclFuncAlltoAllvGda` (13), `ncclFuncAllGatherV` (14), `ncclFuncPutSignal` (15), `ncclFuncSignal` (16), `ncclFuncWaitSignal` (17), `ncclFuncAlltoAllv` (18); `ncclNumFuncs` updated to 19.
* AlltoAll index variables in `alltoall_dda.h` and `alltoall_dda_fabric.h` widened from `int` to `size_t` to prevent overflow on large message sizes.
* DDA fabric and IPC AlltoAll launchers refactored to derive launch geometry from a single inline helper (`ddaAllToAllFabricGeom`, `ddaAllToAllFabricLLGeom`, `ddaAllToAllFabricLL128Geom`, `ddaAllToAllIpcGeom`) shared between the dispatch path and the new block-count query functions.
* `rcclGetCollImplInfo` now covers ReduceScatter and AlltoAll (previously both fell back to `rcclGetAlgoInfo()`); rccl-tests reports accurate algo/proto for all four collective types.
* Algorithm selectors (`rcclSelectAllReduce`, `rcclSelectAllGather`, `rcclSelectReduceScatter`, `rcclSelectAlltoAll`) now tag the task with `rcclSymkExtract` (`ALLOW`/`DENY`) so `taskAppend` does not re-derive the symmetric kernel decision. Setting `NCCL_ALGO=Ring` now correctly prevents the symmetric kernel from being extracted even when both buffers are registered.
* **Breaking: `NCCL_GIN_TYPE` values for AMD backends are not compatible with 2.30.7.** NCCL 2.31 inserted EFA GDA at value 5, so rocSHMEM GDA moved 5→6 and Anvil SDMA moved 6→7. The IB proxy remains `2`. Jobs that still set `NCCL_GIN_TYPE=6` now select rocSHMEM GDA, not Anvil SDMA. See `src/gin/README.md`.
* One-sided RMA supports multiple contexts and signals; the previous restriction to context 0 and signal index 0 has been lifted (`numRmaCtx` / `numRmaSig`).
* Updated the RMA plugin interface to v15.
* Reduced communicator host memory by allocating topology path link arrays to their actual length.
* Widened the LL128 per-thread shared-memory slice from 8 to 32 elements on gfx1250 (MI450/MI455); other architectures are unchanged. Forced Ring/LL128 bandwidth roughly doubles at 16 MB and above. Per-block LDS rises from 36448 to 85472 bytes, and because the scratch region is shared across protocols this applies to every kernel launch, not only LL128 ones. The tuner's protocol selection is unchanged, so the gain is only visible when LL128 is selected explicitly.
* Registered-window CE AllReduce is now eligible without requiring ``NCCL_CTA_POLICY=2``
  (``CTA_POLICY_ZERO``); it is gated only by the per-arch table window and ``RCCL_CE_ALLREDUCE``.
  ``RCCL_FORCE_CE_ALLREDUCE`` now applies exclusively to the non-registered (2-shot/staged) CE
  AllReduce path.
* `RCCL_FORCE_SYMMETRIC` (default `0`): when set to a non-zero value, biases the `ncclAllReduce`
  backend selector to prefer the symmetric kernel path over all other candidates. The selector
  still requires the operand buffers to be registered symmetric windows; if the symmetric
  path is ineligible the selector falls back normally.
* `RCCL_FORCE_DDA` (default `0`): when set to a non-zero value, biases the `ncclAllReduce`
  backend selector to prefer DDA (Direct Data Access) paths over all other candidates,
  including symmetric and Copy Engine. If no DDA path is eligible for the given message
  size and communicator the selector falls back normally.

##### Removed
* `NCCL_TOPO_SCATTER_START_NET`, which selected how the first network device was scattered across GPUs. Rail and plane assignment replaces it.
* `RCCL_DDA_LL128_RS_THREADS` removed (was default `1024`): reduce-scatter LL128 now launches a fixed 512-thread block, one warp per slice, so the thread count is no longer a tuning knob.

##### Resolved issues
* Fixed topology GPU indices that followed the HIP device order instead of the PCI order. This reordered rings and NIC selection, causing an `ncclGather` performance regression on multi-node MI355X systems, and made communicator initialization fail when each rank sees only its own GPU.
* Anvil SDMA (`NCCL_GIN_TYPE=7`) honours strong GIN signals. A strong signal drains every SDMA queue to that peer before it is sent, including puts that are no longer marked dirty. When the signal is fused with its SDMA put (gfx950), the drain covers only the earlier queue entries and the caller does not wait for the fused put itself. Legacy `ncclGin_SignalInc` is strong by default, so a caller that only needs to order the put it just issued should use `ncclGin_WeakSignalInc`. A barrier on this backend still flushes once and sends weak signals.
* Fixed VA GIN signals (`ncclGin_VASignalInc`, `ncclGin_VASignalAdd` and their strong and weak forms) on Anvil SDMA (`NCCL_GIN_TYPE=7`). The backend read the VA descriptor as an indexed signal ID and incremented an unrelated signal cell, so the VA signal never arrived. VA signals now update the window cell on the peer, follow the same strong, weak and fused-signal rules as indexed signals, and `ncclGin::resetSignal` on a VA signal clears the local cell.
* Fixed profiler overhead when no profiler plugin is loaded or only kernel-channel events are enabled.
* Fixed profiler API events reporting rank 0 instead of the originating communicator rank.
* Fixed P2P IPC registration reuse producing out-of-bounds remote addresses when a registered allocation spans multiple cuMem segments.
* Fixed non-thread-safe token parsing that could corrupt configuration parsed concurrently from multiple threads.
* Fixed tuner plugins receiving uninitialized cost-model constants.
* Fixed GIN proxy descriptor shared-memory sizing and alignment.
* Fixed virtual address space exhaustion when symmetric windows backed by the same physical allocation are registered repeatedly.
* Fixed Copy Engine `ncclAlltoAllv` hanging on intra-node communicators of more than 8 ranks with uneven send sizes. Each rank decided from its own send sizes whether to add intra-batch barriers, so ranks could issue different barrier counts. The decision now uses the full send-size matrix, which every rank shares.
* Fixed `NCCL_P2P_DISABLE` and `NCCL_P2P_LEVEL` being ignored on Intel, ARM and Zhaoxin hosts, where the CPU-based default P2P level overwrote the user setting. On such hosts `NCCL_P2P_DISABLE=1` now falls back to SHM as documented, and `NCCL_P2P_LEVEL` is honored.
* Fixed `NCCL_P2P_DISABLE=1` selecting the P2P transport between ranks that share a topology device node, such as MI300X CPX partitions with `NCCL_TOPO_SPLIT_MLOPART=0`. Transport selection now falls back to SHM while preserving zero-hop GPU self paths.
* Fixed LL-protocol latency on gfx950 when RCCL is built against a ROCm older than 7.0.2 without the 128-bit global load/store builtins (for example ROCm 7.0.1, or `-DDWORDX4_INTRINSICS=OFF`). Every store to the user output buffer was followed by a release fence and an L2 writeback, which made 4 KB to 64 KB `ncclBroadcast` about 30% slower on MI355X. Builds against ROCm 7.0.2 or later were not affected and are unchanged.
* Fixed a 15 to 30% `ncclScatter` bandwidth loss from 16 MB upward on multi-node AINIC jobs, caused by the move to two network channels per peer (for example on 4 nodes of 8 GPUs). Scatter now uses and connects the most channels per peer whose flows still fit the P2P channel pool. Point-to-point operations and the other collectives keep their channel count.
* Fixed intermittent data corruption in multi-node collectives, seen as out-of-bounds values in 8-node AllToAll, on hosts that use GPU Direct RDMA through peermem. Since RCCL 2.30.4 the GDR flush scratchpad was registered only through dma-buf, so on peermem hosts the flush degraded to a single RDMA read of the receive buffer, which does not order earlier relaxed-ordering writes. The scratchpad is now also registered through peermem, restoring the strictly ordered RDMA write that precedes the flush read.
* Fixed `ncclCommAbort` and `ncclCommRevoke` hanging when a DDA collective (IPC on gfx942/gfx950, fabric LL/LL128/VMM on gfx1250) was waiting on a peer that never joined. DDA kernels now observe the communicator abort flag while waiting. A parent communicator shrunk with `NCCL_SHRINK_ABORT` no longer uses DDA for later collectives.
* Fixed wrong results and hangs in application kernels that use the GIN device API, such as the `07_kernel_fusion` GIN and hybrid RMSNorm examples. `nccl_device/coop.h` selected `__syncwarp` from `ROCM_VERSION`, which only RCCL's own build defines, so in application code the thread and warp cooperative groups synchronized the whole block with `__syncthreads()`. The GIN barrier calls them from a subset of threads, which left the block's waves out of step. The headers now fall back to the HIP version when `ROCM_VERSION` is not defined.
* Fixed FP8 E4M3 and E5M2 `ncclAvg` returning wrong values for every element of every reduction collective on gfx942, and on architectures that use the software FP8 fallback. The `1/nRanks` scale was packed as an FP8 value in the host's OCP encoding, which those devices decode as FNUZ, so every result came out half as large. The scale is now carried as `float` on all architectures, which also keeps it exact at rank counts FP8 cannot represent: in OCP E4M3, `1/384` was off by 25% and `1/1024` rounded to zero. Single-rank FP8 reductions on gfx942 no longer fail to launch the `oneRankReduce` kernel, and rccl-tests no longer skips FP8 `avg` for `all_reduce`, `all_reduce_bias` and `reduce`.
* Fixed the GIN plugin never being finalized for a communicator that had connected a GIN backend, leaking the plugin's per-communicator context and leaving its reference count one too high after every `ncclCommDestroy`. `ncclGinHostFinalize` closed the GIN connections and then zeroed the whole `ncclGinState`, so the `ncclGinFinalize` that follows it in `commFree` saw no active backend and skipped the plugin `finalize()` call. Host finalize now clears only the per-connection state and hands the backend records to `ncclGinFinalize`, which finalizes each one on the last shared-resource reference. `commFree` returns the first of a `closeColl` or plugin-finalize error after those frees; if both fail, the `closeColl` error is the one reported. An `ncclRmaProxyFinalize` error still aborts the rest of teardown immediately. On the Anvil SDMA backend (`NCCL_GIN_TYPE=7`) this was one leaked init context per connected communicator.
* Restored topo tuning-model init (`ncclTopoTuneModel`) after the 2.31 `ncclTuningInit` switch so multi-node kernels do not launch with `blockDim.x=0`.
* Grouped multi-rank finalize to match the v2.31 teardown barrier.
* Fixed a crash (`HSA_STATUS_ERROR_MEMORY_APERTURE_VIOLATION`) on gfx110x and gfx120x when the Tree algorithm runs over shared memory with more channels than the Tree graph search found, for example with `NCCL_ALGO=Tree` and `NCCL_P2P_DISABLE=1`. Channels added after `ncclTopoPreset` had no intra-node tree links; `ncclTopoPostset` now sets the tree and CollNet chain links on every channel, as it already did for rings.
* Fixed Copy Engine `ncclAllGather`, `ncclAlltoAll`, `ncclScatter` and `ncclGather` returning incorrect data, and no error, when `NCCL_LSA_TEAM_SIZE` was set below the number of ranks on the node. These routines index both their peer list and their buffer offsets by LSA rank, so a communicator wider than its LSA team exchanged data only within that team and placed it at LSA rather than communicator offsets, leaving the slices owned by every other rank unwritten. Copy Engine selection now requires the LSA team to cover the whole communicator on the scratch (`RCCL_FORCE_CE`) path as well as the registered-window path, so such communicators use the kernel path instead. Runs that leave `NCCL_LSA_TEAM_SIZE` unset are unaffected, because the LSA team then spans every rank on the node.
* Fixed the host one-sided RMA API being unavailable on multi-node communicators whose ranks had fused NICs. NIC fusion merges devices for the regular transport only; GIN and RMA select from their own topology node types and never receive a fused device.
* Fixed hierarchical Copy Engine collectives dereferencing a NULL RMA proxy context when `NCCL_RMA_DISABLE=1` was combined with the hierarchical opt-ins. `NCCL_RMA_DISABLE` suppresses the window registration that creates those contexts, but the hierarchical availability check did not consult it, so a multi-node communicator with `NCCL_CTA_POLICY=2` and symmetric send and receive windows admitted the path and then crashed at launch. The internal RMA context predicate now defers to `ncclRmaProxyEnabled`, so such a communicator declines the hierarchical path and uses the kernel path instead.
* Fixed Copy Engine `ncclAllReduce` hanging when one thread drives several GPUs, as in rccl-tests `-g 4`. The staging buffer was set up on first launch, and that setup is a collective window registration, so the first rank waited for peers the same thread had not launched yet. The setup is now queued when the collective is enqueued and runs in each communicator's own group job at `ncclGroupEnd`.
* Fixed Anvil SDMA GIN small-message AllToAll latency after `ncclGinFenceLevel` support. A sub-threshold `gin.put`, a `gin.putValue`, or a standalone `gin.signal` emitted two system-scope fences per peer: one in `fenceBeforeSignal` and one in `signalPeer`. `signalPeer` now emits a single system-scope release immediately before the remote atomic, rather than an agent-scope release before it and a system-scope fence after it, which also stops payload stores from passing the signal cell. A clean-queue IPC `SignalInc` skips `fenceBeforeSignal` entirely, since it would only repeat that release. The SDMA `quiet()` before a signal now runs only when the call itself issued SDMA; previously any call carrying a signal or counter quieted the peer queue, including an IPC put after an earlier SDMA put to the same peer, which now relies on `Flush` to drain that queue. The `NCCL_GIN_ANVIL_SDMA_IPC_AGENT_FENCE` debug knob is removed. Each `Flush` thread drains the peers it owns and then clears exactly the bits it drained, over a mask disjoint from every other thread's. Previously `Flush` loaded a snapshot, quieted it, and cleared the whole word on the coop's leading thread, on every rank, which could drop a concurrent mark whose doorbell had not yet fired and then skip the quiet that mark needed. Clearing after the drain also keeps the invariant that a clear bit means a drained queue, so a second coop flushing the same context cannot return from `Flush` while the first is still inside `quiet()`.
* Fixed queue evictions on every GPU at process exit: RCCL took a ROCr reference with `hsa_init()` when it checked DMA-BUF support and never released it, so ROCr was not torn down before its GPU-mapped buffers were unmapped.
* Fixed multi-receive `irecv` failing with a fatal network error in the `IB` transport when `NCCL_IB_USE_INLINE=1`.
* Fixed Anvil SDMA (`NCCL_GIN_TYPE=7`) GIN contexts sharing signal cells across logical contexts on a connection, which let a consecutive `ncclGinAllContexts` barrier return early on a stale count. Each logical context now has its own signals.
* Fixed GIN signal shadows being sized for the requested context count instead of the rounded-up `ginContextCount`, which let the last contexts write past the shadow region when the GIN connection count does not divide the requested context count.

##### Known issues
* The FP8 ReduceSum and ReduceCopy device APIs are not exported in the LLVM bitcode library.
* On the Anvil SDMA (`NCCL_GIN_TYPE=7`) and rocSHMEM GDA (`NCCL_GIN_TYPE=6`) GIN backends, `ncclGinFlushTimeout` ignores the timeout and blocks until the flush completes; `ncclGinWaitTimeout` does the same on Anvil SDMA. rocSHMEM GDA does not support `ncclGinFlushAsync` or waiting on a GIN request, and those calls trap on the device. The signal and counter wait timeouts work on all backends.
* The following NCCL 2.31 features are NVIDIA-specific and are not available in RCCL: Compute Fabric Transport, the GDAKI and EFA GDA GIN backends, PAT combined with NVLS, NVLink-multicast AllGather, and the CuTeDSL and `nccl4rust` bindings.

#### **rocBLAS** (5.8.0)

##### Added

* gfx1170 and gfx1171 enabled. Level 3 `gemm`, `gemm_ex`, and the functions that internally use GEMM default to the hipBLASLt backend on these architectures. `ROCBLAS_USE_HIPBLASLT` continues to force or disable the hipBLASLt backend.

##### Changed

* On gfx1250, rocBLAS loads the Tensile library catalog that matches the device revision. Strict-revision silicon uses `library/gfx1250-strict` when that catalog is present in the build, and otherwise falls back to `library/gfx1250`.
* On gfx950, hipBLASLt is now the default GEMM backend for every input type except real `f32_r`. Real single precision still defaults to Tensile. `ROCBLAS_USE_HIPBLASLT` continues to force or disable the hipBLASLt backend.
* On gfx942 double-precision Level 3 `gemm`, `gemm_ex`, and the functions that internally use GEMM now default to the hipBLASLt backend. gfx942 devices other than the MI300A (gfx942 with 228 compute units) also default for to hipBLASLt for `f16_r/bf16_r` data, while the remaining data types on gfx942 default to Tensile. `ROCBLAS_USE_HIPBLASLT` continues to force or disable the hipBLASLt backend.
* Batched GEMM now uses the same hipBLASLt selection as other GEMM while a stream is capturing. Previously, batched APIs were forced to the Tensile backend during stream capture.
* `trsm`, `trsm_batched`, and `trsm_strided_batched` print the required and available workspace sizes when they return `rocblas_status_memory_error` because a fixed-size handle workspace is too small if the new `ROCBLAS_VERBOSE_ERROR` environment variable is set. Previously this failure looked identical to running out of device memory.

##### Optimized

* Improved the performance of Level 2 `gemv` non-transposed (`TransA == N`) by selecting the n-split reduction from the launch shape rather than a fixed output-length crossover. The split applies when the output grid has at most 8 tiles and the column split produces at least 2 parallel blocks, or when the output length is at or below the crossover.
* Improved the performance of Level 3 `gemm` and `gemm_ex` for transposed operands with a very large summation dimension. k-chunking, which keeps Tensile element offsets inside 32 bits, previously used `max(lda, ldb)` for every transpose combination, so a unit-stride summation was split into many small launches. The bound is now the k stride of each operand: the leading dimension when that operand is k-major, and 1 otherwise. k-major layouts that can overflow are still chunked.

##### Removed

* The `ROCBLAS_USE_HIPBLASLT_BATCHED` environment variable, deprecated in `rocBLAS 5.6.0`, is removed and is now ignored. Batched GEMM is controlled by the same environment variable as all other GEMM, so a batched-only override is no longer required. Use `ROCBLAS_USE_HIPBLASLT=0` to select the Tensile backend for all GEMM, including batched.

##### Resolved issues

* Fix incorrect results and out-of-bounds reads from Level 1 ILP64 `dot` and `dotc`, including batched, strided-batched, and `_ex` forms, when a negative increment is wide enough to take the 64-bit increment path and `n` fits the single-block reduction. That path shifted by `(n - 1)` before calling the launcher, which applies the same walk, and the `y` shift tested `incx` rather than `incy`. The offsets are now passed through unshifted.
* Fix incorrect results and out-of-bounds indexing from Level 1 ILP64 `axpy` and `axpy_ex`, including batched and strided-batched forms, when `batch_count` is greater than 65535 and an increment is wide enough to take the 64-bit kernel. That kernel was launched with the full `batch_count` rather than the chunk length, so each pass walked past the current batch and applied `alpha * x` again.
* Fix incorrect results and out-of-bounds accesses from Level 1 ILP64 `rot` and `rot_ex`, including batched and strided-batched forms, when `n` or `batch_count` is large enough to chunk the launch and `incx` or `incy` is negative. Chunk offsets were `n_base * inc`, and the wide-increment path used `-inc * n_base`, so a negative increment started at the wrong end of the vector. Each chunk now starts at `offset + (inc < 0 ? -inc * (n - n_chunk - n_base) : n_base * inc)`.
* Fix Level 2 `trsv` and Level 3 `trsm`, `trtri`, `trmm`, and `gemmt` returning `rocblas_status_success` when an internal GEMM, GEMV, vector copy, or block helper failed. Those statuses are now returned to the caller. On the negative-`incx` path, a failed `trsv` also restores the caller's input vector, which the solve reverses in place.
* Fix `rocblas_get_commit_hash_string` writing past the caller's buffer. The length check compared against the size of a pointer, so a buffer shorter than the commit hash was accepted, and the copy omitted the terminating null. `rocblas_get_version_string` and `rocblas_get_commit_hash_string` now require the size reported by the matching `*_size` query, including the terminating null.

#### **ROCdbgapi** (0.82.1)

##### Added
- Support for the following architectures:
 - `gfx1170`
 - `gfx1171`
 - `gfx1172`
 - `gfx11-7-generic`

#### **RDC** (1.4.0)

##### Changed

* **`rdc_health_get()` returns `RDC_ST_GROUP_NOT_FOUND` for a GPU group that does not exist**.
  * It returned `RDC_ST_OK` with no health watches, so `rdci health -f` showed every watch as Off and exited successfully, and `rdci health -c` failed asking for watches to be enabled. Both commands now fail with a message that the group does not exist.

##### Resolved Issues

* `rdci health -s` on a GPU group that does not exist no longer leaves an unused field group in rdcd. Repeated failures filled the field group table, and commands that need a new field group, such as `rdci dmon`, then failed until rdcd was restarted.

* `rdc_status_string()` now describes `RDC_ST_DISABLED_MODULE`. It returned "Unknown", so the rdcd log reported a disabled module as an unknown error.

* `RDC_FI_POWER_USAGE` now reports the socket power of AMD Instinct MI300 and newer GPUs. It reported 4294967295000000 µW, so power policies on these GPUs triggered on every check.
  * Partitions that have no socket power reading of their own now report N/A instead.

* `rdci topo` without `-i` now always reports that the GPU index is missing. It could instead skip that check and print the topology of an arbitrary GPU or an unrelated error.

* RDC no longer reports another GPU's XGMI link states, or garbage values that could crash `rdcd`, for a GPU whose link status AMD SMI cannot read. That GPU now reports no links.
  * `rdci link` now prints an error and exits with a nonzero status when it cannot get the link status, instead of printing nothing and exiting with 0.

* The `RDC_FI_GPU_MEMORY_CUR_BANDWIDTH` fix for zero readings under DMA or copy-only memory traffic is now included.

* `rdci topo` and `rdc_device_topology_get()` now report the links of the requested GPU. They reported links between other GPU pairs, which showed the wrong link types on systems where GPU pairs are not all linked the same way, such as XGMI hives joined by PCIe.
  * `rdci topo` now exits with an error code when the GPU index is out of range or the topology query fails, instead of exiting 0, and `rdc_device_topology_get()` reports link types RDC does not define as `RDC_IOLINK_TYPE_UNDEFINED`.

* Opening or stopping an RDC handle no longer crashes the process when another handle is in use, and concurrent `rdc_connect()` calls no longer fail at random.

* Certificates signed with the RDC certificate authority setup in `authentication/openssl.cnf` can no longer sign other certificates. A certificate request could ask to become a signing certificate, and its holder could then create client and server certificates that `rdcd` and `rdci` trust.
  * If you signed requests that other people created, check the certificates in the `newcerts` directory of your CA for `CA:TRUE`. If you find one, create a new root certificate and reissue all certificates.

* `rdci discovery` now reports an error instead of exiting successfully with an incomplete GPU list when `rdcd` cannot read a GPU's attributes. Failed `rdc_device_get_attributes()` calls now return zeroed attributes, and `rdcd` replies to failed requests no longer contain garbage values.

* `rdci dmon` and `rdci health -s` name the missing group again, for example `Cannot find the group 4242`. Since ROCm 7.12 they printed only `Group not found` (or `Fieldgroup not found` for a missing field group in `rdci dmon`).

* `RDC_FI_GPU_PARTITION_COUNT` now reports how many compute partitions the GPU is split into, for example 1 in SPX mode. It reported 65535 or N/A in every partition mode on most GPUs.
  * A GPU that cannot be partitioned now reports 1.

* Stopping `rdcd`, or embedded RDC with `rdc_stop_embedded()`, no longer hangs. Rarely, the stop never returned and the process had to be killed.


#### **rocDecode** (1.12.0)

##### Added

* Invalid video size handling for AVC/HEVC.
* Added support for explicitly loading librocm_sysdeps_va via dlopen, ensuring complete isolation from the system libva library.
* Initial Windows support: hardware-accelerated video decoding on Windows via the vaon12 backend (Mesa's VA-API on D3D12 translation layer). Requires HIP and the vaon12 VA-API driver from TheRock for Windows.
* On Windows, the TheRock VA-API DLLs (`rocm_sysdeps_va_win32.dll`, `rocm_sysdeps_va.dll`) are loaded at run time, matching the Linux dlopen behavior. `%ROCM_PATH%\lib\rocm_sysdeps\bin` no longer needs to be on `PATH`.
* HEVC: generation of unavailable reference pictures for a BLA picture or a CRA picture with NoRaslOutputFlag equal to 1 (H.265 clause 8.3.3).

##### Resolved issues

* Fixed decode errors of some AVC interlaced container streams by adding support for the picture data packet from the demuxer which contains multiple pictures.
* Corrected fake CTest passes.
* Resolved vendored libva link issue in samples without extra env vars.
* Fixed a stray RASL picture being output from the first CRA picture after an application signals end of sequence with the `ROCDEC_PKT_ENDOFSTREAM` packet flag. The flag now ends the coded video sequence the same way an in-band EOS NAL unit does, so the next IRAP picture gets `NoRaslOutputFlag` equal to 1.
* Fixed out-of-bounds memory access issues in the HEVC, AVC, AV1, VP9 and base parsers.
* Fixed `rocDecGetDecoderCaps` advertising 16-bit output surface formats for 8-bit content.

#### **rocFFT** (1.0.41)

##### Added

* Added further support for very large FFTs (length greater than 2^32).
* JIT callbacks specified with the `rocfft_plan_description_set_load_callback` and
  `rocfft_plan_description_set_store_callback` APIs can now be used with transforms that have fields
  specified on the same plan description.

##### Changed

* Function pointer callbacks specified with the deprecated `rocfft_execution_info_set_load_callback` and
  `rocfft_execution_info_set_store_callback` APIs are now disallowed with transforms that have fields
  specified on the plan description.

##### Resolved issues

* Fixed possible failures of `rocfft_plan_create` for multi-device plans.
* Fixed out-of-bounds memory access when executing a multi-device complex-to-complex plan whose
  bricks used non-contiguous data layouts.
* Fixed a memory leak when `rocfft_plan_create` fails.
* Fixed `rocfft_plan_create` failures for large prime-length 1D complex transforms when a scale factor is set.
* Fixed a potential write-after-free if plans are destroyed during process teardown.
* Fixed a possible hang in `rocfft_cleanup` in builds with RCCL enabled, for single-process multi-device usage.
* Fixed incorrect results for 1D complex transforms of lengths involving large prime factors
  with large batch sizes, where 32-bit kernel indexing could overflow.
* Fixed `rocfft_plan_create` failures for 1D complex transforms of lengths involving large prime
  factors when the output data layout uses a batch distance larger than the transform length.
* Fixed `rocfft_plan_create` failures and GPU memory access faults for 1D complex transforms of
  lengths involving large prime factors when batches are interleaved (unit batch distance).

##### Optimized

* Improved performance of unit-strided, interleaved, complex-to-real FFTs on gfx1201, gfx90a, gfx942, and gfx950 for the following lengths:
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
  * (216,104,104)
  * (216,104,100)
  * (224,104,104)
  * (224,108,104)
  * (224,108,108)

#### **rocJPEG** (1.13.0)

##### Added

* Added the `rocJpegGetDecoderCaps` API and the `RocJpegDecodeCaps` structure to query the decode capabilities of a rocJPEG handle, including the number of JPEG cores, the supported JPEG coding processes, image resolutions, chroma subsamplings, and output formats, the native output surface format of each chroma subsampling as verified with the VA-API driver, and the hardware ROI decode and color conversion support.
* Added the `RocJpegSurfaceFormat` enum to describe the layout of the decoded image produced by the VCN JPEG decoder, and the `RocJpegCodingProcess` enum to report the supported JPEG coding processes.
* Added the `jpegDecodeCaps` sample to demonstrate the `rocJpegGetDecoderCaps` API.

#### **ROCm Compute Profiler** (3.10.0)

##### Added

* Added the `LDS Utilization` metric to the gfx115x Memory Chart.

* Added two wave utilization metrics to PC sampling analysis.
  * `active_thread_percent` is the percent of a wave's lanes that were active at an instruction, so a low value points at control flow divergence. Both sampling methods report it.
  * `wave_occupancy_percent` is the percent of the machine's wave slots that held a wave. Only stochastic sampling reports it, because a host-trap record carries no wave count.
  * Both appear in the analyze terminal table and in each kernel's `per_kernel_pc_sampling/` CSV.

* Added the two wave utilization metrics to the analysis database summary view, so `compute_pc_sampling_summary_view` and the `pc_sampling_summary.csv` export carry them alongside the sample counts.

* Added a profile-mode warning on gfx115x when the `AUTO` performance level can gate the perfmon clock and zero PMC counters such as `TCP_REQ`, with a link to the ROCprofiler-SDK `STABLE_STD` workaround.

* Added CLI guidance for viewing the wide memory chart without line wrapping (`less -RS` or `code -`).

* Added Python 3.14 support.

* Added an analyze-only install path. `pip install "rocm-profiler[compute-analyze]"` into a virtual environment installs the analysis dependencies for you.

* Added `rocprof-compute analyze --verify-deps`, which reports any missing analysis dependencies.

* Added Memory Bandwidth Analysis for gfx950 under `--experimental --membw-analysis`. Profile with the flag to collect block 30 counters; analyze detects the data automatically and annotates active GL1/GL2/EA stall bottlenecks on the memory chart with a guidance panel.

* Added the rocprof-compute Agent Skills and the documentation they link to to the install under `share/rocprofiler-compute/`, with an `install-skills.sh` script that installs or removes them for Claude Code, Codex, or Cursor. `rocprof-compute --help` shows where to find them.

##### Changed

* Dispatch IDs now start at 1 instead of 0.

* Renamed the profile-mode dispatch filter to `--kernel-iteration-range`, matching the rocprofv3 option it drives. Update any profile command by replacing `-d/--dispatch` with `--kernel-iteration-range` to select dispatches.
  * `--dispatch` is no longer accepted in profile-mode.
  * `-d` is now the short form of `--output-directory` in profile-mode, also matching rocprofv3.

* gfx115x Memory Chart improvements.
  * Renamed memory chart metric names for more clarity.
  * Each edge now reports the traffic measured at the interface it represents.
  * Updated arrows, labels, and the legend in the memory chart to better represent their meaning.

* The Torch trace collector now ships prebuilt for PyTorch 2.13 and 2.14, so
  `--torch-trace` traces operators on every thread, including the autograd
  threads that run the backward pass.

* Named each per-kernel PC sampling folder `<short_name>_uuid_<kernel_uuid>` instead of `kernel_<kernel_uuid>`, and added the matching `short_name` column to `kernel.csv`. The short name is the demangled identifier captured while profiling.

* Redesigned the CDNA (gfx9) Memory Chart with a new Rich-based layout that improves readability in the terminal. Added Non-buffer/Buffer request breakdowns (Read/Write/Atomic wavefronts) and L2-Fabric bandwidth metrics across all CDNA architectures.
  * gfx908–gfx942: added HBM and remote traffic percentages.
  * gfx950: added LDS Read/Write/Atomic instruction counts and per-channel bandwidth for HBM, xGMI, and PCIe.

* Analyze mode now auto-detects memory bandwidth analysis data from the profiling output. The `--membw-analysis` analyze option has been removed; use `--membw-analysis` only at profile time.

* Renamed gfx1250 metric labels for clarity.
  * `VMEM Atomic RTN` to `VMEM Atomic Return` and `VMEM Atomic NORTN` to `VMEM Atomic Non-Return` in the WGP VMEM Instruction Mix panel.
  * `LDS Atomic RTN` to `LDS Atomic Return` and `LDS Atomic NORTN` to `LDS Atomic Non-Return` in the WGP LDS Instruction Mix panel.
  * `Total Requests - Sectors` to `Total GL0 Sectors`, `Total Requests - Sector Reads` to `GL0 Read Sectors`, and `Total Requests - Sector Writes` to `GL0 Write Sectors` in the GL0 Cache and LDS panel.

##### Removed

* Removed the `--kernel-verbose` analyze option and the kernel name shortener it drove. The option had no effect on any output.

* Removed the Nuitka standalone binary build (`STANDALONEBINARY`, `STANDALONEBINARY_EXTRACT_DIR`), its RHEL 8 docker recipe, and the `--call-binary` pytest option that exercised it.

* Removed the `SKIP_NATIVE_TOOL_BUILD` build option. The counter collection tool is always built, and its sources are no longer installed for runtime compilation.

* Removed the deprecated `Active CUs` metric from the System Speed-of-Light panel and the Memory Chart SVG for all CDNA architectures (gfx908, gfx90a, gfx940, gfx941, gfx942, gfx950). Use `CU Utilization` instead.

##### Optimized

* HBM and remote traffic percentages are now more accurate, with all their counters collected in a single profiling pass.

* Analyze mode produces less warning noise. Repeated warnings are de-duplicated, and messages for metrics that evaluate to N/A moved to debug level.

* Analyze mode uses less disk space. It reads the profiling output directly instead of writing an extra merged copy of all counter data into the workload directory.

* Analyze mode now stops as soon as it finds unreadable profiling data, instead of continuing with an incomplete set of counters.

* Improved the profiling failure message when the workload and the profiler load different ROCm installations. The error now points to the PyTorch and `rocm[profiler]` install instructions instead of only showing the LLVM abort.

##### Resolved issues

* Fixed the standalone roofline HTML so it always opens with the same axes for a given GPU, which makes two runs comparable. The axes come from the benchmarked bandwidth and compute ceilings, not from the kernels in the run.

* Fixed `L2 Cache (per Channel)` labels to use a `Metric` column and numbered `Channel` row labels in CLI, TUI, and analysis database output.

* Fixed `--set` running the roofline microbenchmark, which is never part of a metric set.

* Fixed PC sampling source snapshots to use canonical paths and include source contents and checksums in analysis exports.

* Fixed false `0` values in the gfx115x Memory Chart; missing counter data now reports `N/A`.

* Fixed `GL2-Fabric Write BW` understating write bandwidth on gfx115x in the System Speed-of-Light and Memory Chart panels.

* Fixed `profile -b 3 --experimental --membw-analysis` not collecting memory bandwidth analysis counters.

* Fixed `analyze -b` crashing when only the gfx950 LDS bandwidth metrics are selected.

##### Upcoming changes

* The analyze-mode `--gui` and `--tui` options are deprecated and will be removed in a future release. Use the default CLI analysis instead.

##### Known issues

* On gfx115x, `TCP_REQ*` counters and the `GL0` metrics derived from them can read zero because the perfmon clock is power-gated at the `AUTO` performance level.

#### **ROCm Debugger (ROCgdb)** (16.3)

##### Added

- Support for local memory (LDS) out-of-range address exception reporting
  on architectures that support it (gfx12.5 or later).  When enabled, GDB
  reports a SIGBUS instead of the access silently failing.  Off by default;
  can be set and viewed with:
  - `set  amdgpu lds-exception [on|off]`
  - `show amdgpu lds-exception`

#### **ROCm Systems Profiler** (1.10.0)

##### Added

- `rocsys` unified CLI entry point. `rocsys -- ./app` and `rocsys profile -- ./app`
  produce a full trace profile. `rocsys rewrite` runs `rocprof-sys-instrument`
  in binary-rewrite mode (`-o`). Other subcommands (`instrument`, `causal`,
  `avail`, `python`, `attach`) forward to the existing tools. The
  `rocprof-sys-*` binaries are unchanged.
- AMD Infinity Storage I/O telemetry. Per-GPU I/O counters are sampled into
  the profiler output. Enable collection with `ROCPROFSYS_USE_HIPFILE` and select metrics with
  `ROCPROFSYS_HIPFILE_METRICS`. See
  [hipFile Infinity Storage I/O telemetry](./docs/how-to/hipfile-telemetry.rst).
- `--detach-after <seconds>` flag on `rocprof-sys-attach` for non-interactive use; the binary
  sleeps for the given duration after attach instead of waiting for `ENTER` on stdin.
- `rocprof-sys-attach` finds the tool library in the target's own ROCm installation when the
  target cannot see this installation's library (for example, inside a container), so
  `ROCPROF_ATTACH_TOOL_LIBRARY` is no longer needed there.

##### Resolved issues

- Fixed a crash (`SIGSEGV`) in `rocprof-sys-instrument` when instrumenting Fortran+HIP
  programs compiled with `gfortran -g` or `-g -O0`. gfortran encodes assumed-shape and
  allocatable array bounds as DWARF location expression blocks rather than integer
  constants at debug optimization levels; DynInst was treating these block-form
  attributes as errors and dereferencing a null pointer in `parseSubrange()`.
  Fixed in [ROCm/dyninst#33](https://github.com/ROCm/dyninst/pull/33).
- Fixed `rocprof-sys-attach` aborting the target process when attaching to a process
  running in a different mount namespace (for example, a container). The tool library
  path is now validated against the target's mount namespace before attaching, failing
  cleanly with a diagnostic instead.

##### Removed

- Legacy direct Perfetto tracing mode. Perfetto traces are now always generated from the
  trace cache. `ROCPROFSYS_TRACE_LEGACY` and `--trace-legacy` / `-L` were removed (passing `-L` is
  now an unknown-argument error); the `tracing.legacy` key was removed from JSON
  configuration export.
- Timemory component annotations (for example `thread_cpu_clock` and PAPI counter
  values) are no longer attached to Perfetto slices. They will be available through the
  timemory output formats.

#### **rocPRIM** (5.0.0)

##### Added

* Added C++ 17 style type_traits utilities.
 * is_floating_point_v
 * is_integral_v
 * is_arithmetic_v
 * is_fundamental_v
 * is_unsigned_v
 * is_signed_v
 * is_scalar_v
 * is_compound_v
* gfx1250-strict support.

##### Optimizations

* Updated existing `ordered_block_id` and `lookback_scan` optimizations to include gfx1250/gfx1250-strict.
* `lookback_scan` has improved data padding/alignment to reduce false sharing.

##### Changed

* Replaced the soon-to-be-deprecated `__hip_atomic_*` builtins with the equivalent `__scoped_atomic_*` builtins.

##### Resolved Issues

* Fixed a crash in `histogram_even`, `histogram_range`, and the multi-channel histogram variants on gfx942 when the bin count does not fit in shared memory and the stream is `hipStreamLegacy`. The device is now resolved with `get_device_from_stream`, which accepts the default, per-thread, and legacy streams.
* Incorrect miscategorization of RDNA4 and CDNA5 GPUs.

#### **ROCprofiler-SDK** (1.4.2)

##### Added

- HSA API tracing support for `hsa_amd_enable_high_precision_timestamps`.

##### Changed

- rocPD schema 3.0.5 stores SPM counter location on `rocpd_info_pmc`. Each agent and combination of counter, XCC, shader engine, shader array, WGP, and instance has its own pmc id. Location dimensions not provided for a counter are stored as `NULL` and left blank in CSV output. The instance index is always stored. `rocpd_pmc_event` keeps the sample value and points at that id.
- Request ROCr high-precision timestamp correlation during HSA registration when GPU timestamp collection is configured and the runtime provides the tools API. Older runtimes and tools that only trace CPU APIs or markers retain the existing behavior.

##### Resolved issues

- When a context that needs packet interception replaces a queue created by `hsa_amd_queue_create` with a system-memory intercept queue, the device-memory placement flags are now cleared in the caller's descriptor. HIP reads these flags back and previously assumed a device-memory ring buffer that profiling had silently replaced.

#### **rocRAND** (5.2.0)

##### Optimized

* Added `__restrict__` qualifiers to the device API benchmark, enabling better compiler optimization.
* Reduce shared memory usage and use the ADDC intrinsic in the Philox generator.

##### Resolved Issues

* Internal kernels are no longer exported from `librocrand.so`.
* Fixed incorrect values generated by Sobol method when a single generation request crosses end of sequence period. Also fixes an out-of-bounds read in host generation.
* Fixed an issue with Sobol teardown aborting after hipDeviceReset.
* Fixed the host API scrambled Sobol generators using incorrect scramble constants. The output of ROCRAND_RNG_QUASI_SCRAMBLED_SOBOL32 and ROCRAND_RNG_QUASI_SCRAMBLED_SOBOL64 changes as a result.

##### Added

* New unit tests for `src/rng/sobol.hpp` in `test/sobol_exit/test_rocrand_sobol_exit.cpp` to cover the issue with Sobol teardown aborting after hipDeviceReset.

#### **rocSHMEM** (3.8.0)

##### Fixed
* Fixed tile RMA operations (`rocshmem_tile_put`, `rocshmem_tile_get`) to correctly apply non-zero `start_coord` values to both the source and destination tensors; previously only the remote tensor was offset, causing incorrect data movement when `start_coord != 0`.
  Fully-packed column-major tiles are now also classified as `Contiguous`, enabling a single flat memcpy instead of per-column copies.

##### Added
* GDA device code refactor (#7217)
  * Core GDA device code is now header-only.
  * Removed indirection and branching when compiled for a single GDA provider.
  * Significantly reduced size of device code GDA Queue Pair management objects.
  * Reduced latency of `rocshmem_quiet` for AMD Pensando IONIC provider.
    This also reduces the latency of blocking communication calls.
  * Communications using the GDA backend can now selectively choose at compilation time
    to reduce the internal GDA Queue Pair object correctness and safety constraints,
    improving application performance when such relaxations are known to be safe by the caller.
    These relaxations can be applied independently at each call site.
    This is currently an internal mechanism and is not exposed by the public rocSHMEM APIs.
* Added standalone signal APIs:
    * `rocshmem_signal_add` and `rocshmem_ctx_signal_add`
    * `rocshmem_signal_set` and `rocshmem_ctx_signal_set`
    * `rocshmem_signal_wait_until`

##### Deprecated
* Deprecated `rocshmem_signal_fetch_wg` and `rocshmem_signal_fetch_wave`.
  Use `rocshmem_signal_fetch` instead.

#### **rocSOLVER** (3.38.0)

##### Added


* Hessenberg reduction routines
    * GEHD2
    * GEHRD

* Support added for the gfx1250-strict architecture.

##### Changed

* Clarified in the API documentation that rocsolver_rfinfo is an input and output parameter as it could be modified within the functions.

##### Optimized

* Improved performance of expert eigensolvers SYEVDX/HEEVDX, SYGVDX/HEGVDX

##### Resolved issues

* Fixed failure of GETRS for batch_count > 64 * 1024

* Fixed failure of GETRF for batch_count > 64 * 1024 for certain sizes such as m = 3, n = 4

* Improved kernel launches to detect invalid launch configuration.

* Fixed sparse functions potentially not using the stream set in the handle.

#### **rocSPARSE** (5.1.0)
##### Added

* Added the `rocsparse_spsort` and `rocsparse_spsort_buffer_size` generic routines, which sort a sparse matrix in place or into a separate target matrix. COO matrices are sorted by row or by column, and CSR and CSC matrices have the indices within each row or column sorted. The values are reordered with the indices. Uniform strided batched matrices are supported, including broadcasting a single source matrix to every sample of the target matrix.
* Added the `rocsparse_spsort_descr_create`, `rocsparse_spsort_descr_destroy`, and `rocsparse_spsort_set_input` auxiliary routines, together with the `rocsparse_spsort_descr` type and the `rocsparse_spsort_alg`, `rocsparse_spsort_input`, and `rocsparse_spsort_stage` enums.
* Added support for the `gfx1250-strict` architecture.
* Added the batch support for `rocsparse_sptrsm`.
* Added the `rocsparse_dnmat_transpose` generic routine for dense matrix transposing.
* Added the `rocsparse_dnmat_copy_data` generic routine for dense matrix data copying.

##### Changed

* Replaced the soon-to-be-deprecated `__hip_atomic_*` builtins with the equivalent `__scoped_atomic_*` builtins.
* ROCmCMakeBuildTools 0.11.0 or later is now required. By default, CMake configuration fails when it isn't found instead of downloading rocm-cmake from GitHub. Configure with `-DALLOW_FETCH_DEPS=ON` to download rocm-cmake 0.14.0 from ROCm 7.2.4 instead. The deprecated `ROCM` package fallback was removed.
* GoogleTest 1.11.0 or later must be installed before building the client tests (`BUILD_CLIENTS_TESTS`). Alternatively, set `FETCHCONTENT_SOURCE_DIR_GOOGLETEST` to a googletest source tree, or configure with `-DALLOW_FETCH_DEPS=ON` to download GoogleTest 1.15.2. Both build GoogleTest statically and don't install it.
* `install.sh -d` no longer builds and installs GoogleTest from source.
* The 24 SuiteSparse test matrices are now stored with DVC in `clients/matrices`. Run `dvc pull -R projects/rocsparse/clients/matrices` before building the tests. In a sparse checkout, add the repository-root `.dvc` directory first with `git sparse-checkout add .dvc`. By default, CMake configuration fails when matrices are missing instead of downloading them from SuiteSparse. Configure with `-DALLOW_FETCH_DEPS=ON` to download them instead, which also applies to `ROCSPARSE_TEST_MIRROR`. The `--matrices-dir` and `--matrices-dir-install` options of `install.sh` and the `--matrices-dir` option of `rmake.py` still download missing matrices.

##### Removed

* Removed the GoogleTest superbuild in `deps/CMakeLists.txt`, together with `deps/external-gtest.cmake` and `cmake/get-cli-arguments.cmake`.

##### Optimized

* Improved `rocsparse_Xgemvi` performance, especially when `n` is greater than `m`.
* Improved the performance of the sparse triangular solves and the incomplete factorizations on gfx942 and gfx950. The kernels no longer write back and invalidate the L2 cache for every solved or factorized row. The row results and their completion flags are now exchanged with device-scope atomic stores and loads instead. This affects `rocsparse_spsv` and `rocsparse_sptrsv` (CSR, CSC, COO, and ELL formats), `rocsparse_spsm` and `rocsparse_sptrsm` (CSR, CSC, and COO formats), `rocsparse_Xcsrsv_solve`, `rocsparse_Xcsrsm_solve`, `rocsparse_Xcsrilu0`, `rocsparse_Xcsric0`, `rocsparse_spilu0`, `rocsparse_spic0`, and `rocsparse_spildlt0`. It also affects `rocsparse_Xbsrsv_solve` with block sizes up to 32, `rocsparse_Xbsrilu0` with block sizes up to 8, and `rocsparse_Xbsric0` with block sizes up to 16 and at most 128 blocks per block row. Larger block sizes and `rocsparse_Xbsrsm_solve` keep the previous synchronization, because their kernels reread row results too often for device-scope atomics to pay off.
* Improved the performance of the analysis of the CSR, CSC, COO, and BSR triangular solves and incomplete factorizations on gfx942 and gfx950. This affects the analysis stage of `rocsparse_spsv`, `rocsparse_sptrsv`, `rocsparse_spsm`, and `rocsparse_sptrsm`, and `rocsparse_Xcsrsv_analysis`, `rocsparse_Xcsrsm_analysis`, `rocsparse_Xbsrsv_analysis`, `rocsparse_Xbsrsm_analysis`, `rocsparse_Xcsrilu0_analysis`, `rocsparse_Xcsric0_analysis`, `rocsparse_Xbsrilu0_analysis`, `rocsparse_Xbsric0_analysis`, and the analysis stage of `rocsparse_spilu0`, `rocsparse_spic0`, and `rocsparse_spildlt0`. Each row previously updated the maximum row length with its own device-scope atomic on a single address, which serializes on these architectures. On gfx942 and gfx950, each row now updates a per-workgroup maximum with a shared-memory atomic, and the last row of the workgroup to finish issues a single device-scope atomic.
* Optimized `rocsparse_spgemm` (CSR and BSR), `rocsparse_csrgemm_nnz`, `rocsparse_Xcsrgemm`, `rocsparse_csrgemm_symbolic`, `rocsparse_Xcsrgemm_numeric`, `rocsparse_bsrgemm_nnzb`, and `rocsparse_Xbsrgemm` for rows of `C` whose hash table does not fit in shared memory. For BSR matrices with a block dimension greater than 1, only `rocsparse_bsrgemm_nnzb` is affected.

##### Resolved issues

* Fixed an out-of-bounds write in `rocsparse_coosort_by_row` and `rocsparse_coosort_by_column` when every row (or column) contains a single non-zero entry. The temporary workspace for the segment computation was one index too small, which could corrupt the segment offsets and produce incorrectly sorted results.
* Fixed `rocsparse_dense_to_sparse` (including the Blocked ELL format), `rocsparse_Xnnz`, `rocsparse_Xdense2csr`, `rocsparse_Xdense2csc`, `rocsparse_Xcsr2dense`, `rocsparse_Xcsc2dense`, `rocsparse_Xcoo2dense`, and `rocsparse_sparse_to_dense` for matrices with very many rows, columns, block rows, or non-zeros. The conversion kernels sized the launch grid from these counts without a grid-stride loop, so a large enough matrix exceeded the 32-bit kernel dispatch limit. The launch grids are now clamped to the device limit and the kernels use grid-stride loops. With 32-bit indices, `rocsparse_Xcsr2dense`, `rocsparse_Xcsc2dense`, and `rocsparse_Xcoo2dense` also computed the dense element count in 32 bits and skipped zero-filling a dense matrix with `2^31` or more elements. The count is now 64-bit, so dense matrices with fewer than `2^32` elements are zero-filled correctly.
* Fixed an overflow issue in `rocsparse_roti` and the generic `rocsparse_rot` routine. When using 64-bit indices and `nnz` >= `2^32`, a 32-bit element-index calculation could overflow, leaving some elements unrotated and causing low-index elements to be processed with incorrect data. The kernel now computes element indices in 64-bit arithmetic and uses a grid-stride loop with the launch grid clamped to the 32-bit work-item dispatch limit.
* Fixed an overflow issue in `rocsparse_Xsctr` and the generic `rocsparse_scatter` routine. When using 64-bit indices and `nnz` >= `2^32`, a 32-bit element-index calculation could overflow and prevent some elements from being scattered. The kernel now computes element indices in 64-bit arithmetic and uses a grid-stride loop with the launch grid clamped to the 32-bit work-item dispatch limit.
* Fixed an integer overflow in the ELL SpMV kernels used by `rocsparse_spmv` (ELL format) and the legacy `rocsparse_Xellmv` routines when operating on matrices with more than `2^32` rows and 64-bit index types. The overflow caused rows beyond the `2^32` boundary to be mapped to incorrect row indices, leaving some output values uncomputed. Row indices are now always computed using the matrix index type.
* Fixed an out-of-bounds read of the row map in the block-row incomplete factorization kernels used by `rocsparse_spic0` and `rocsparse_spilu0`. Eight of the nine BSR kernels indexed the map with the raw block index and never compared it against the block row count, so a launch grid rounded up past that count read outside the map allocation. All nine now bounds check, which is what the CSR kernels in the same family already did.
* Fixed `rocsparse_spic0` and `rocsparse_spilu0` silently truncating the launch grid for matrices with more block rows than the launch grid can hold. The block row count is taken from the sparse matrix descriptor as a 64-bit value and was narrowed into the 32-bit grid extent without a clamp. The binding grid.x limit is `(2^32 - 1) / blockDim.x` blocks, because `gridDim.x * blockDim.x` must stay below 2^32, rather than the 2147483647 that `maxGridSize[0]` reports. All BSR kernels now clamp the extent and recover the remaining block rows inside the kernel.
* Fixed an integer overflow in `rocsparse_Xgtsv_interleaved_batch`, which produced incorrect results when `batch_count * m` or `batch_stride * m` exceeded `INT32_MAX`. The interleaved layout makes every array `batch_stride * m` elements long, but the solver kernels computed the row offsets into those arrays in 32-bit arithmetic. The offsets are now computed in 64-bit arithmetic.
* Fixed kernel launches whose `grid.x * blockDim.x` exceeded the 32-bit work-item dispatch limit. CDNA rejects those launches (`hipErrorInvalidConfiguration` / `rocsparse_status_internal_error`). `rocsparse_axpby` (both the `beta` scaling of `y` and the `axpyi` update), `rocsparse_axpyi`, `rocsparse_roti` and `rocsparse_rot`, `rocsparse_scatter`, ELL SpMV (non-transposed and transposed), and COO AoS SpMV (atomic and transposed) now clamp `grid.x` to `(2^32 - 1) / blockDim.x` and cover the remainder with a grid-stride loop.
* Fixed an integer overflow in the binary search that maps nonzeros to rows in the nnz-split algorithms of `rocsparse_spmv` (CSR) and `rocsparse_spmm` (CSR and CSC). With 32-bit indices and matrices with more than `2^30` rows, the search midpoint `(left + right) / 2` could overflow, causing out-of-bounds reads or incorrect row assignments. The midpoint is now computed as `left + (right - left) / 2`.
* Fixed an invalid kernel launch in legacy `rocsparse_Xbsr2csr` and the 64-bit BSR-to-CSR conversion, where the 1024-thread kernels overflowed the 32-bit work-item grid limit at about 4.2 million block rows because the grid was not clamped.
* Fixed an integer overflow in the SELL SpMV kernels used by `rocsparse_v2_spmv`. With 32-bit indices, the row index of each thread was computed in 32-bit arithmetic, so for matrices with close to `2^31` rows the threads past the last row wrapped to a negative row index that passed the `row < m` check. Those threads could read past the end of the slice offsets, and the non-transposed kernel could write `y` at a negative index. The row index is now computed in 64-bit arithmetic. The transposed kernel for slice sizes up to 128 also no longer reads `x` for threads past the last row.
* Fixed an overflow issue in the COO atomic SpMV kernels used by `rocsparse_spmv` and `rocsparse_v2_spmv` for COO and COO AoS matrices. When using 64-bit indices and `nnz` >= `2^32`, a 32-bit element-ID calculation could overflow and prevent some nonzeros from being accumulated. The kernels now compute element IDs in 64-bit arithmetic.
* Fixed `install.sh --matrices-dir-install` and `--matrices-dir`, which reported success when a test matrix couldn't be extracted, downloaded or converted.
* Fixed the test matrix conversion with `ROCSPARSE_MTX_DIR`, which moved the `.mtx` files out of that directory and deleted them after conversion. They are now copied, and matrices missing from that directory are taken from the DVC archives.
* Fixed the test matrix conversion in address sanitizer builds, where the converter failed to load the shared address sanitizer runtime.
* Fixed the test matrix converter, which accepted a `.mtx` file with fewer entries than its header declares.

#### **rocThrust** (5.0.0)

##### Added
* rocThrust is now largely in feature parity with CCCL/thrust v3.0.3. If there isn't a libhipcxx corresponding to CCCL/libcudacxx version 3.0.3 or later, `thrust::tuple`, `thrust::pair` and `thrust::zip_iterator` will fall back to rocThrust 4.4.0 implementations.
 
##### Removed

* rocThrust compatibility with PyTorch v2.9 and v2.10 has been removed in this release.  Use PyTorch v2.11 or later.

#### **rocWMMA** (2.2.2)

##### Added

* Added support for the `gfx1250-strict` target.
* Added community samples for grouped MoE GEMM (`grouped_gemm_moe`), Stream-K GEMM (`simple_gemm_streamk`), and Flash Attention v2 (`simple_flash_attn_v2`).
* Added host tests for test-kernel generation.

##### Changed

* Reduced test build time through shared support objects, split GEMM instantiations, and lower template overhead.
* Consolidated documentation dependencies in `rocm-docs-core`.

##### Resolved issues

* Fixed HIP and OpenMP dependency discovery in the installed CMake package.
* Fixed host/device annotations on constexpr functions for compatibility with stricter Clang diagnostics.
* Fixed resource leaks in DLRM and hipRTC regression tests.

#### **RPP** (3.2.0)

##### Added

- Single-image processing support for 8 kernels (Brightness, Blend, Box Filter, Crop, Flip, Gaussian Filter, Median Filter, Resize Nearest Neighbor) to match performance with OpenCV
- Runtime backend selection parameter (`RppBackend executionBackend`) for all RPP tensor API functions
- Backend tracking in `rppHandle_t` to store backend type (HOST or HIP)
- `RPP_ERROR_HIP_LAUNCH` error type for reporting HIP kernel launch errors
- `rppGetStatusString()` to convert an `RppStatus` value into its enumerator name
- `rpp_BACKEND_TYPE` and `rpp_AUDIO_AUGMENTATIONS_SUPPORT` variables to RPP CMake package config

##### Changed

- All RPP tensor API functions now unified with a single function signature
- Updated all test suite calls to use unified API with backend parameter
- Enhanced layout validation for image augmentations within unified API
- Test suite dependency on TurboJPEG removed. Updated to use .rgb files as input.
- OpenCV and pandas are changed to optional dependency for the test suite
- CMakeLists.txt updated to remove batch PD references
- Updated test suite to use `rpp_BACKEND_TYPE` and `rpp_AUDIO_AUGMENTATIONS_SUPPORT` variables from RPP CMake package config instead of header parsing
- `find_package(rpp)` now automatically passes on public include directories to the target link interface

##### Removed

- BatchPD legacy support completely removed
- LEGACY_SUPPORT compilation flag and all code enclosed within it
- OpenCL backend support
- Batch PD test suite and installation
