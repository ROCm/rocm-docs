# Transition guide from legacy ROCm release stream

The [ROCm Core SDK](https://rocm.docs.amd.com/en/latest/index.html#rocm-core-sdk) is built on TheRock, AMD's new build system. The transition from the legacy ROCm release stream began with [ROCm Core SDK 7.14.0](https://rocm.docs.amd.com/en/docs-7.14.0/about/release-notes.html), the first production release. [ROCm 10.1.0](https://rocm.docs.amd.com/en/latest/about/release-notes.html) is the latest production release. For more on the transition, see [ROCm 7.14: TheRock Goes Production and Expands AMD's AI Software Platform](https://rocm.blogs.amd.com/ecosystems-and-partners/rocm-7.14-blog/README.html).

## Major changes

<table class="rocm-docs-table table">
  <thead>
    <tr>
      <th class="head">Feature</th>
      <th class="head">ROCm Core SDK</th>
      <th class="head">ROCm legacy</th>
      <th class="head">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Installation directory</td>
      <td><code class="docutils literal notranslate"><span class="pre">/opt/rocm/core-10.1</span></code></td>
      <td><code class="docutils literal notranslate"><span class="pre">/opt/rocm-7.2/</span></code></td>
      <td>To support additional release streams downstream of the ROCm Core SDK.</td>
    </tr>
    <tr>
      <td>Package names</td>
      <td><code class="docutils literal notranslate"><span class="pre">amdrocm-{component}</span></code></td>
      <td><code class="docutils literal notranslate"><span class="pre">rocm-[$component]</span></code> or <code class="docutils literal notranslate"><span class="pre">roc[$component]</span></code> or <code class="docutils literal notranslate"><span class="pre">hip[$component]</span></code></td>
      <td>Unique package prefix to avoid conflicts with upstream packages.</td>
    </tr>
    <tr>
      <td>Extras directory</td>
      <td><code class="docutils literal notranslate"><span class="pre">/opt/rocm/extras-10/</span></code></td>
      <td>N/A</td>
      <td>Shared install prefix scoped to each ROCm major version for projects built on the ROCm Core SDK.</td>
    </tr>
  </tbody>
</table>

## Paths and linking

For installations using your
Linux distribution's package manager, the `amdrocm` meta package configures
`update-alternatives` and provides backward-compatible symlinks for
`/opt/rocm/bin`, `/opt/rocm/lib`, and other `/opt/rocm/` directories.

## Installation formats

ROCm Core SDK is available in the following distribution formats. For step-by-step installation instructions, see {doc}`Install ROCm </install/rocm>`.

<table class="rocm-docs-table table">
  <thead>
    <tr>
      <th class="head">Format</th>
      <th class="head">Details</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>DEB / RPM packages</strong></td>
      <td>System-wide install through your package manager (<code>apt</code>, <code>dnf</code>, or <code>yum</code>). The most familiar install path on a managed Linux system. Available from <a href="https://repo.amd.com/">repo.amd.com</a>.</td>
    </tr>
    <tr>
      <td><strong>Tarball archives</strong></td>
      <td>
        Self-contained install that requires neither root nor a package manager, suited to HPC module systems and custom install locations. Archives follow the naming convention <code>therock-dist-linux-{FAMILY}-{VERSION}.tar.gz</code> (for example, <code>therock-dist-linux-gfx110X-all-{VERSION}.tar.gz</code>). For the <code>{FAMILY}</code> value for your GPU, see <a href="#architecture-specific-packages">Architecture-specific packages available in ROCm 10.1.0</a>.<br><br>
        Extract to any directory, then set <code>PATH</code>, <code>LD_LIBRARY_PATH</code>, and <code>ROCM_PATH</code> to point to the extracted location (default: <code>/opt/rocm/core</code>). Tarballs don't create symlinks or resolve dependencies.<br><br>
        Available from <a href="https://repo.amd.com/">repo.amd.com</a>.
      </td>
    </tr>
    <tr>
      <td><strong>Python wheels</strong></td>
      <td>
        Install ROCm libraries directly into a virtual environment with <code>pip</code>, for Python-only workflows. Use the ROCm Python package index:<br><br>
        <code>python -m pip install --index-url &lt;ROCm-package-index&gt; "rocm[libraries,devel]"</code><br><br>
        Framework wheels such as PyTorch, JAX, and vLLM are distributed separately.
      </td>
    </tr>
    <tr>
      <td><strong>Runfile installer</strong></td>
      <td>Single-file guided installer with interactive and silent modes. Supports a custom install directory, automatic GPU detection, and optional driver installation. Use it when you want neither a package manager nor manual tarball extraction.</td>
    </tr>
  </tbody>
</table>

### Choosing a format

| If you need… | Use |
|---|---|
| Automatic updates and dependency tracking on bare metal | **DEB / RPM packages** |
| A non-root install or multiple ROCm versions side by side | **Tarball** |
| Only the Python interface to GPU-accelerated libraries in a virtual environment | **Wheel** |
| A guided install without a package manager | **Runfile** |

## Software packages

ROCm Core SDK packages are more consolidated than the legacy ROCm release
stream. For example, hipBLAS and rocBLAS are now combined into one package,
`amdrocm-blas`. The table below lists the packages, their contents, and the
corresponding legacy packages.

(linux-packages-available-in-rocm-10-1-0)=

### Linux packages available in ROCm 10.1.0

<table class="rocm-docs-table table">
  <thead>
    <tr>
      <th class="head">ROCm Core SDK package</th>
      <th class="head">Package contents</th>
      <th class="head">ROCm legacy package</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>amdrocm-amdsmi</td>
      <td>amd-smi</td>
      <td>amd-smi-lib, rocm-smi-lib</td>
    </tr>
    <tr>
      <td>amdrocm-llvm</td>
      <td>amdclang++, hipcc, flang</td>
      <td>rocm-llvm, rocm-llvm-dev, Fortran compiler (included in rocm-llvm OpenMP runtime)</td>
    </tr>
    <tr>
      <td>amdrocm-runtime</td>
      <td>HIP, ROCR, runtime compilation</td>
      <td>hip-runtime-amd, rocm-hip-runtime, rocm-language-runtime, hsa-rocr, comgr</td>
    </tr>
    <tr>
      <td>amdrocm-fft</td>
      <td>rocFFT, hipFFT, hipFFTW</td>
      <td>rocfft, hipfft</td>
    </tr>
    <tr>
      <td>amdrocm-blas</td>
      <td>rocBLAS, hipBLAS, hipBLASLt, hipSPARSELt</td>
      <td>rocblas, hipblas, hipblaslt, hipsparselt</td>
    </tr>
    <tr>
      <td>amdrocm-sparse</td>
      <td>rocSPARSE, hipSPARSE</td>
      <td>rocsparse, hipsparse</td>
    </tr>
    <tr>
      <td>amdrocm-hipfile</td>
      <td>hipFile</td>
      <td>—</td>
    </tr>
    <tr>
      <td>amdrocm-solver</td>
      <td>rocSOLVER, hipSOLVER</td>
      <td>rocsolver, hipsolver, rocalution</td>
    </tr>
    <tr>
      <td>amdrocm-dnn</td>
      <td>hipDNN, MIOpen</td>
      <td>miopen-hip</td>
    </tr>
    <tr>
      <td>amdrocm-rand</td>
      <td>rocRAND, hipRAND</td>
      <td>rocrand, hiprand</td>
    </tr>
    <tr>
      <td>amdrocm-ccl-dev / devel</td>
      <td>rocPRIM, rocThrust, hipCUB, rocwmma, libhipcxx</td>
      <td>rocprim, rocthrust, hipcub, rocwmma</td>
    </tr>
    <tr>
      <td>amdrocm-profiler</td>
      <td>rocprofiler-systems, rocprofiler-compute, rocprofiler-sdk, roctracer</td>
      <td>rocprofiler, rocprofiler-compute, rocprofiler-systems, rocprofiler-sdk, roctracer</td>
    </tr>
    <tr>
      <td>amdrocm-profiler-base</td>
      <td>rocprofiler-sdk, roctracer</td>
      <td>rocprofiler-register, roctracer, hsa-amd-aqlprofile</td>
    </tr>
    <tr>
      <td>amdrocm-base</td>
      <td>rocminfo, rocm-core</td>
      <td>rocm-core, rocminfo, rocm-cmake, half</td>
    </tr>
    <tr>
      <td>amdrocm-ck</td>
      <td>Composable Kernel</td>
      <td>composablekernel</td>
    </tr>
    <tr>
      <td>amdrocm-debugger</td>
      <td>rocgdb, ROCdbgapi, ROCR Debug Agent</td>
      <td>rocm-gdb, rocm-dbgapi, rocm-debug-agent</td>
    </tr>
    <tr>
      <td>amdrocm-hipify</td>
      <td>HIPIFY</td>
      <td>hipify-clang</td>
    </tr>
    <tr>
      <td>amdrocm-opencl</td>
      <td>OpenCL runtime and ICD loader</td>
      <td>rocm-opencl-runtime, rocm-opencl, hip-opencl</td>
    </tr>
    <tr>
      <td>amdrocm-decode</td>
      <td>rocDecode</td>
      <td>rocdecode</td>
    </tr>
    <tr>
      <td>amdrocm-jpeg</td>
      <td>rocJPEG</td>
      <td>rocjpeg</td>
    </tr>
    <tr>
      <td>amdrocm-rpp</td>
      <td>ROCm Performance Primitives (RPP)</td>
      <td>rpp</td>
    </tr>
    <tr>
      <td>amdrocm-thread</td>
      <td>hipThreads</td>
      <td>—</td>
    </tr>
    <tr>
      <td>amdrocm-rccl</td>
      <td>rccl</td>
      <td>rccl</td>
    </tr>
    <tr>
      <td>amdrocm-rocshmem</td>
      <td>rocSHMEM</td>
      <td>rocshmem</td>
    </tr>
    <tr>
      <td>amdrocm-rdc</td>
      <td>ROCm Data Center Tool</td>
      <td>rdc</td>
    </tr>
    <tr>
      <td>amdrocm-sysdeps</td>
      <td>Bundled third-party dependencies (libdrm, libelf, numa, libVA)</td>
      <td>System dependencies</td>
    </tr>
  </tbody>
</table>

Packages are offered in the following variants:

- **For all supported GPUs:** Works across all GPUs supported by ROCm (for example, `apt install amdrocm-core-sdk10.1`).
- **For a specific GPU architecture:** Smaller install size, but requires you to know the GPU installed in your system (for example, `apt install amdrocm-core-sdk10.1-gfx110x`).

Installing all GPU architectures is not required. You can install packages for a specific architecture, multiple architectures side by side, or all supported GPU architectures.

When redistributing software built on the ROCm Core SDK (for example, in container images), choose the all-architecture variant for broad hardware support. If disk footprint is a concern, use a single-architecture variant instead.

(architecture-specific-packages)=

### Architecture-specific packages available in ROCm 10.1.0

Tarball archives use *family* names that differ from the deb/rpm package suffixes. The **Tarball family name** column maps each package suffix to its corresponding tarball family.

<table class="rocm-docs-table table">
  <thead>
    <tr>
      <th class="head">Architecture family</th>
      <th class="head">Package suffix</th>
      <th class="head">Tarball family name</th>
      <th class="head">Product name (not exhaustive)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>CDNA4</td>
      <td>-gfx950</td>
      <td>gfx950-dcgpu</td>
      <td>AMD Instinct MI355X / MI350X</td>
    </tr>
    <tr>
      <td>CDNA3</td>
      <td>-gfx942</td>
      <td>gfx94X-dcgpu</td>
      <td>AMD Instinct MI325X / MI300X / MI300A</td>
    </tr>
    <tr>
      <td>CDNA2</td>
      <td>-gfx90a</td>
      <td>gfx90a</td>
      <td>AMD Instinct MI250X / MI250 / MI210</td>
    </tr>
    <tr>
      <td>CDNA</td>
      <td>-gfx908</td>
      <td>—</td>
      <td>AMD Instinct MI100</td>
    </tr>
    <tr>
      <td>RDNA4</td>
      <td>-gfx1200<br>-gfx1201</td>
      <td>gfx120X-all</td>
      <td>AMD Radeon RX 9070 / AMD Radeon RX 9060 / AMD Radeon RX 9070 XT / AMD Radeon RX 9060 XT / AMD Radeon RX 9070 GRE / AMD Radeon AI PRO R9700S / AMD Radeon AI PRO R9700 / AMD Radeon AI PRO R9600D / AMD Radeon RX 9060 XT LP</td>
    </tr>
    <tr>
      <td>RDNA3.5</td>
      <td>-gfx1150<br>-gfx1151<br>-gfx1152<br>-gfx1153</td>
      <td>—</td>
      <td>AMD Ryzen AI 9 465 / AMD Ryzen AI 9 365 / AMD Ryzen AI 9 HX 475 / AMD Ryzen AI 9 HX 470 / AMD Ryzen AI 9 HX 375 / AMD Ryzen AI 9 HX 370 / AMD Ryzen AI 9 PRO 465 / AMD Ryzen AI 9 PRO HX 475 / AMD Ryzen AI 9 PRO HX 470 / AMD Ryzen AI 9 HX PRO 375 / AMD Ryzen AI 9 HX PRO 370 / AMD Ryzen AI Max 390 / AMD Ryzen AI Max 385 / AMD Ryzen AI Max+ 395 / AMD Ryzen AI Max+ 392 / AMD Ryzen AI Max+ 388 / AMD Ryzen AI Max PRO 390 / AMD Ryzen AI Max PRO 385 / AMD Ryzen AI Max PRO 380 / AMD Ryzen AI Max+ PRO 395 / AMD Ryzen AI 7 450 / AMD Ryzen AI 7 350 / AMD Ryzen AI 7 345 / AMD Ryzen AI 5 340 / AMD Ryzen AI 5 330 / AMD Ryzen AI 7 PRO 450 / AMD Ryzen AI 5 PRO 440 / AMD Ryzen AI 7 PRO 350 / AMD Ryzen AI 5 PRO 340</td>
    </tr>
    <tr>
      <td>RDNA3</td>
      <td>-gfx1100<br>-gfx1101<br>-gfx1102<br>-gfx1103</td>
      <td>gfx110X-all</td>
      <td>AMD Radeon RX 7700 / AMD Radeon RX 7600 / AMD Radeon PRO V710 / AMD Radeon PRO W7900 / AMD Radeon PRO W7800 / AMD Radeon PRO W7700 / AMD Radeon RX 7900 XT / AMD Radeon RX 7800 XT / AMD Radeon RX 7700 XT / AMD Radeon RX 7700 XE / AMD Radeon RX 7900 XTX / AMD Radeon RX 7900 GRE / AMD Radeon PRO W7800 48GB / AMD Radeon PRO W7900 Dual Slot</td>
    </tr>
    <tr>
      <td>RDNA2</td>
      <td>-gfx1030</td>
      <td>—</td>
      <td>AMD Radeon PRO V620 / AMD Radeon PRO W6800</td>
    </tr>
  </tbody>
</table>

## ROCm Core SDK component changes (moved or removed)

### Planned for future releases

- ROCm-Extras: hipfort, rocPyDecode, rocAL, MIVisionX

### Moved to ROCm-Extras

- ROCm Validation Suite
- TransferBench
- ROCm Optiq

### Moved to Standalone/ONNX

- ONNX runtime

### Removed

- [ROCm SMI](https://rocm.docs.amd.com/en/latest/about/release-notes.html#rocm-smi-deprecation) (replaced by AMD SMI)
- ROCm Bandwidth Test (end-of-life as of the TheRock-based ROCm 7.14.0 release; use TransferBench or RVS instead)

## Notable package relocations

- rocMLIR (now included in MIGraphX)
- HIPCC (now included in `amdrocm-llvm`)
- FLANG (now included in `amdrocm-llvm`)
- ROCm CMake (now in `amdrocm-base`)
- ROCTracer (now in `amdrocm-profiler-base`)
- ROCProfiler (functionality in `amdrocm-profiler`)

## Components available in the ROCm Core SDK, ROCm-Extras, and Standalone/ONNX

<table class="rocm-docs-table table">
  <thead>
    <tr>
      <th class="head"></th>
      <th class="head">Category</th>
      <th class="head">Present</th>
      <th class="head">Absent/Moved</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="8" class="stub" style="vertical-align: middle"><strong>ROCm Core SDK</strong></td>
      <td>Math and compute libraries</td>
      <td>CK, hipBLAS, hipBLASLt, hipCUB, hipFFT, hipRAND, hipSOLVER, hipSPARSE/SPARSELt, libhipcxx, MIOpen, rocBLAS, rocFFT, rocRAND, rocSOLVER, rocSPARSE, rocPRIM, rocThrust, rocWMMA</td>
      <td>hipfort, rocALUTION</td>
    </tr>
    <tr>
      <td>Communication libraries</td>
      <td>RCCL, rocSHMEM</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Media and vision libraries</td>
      <td>rocDecode, rocJPEG, ROCm Performance Primitives (RPP)</td>
      <td>rocPyDecode, rocAL, MIVisionX, MIGraphX, CK (moved to math and compute)</td>
    </tr>
    <tr>
      <td>Storage libraries</td>
      <td>hipFile</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Threading libraries</td>
      <td>hipThreads</td>
      <td>—</td>
    </tr>
    <tr>
      <td>Runtime, compilers, build tools</td>
      <td>HIP, HIPIFY, LLVM</td>
      <td>HIPCC (moved to <code class="docutils literal notranslate"><span class="pre">amdrocm-llvm</span></code>), FLANG (moved to <code class="docutils literal notranslate"><span class="pre">amdrocm-llvm</span></code>), ROCm CMake (moved to <code class="docutils literal notranslate"><span class="pre">amdrocm-base</span></code>)</td>
    </tr>
    <tr>
      <td>Profiling and debugging tools</td>
      <td>ROCm Compute Profiler, ROCm Systems Profiler, ROCprofiler-SDK, ROCdbgapi, ROCm Debugger, ROCR Debug Agent</td>
      <td>ROCTracer (moved to <code class="docutils literal notranslate"><span class="pre">amdrocm-profiler-base</span></code>), ROCProfiler (functionality moved to <code class="docutils literal notranslate"><span class="pre">amdrocm-profiler</span></code>)</td>
    </tr>
    <tr>
      <td>Control and monitoring tools</td>
      <td>AMD SMI, ROCm Data Center Tool, rocminfo</td>
      <td>ROCm SMI (removed), ROCm Validation Suite, ROCm Bandwidth Test (removed)</td>
    </tr>
    <tr>
      <td class="stub" style="vertical-align: middle"><strong>ROCm Extras</strong></td>
      <td>—</td>
      <td>ROCm Validation Suite, TransferBench, ROCm Optiq</td>
      <td>—</td>
    </tr>
    <tr>
      <td class="stub" style="vertical-align: middle"><strong>Standalone/ONNX</strong></td>
      <td>—</td>
      <td>rocMLIR, ONNX runtime</td>
      <td>—</td>
    </tr>
  </tbody>
</table>
