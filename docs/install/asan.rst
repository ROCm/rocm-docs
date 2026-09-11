.. meta::
   :description: How to install AMD ROCm with AddressSanitizer (ASAN) instrumentation for AMD Instinct GPUs
   :keywords: linux, install, download, setup, asan, addresssanitizer, sanitizer, ubuntu, debian, red, hat, rhel, oracle, rocky, suse, sles, instinct, mi300a, mi300x, mi325x, mi350p, mi350x, mi355x, gfx942, gfx950, package, manager, tarball

:selector-toc2: Installation environment
:selector-toc2-icon: fa-solid fa-computer

***************************
Install AMD ROCm with ASAN
***************************

.. _rocm-asan-install:

ASAN (AddressSanitizer) builds of ROCm are available for specific AMD Instinct
GPU architectures and can be installed using the package manager or a tarball.

.. important::

   - ASAN builds are only available for ``gfx942`` and ``gfx950`` architectures,
     plus a multiarch build (``all``, both gfx942 and gfx950).
   - ASAN rpm and debian packages use the naming convention ``amdrocm-asan10.0`` or
     ``amdrocm-asan10.0-gfxXYZ``.
   - ASAN rpm and debian packages install to ``/opt/rocm/core-asan-10.0``, separate from
     regular ROCm installations at ``/opt/rocm/core-10.0``.
   - ASAN packages are approximately 4× larger than a standard ROCm installation due to debug symbols and ASAN instrumentation.

----

.. _rocm-asan-install-selector:

Use the following selector to choose your GPU architecture, operating system,
and installation method.

.. selector:: Device family
   :key: fam

   .. selector-option:: All
      :value: all
      :width: 50%

   .. selector-option:: AMD Instinct™
      :value: instinct
      :width: 50%
      :toc-label: AMD Instinct

.. selected:: fam=instinct

   .. selector-dropdown:: Instinct GPU
      :key: gpu
      :sort: desc

      .. selector-option:: AMD Instinct MI355X (gfx950)
         :value: amd-instinct-mi355x gfx=gfx950

      .. selector-option:: AMD Instinct MI350X (gfx950)
         :value: amd-instinct-mi350x gfx=gfx950

      .. selector-option:: AMD Instinct MI350P (gfx950)
         :value: amd-instinct-mi350p gfx=gfx950

      .. selector-option:: AMD Instinct MI325X (gfx942)
         :value: amd-instinct-mi325x gfx=gfx942

      .. selector-option:: AMD Instinct MI300X (gfx942)
         :value: amd-instinct-mi300x gfx=gfx942

      .. selector-option:: AMD Instinct MI300A (gfx942)
         :value: amd-instinct-mi300a gfx=gfx942

.. selector:: Operating system
   :key: os

   .. selector-option:: Ubuntu
      :value: ubuntu
      :width: 4

   .. selector-option:: Debian
      :value: debian
      :width: 4

   .. selector-option:: RHEL
      :value: rhel
      :width: 4
      :toc-label: Red Hat Enterprise Linux

   .. selector-option:: Oracle Linux
      :value: oracle-linux
      :width: 4

   .. selector-option:: Rocky Linux
      :value: rocky-linux
      :width: 4

   .. selector-option:: SLES
      :value: sles
      :width: 4
      :toc-label: SUSE Linux Enterprise Server

.. selected:: os=ubuntu

   .. selector:: Ubuntu version
      :key: ubuntu-ver

      .. selector-option:: 26.04
         :value: 26.04
         :width: 4

      .. selector-option:: 24.04
         :value: 24.04
         :width: 4

      .. selector-option:: 22.04
         :value: 22.04
         :width: 4

.. selected:: os=debian

   .. selector:: Debian version
      :key: debian-ver

      .. selector-option:: 13
         :value: 13
         :width: 6

      .. selector-option:: 12
         :value: 12
         :width: 6

.. selected:: os=rhel

   .. selector:: RHEL version
      :key: rhel-ver

      .. selector-option:: 10
         :value: 10
         :width: 4

      .. selector-option:: 9
         :value: 9
         :width: 4

      .. selector-option:: 8
         :value: 8
         :width: 4

.. selected:: os=oracle-linux

   .. selector:: Oracle Linux version
      :key: oracle-linux-ver

      .. selector-option:: 10
         :value: 10
         :width: 4

      .. selector-option:: 9
         :value: 9
         :width: 4

      .. selector-option:: 8
         :value: 8
         :width: 4

.. selected:: os=rocky-linux

   .. selector:: Rocky Linux version
      :key: rocky-linux-ver

      .. selector-option:: 9
         :value: 9
         :width: 12

.. selected:: os=sles

   .. selector:: SLES version
      :key: sles-ver

      .. selector-option:: 16
         :value: 16
         :width: 6

      .. selector-option:: 15
         :value: 15
         :width: 6

.. selector:: Installation method
   :show-cond: os=ubuntu os=debian
   :key: i

   .. selector-option:: apt
      :value: pkgman
      :width: 6

   .. selector-option:: Tarball
      :value: tar
      :width: 6

.. selector:: Installation method
   :show-cond: os=rhel os=oracle-linux os=rocky-linux
   :key: i

   .. selector-option:: dnf
      :value: pkgman
      :width: 6

   .. selector-option:: Tarball
      :value: tar
      :width: 6

.. selector:: Installation method
   :show-cond: os=sles
   :key: i

   .. selector-option:: zypper
      :value: pkgman
      :width: 6

   .. selector-option:: Tarball
      :value: tar
      :width: 6

----

.. _rocm-asan-install-prerequisites:

Prerequisites
=============

Before installing ROCm ASAN, make sure your system meets the ROCm hardware, software, and driver requirements. For more information, see :doc:`Install AMD ROCm <rocm>`. 

For system requirements and support information, see the :doc:`Compatibility matrix </compatibility/compatibility-matrix>`.

.. _rocm-asan-install-rocm:

Install ROCm ASAN
=================

Use the following instructions to install ROCm ASAN packages on your system.

.. ========================================================== PACKAGE MANAGER ==

.. selected:: i=pkgman
   :heading: Register ROCm repositories
   :heading-level: 3

   Complete the ROCm installation prerequisites to install dependencies and configure GPU access permissions before proceeding.

   .. selected:: os=ubuntu

      .. selected:: ubuntu-ver=26.04

         .. code-block:: bash

            # Download and install GPG key
            sudo mkdir --parents --mode=0755 /etc/apt/keyrings
            wget https://stable.repo.amd.com/rocm/gpg/packages.gpg -O - | \
                gpg --dearmor | sudo tee /etc/apt/keyrings/amdrocm.gpg > /dev/null

            sudo tee /etc/apt/sources.list.d/amdrocm-stable.sources << EOF
            X-Repo-Id: amdrocm-stable
            Types: deb
            URIs: https://stable.repo.amd.com/rocm/core/packages-asan/ubuntu2604/
            Suites: stable
            Components: main
            Architectures: amd64
            Signed-By: /etc/apt/keyrings/amdrocm.gpg
            Enabled: yes
            EOF

            sudo apt update

      .. selected:: ubuntu-ver=24.04

         .. code-block:: bash

            # Download and install GPG key
            sudo mkdir --parents --mode=0755 /etc/apt/keyrings
            wget https://stable.repo.amd.com/rocm/gpg/packages.gpg -O - | \
                gpg --dearmor | sudo tee /etc/apt/keyrings/amdrocm.gpg > /dev/null

            sudo tee /etc/apt/sources.list.d/amdrocm-stable.sources << EOF
            X-Repo-Id: amdrocm-stable
            Types: deb
            URIs: https://stable.repo.amd.com/rocm/core/packages-asan/ubuntu2404/
            Suites: stable
            Components: main
            Architectures: amd64
            Signed-By: /etc/apt/keyrings/amdrocm.gpg
            Enabled: yes
            EOF

            sudo apt update

      .. selected:: ubuntu-ver=22.04

         .. code-block:: bash

            # Download and install GPG key
            sudo mkdir --parents --mode=0755 /etc/apt/keyrings
            wget https://stable.repo.amd.com/rocm/gpg/packages.gpg -O - | \
                gpg --dearmor | sudo tee /etc/apt/keyrings/amdrocm.gpg > /dev/null

            sudo tee /etc/apt/sources.list.d/amdrocm-stable.sources << EOF
            X-Repo-Id: amdrocm-stable
            Types: deb
            URIs: https://stable.repo.amd.com/rocm/core/packages-asan/ubuntu2204/
            Suites: stable
            Components: main
            Architectures: amd64
            Signed-By: /etc/apt/keyrings/amdrocm.gpg
            Enabled: yes
            EOF

            sudo apt update

   .. selected:: os=debian

      .. selected:: debian-ver=13

         .. code-block:: bash

            # Download and install GPG key
            sudo mkdir --parents --mode=0755 /etc/apt/keyrings
            wget https://stable.repo.amd.com/rocm/gpg/packages.gpg -O - | \
                gpg --dearmor | sudo tee /etc/apt/keyrings/amdrocm.gpg > /dev/null

            sudo tee /etc/apt/sources.list.d/amdrocm-stable.sources << EOF
            X-Repo-Id: amdrocm-stable
            Types: deb
            URIs: https://stable.repo.amd.com/rocm/core/packages-asan/debian13/
            Suites: stable
            Components: main
            Architectures: amd64
            Signed-By: /etc/apt/keyrings/amdrocm.gpg
            Enabled: yes
            EOF

            sudo apt update

      .. selected:: debian-ver=12

         .. code-block:: bash

            # Download and install GPG key
            sudo mkdir --parents --mode=0755 /etc/apt/keyrings
            wget https://stable.repo.amd.com/rocm/gpg/packages.gpg -O - | \
                gpg --dearmor | sudo tee /etc/apt/keyrings/amdrocm.gpg > /dev/null

            sudo tee /etc/apt/sources.list.d/amdrocm-stable.sources << EOF
            X-Repo-Id: amdrocm-stable
            Types: deb
            URIs: https://stable.repo.amd.com/rocm/core/packages-asan/debian12/
            Suites: stable
            Components: main
            Architectures: amd64
            Signed-By: /etc/apt/keyrings/amdrocm.gpg
            Enabled: yes
            EOF

            sudo apt update

   .. selected:: os=rhel

      .. selected:: rhel-ver=10

         .. code-block:: bash

            sudo tee /etc/yum.repos.d/amdrocm-stable.repo <<EOF
            [amdrocm-stable]
            name=ROCm 10.0.0
            baseurl=https://stable.repo.amd.com/rocm/core/packages-asan/rhel10/x86_64
            enabled=1
            gpgcheck=1
            gpgkey=https://stable.repo.amd.com/rocm/gpg/packages.gpg
            EOF

            sudo dnf clean all

      .. selected:: rhel-ver=9

         .. code-block:: bash

            sudo tee /etc/yum.repos.d/amdrocm-stable.repo <<EOF
            [amdrocm-stable]
            name=ROCm 10.0.0
            baseurl=https://stable.repo.amd.com/rocm/core/packages-asan/rhel9/x86_64
            enabled=1
            gpgcheck=1
            gpgkey=https://stable.repo.amd.com/rocm/gpg/packages.gpg
            EOF

            sudo dnf clean all

      .. selected:: rhel-ver=8

         .. code-block:: bash

            sudo tee /etc/yum.repos.d/amdrocm-stable.repo <<EOF
            [amdrocm-stable]
            name=ROCm 10.0.0
            baseurl=https://stable.repo.amd.com/rocm/core/packages-asan/rhel8/x86_64
            enabled=1
            gpgcheck=1
            gpgkey=https://stable.repo.amd.com/rocm/gpg/packages.gpg
            EOF

            sudo dnf clean all

   .. selected:: os=oracle-linux

      .. selected:: oracle-linux-ver=10

         .. code-block:: bash

            sudo tee /etc/yum.repos.d/amdrocm-stable.repo <<EOF
            [amdrocm-stable]
            name=ROCm 10.0.0
            baseurl=https://stable.repo.amd.com/rocm/core/packages-asan/rhel10/x86_64
            enabled=1
            gpgcheck=1
            gpgkey=https://stable.repo.amd.com/rocm/gpg/packages.gpg
            EOF

            sudo dnf clean all

      .. selected:: oracle-linux-ver=9

         .. code-block:: bash

            sudo tee /etc/yum.repos.d/amdrocm-stable.repo <<EOF
            [amdrocm-stable]
            name=ROCm 10.0.0
            baseurl=https://stable.repo.amd.com/rocm/core/packages-asan/rhel9/x86_64
            enabled=1
            gpgcheck=1
            gpgkey=https://stable.repo.amd.com/rocm/gpg/packages.gpg
            EOF

            sudo dnf clean all

      .. selected:: oracle-linux-ver=8

         .. code-block:: bash

            sudo tee /etc/yum.repos.d/amdrocm-stable.repo <<EOF
            [amdrocm-stable]
            name=ROCm 10.0.0
            baseurl=https://stable.repo.amd.com/rocm/core/packages-asan/rhel8/x86_64
            enabled=1
            gpgcheck=1
            gpgkey=https://stable.repo.amd.com/rocm/gpg/packages.gpg
            EOF

            sudo dnf clean all

   .. selected:: os=rocky-linux

      .. code-block:: bash

         sudo tee /etc/yum.repos.d/amdrocm-stable.repo <<EOF
         [amdrocm-stable]
         name=ROCm 10.0.0
         baseurl=https://stable.repo.amd.com/rocm/core/packages-asan/rhel9/x86_64
         enabled=1
         gpgcheck=1
         gpgkey=https://stable.repo.amd.com/rocm/gpg/packages.gpg
         EOF

         sudo dnf clean all

   .. selected:: os=sles

      .. selected:: sles-ver=16

         .. code-block:: bash

            sudo tee /etc/zypp/repos.d/amdrocm-stable.repo <<EOF
            [amdrocm-stable]
            name=ROCm 10.0.0
            baseurl=https://stable.repo.amd.com/rocm/core/packages-asan/sles16/x86_64
            enabled=1
            gpgcheck=1
            gpgkey=https://stable.repo.amd.com/rocm/gpg/packages.gpg
            EOF

            sudo zypper --gpg-auto-import-keys refresh

      .. selected:: sles-ver=15

         .. code-block:: bash

            sudo tee /etc/zypp/repos.d/amdrocm-stable.repo <<EOF
            [amdrocm-stable]
            name=ROCm 10.0.0
            baseurl=https://stable.repo.amd.com/rocm/core/packages-asan/sles15/x86_64
            enabled=1
            gpgcheck=1
            gpgkey=https://stable.repo.amd.com/rocm/gpg/packages.gpg
            EOF

            sudo zypper --gpg-auto-import-keys refresh

.. selected:: i=pkgman
   :heading: Install ROCm ASAN packages
   :heading-level: 3

   After registering the repository, install ROCm ASAN packages for your target
   GPU architecture. See :ref:`ROCm ASAN meta packages
   <rocm-asan-install-meta-packages>` for additional installation options.

   .. selected:: os=ubuntu os=debian

      .. selected:: fam=all

         .. code-block:: bash

            sudo apt install amdrocm-asan10.0

      .. selected:: gfx=gfx942

         .. code-block:: bash

            sudo apt install amdrocm-asan10.0-gfx942

      .. selected:: gfx=gfx950

         .. code-block:: bash

            sudo apt install amdrocm-asan10.0-gfx950

   .. selected:: os=rhel os=oracle-linux os=rocky-linux

      .. selected:: fam=all

         .. code-block:: bash

            sudo dnf install amdrocm-asan10.0

      .. selected:: gfx=gfx942

         .. code-block:: bash

            sudo dnf install amdrocm-asan10.0-gfx942

      .. selected:: gfx=gfx950

         .. code-block:: bash

            sudo dnf install amdrocm-asan10.0-gfx950

   .. selected:: os=sles

      .. selected:: fam=all

         .. code-block:: bash

            sudo zypper install amdrocm-asan10.0

      .. selected:: gfx=gfx942

         .. code-block:: bash

            sudo zypper install amdrocm-asan10.0-gfx942

      .. selected:: gfx=gfx950

         .. code-block:: bash

            sudo zypper install amdrocm-asan10.0-gfx950

.. ============================================================ META PACKAGES ==

.. selected:: i=pkgman
   :heading: ROCm ASAN meta packages
   :heading-level: 4

   .. _rocm-asan-install-meta-packages:

   Meta packages group related components and dependencies together, allowing
   you to install only what is necessary for your use case. The following table
   describes available ROCm ASAN meta packages:

   .. selected:: os=rhel

      .. selected:: rhel-ver=9

         .. list-table::
            :header-rows: 1
            :widths: 25 20 30 25

            * - Meta package name
              - Use case
              - Description
              - Contents
            * - ``amdrocm-asan10.0``
              - ROCm Base
              - Core runtime environment. Install this to run ROCm applications with
                ASAN instrumentation.
              - Runtimes, libraries, system control and monitoring tools, and other
                essential components with ASAN.
            * - ``amdrocm-core-devel-asan10.0``
              - ROCm Developer Essentials
              - Development environment. Install this to build ROCm applications with
                ASAN support.
              - ``amdrocm-asan10.0`` plus compilers, CMake configurations, static
                library files, and headers with ASAN.
            * - ``amdrocm-developer-tools-asan10.0``
              - ROCm Profiler
              - Install this to profile and optimize ROCm applications with ASAN.
              - Profilers and related tools with ASAN instrumentation.
            * - ``amdrocm-opencl-asan10.0``
              - ROCm OpenCL
              - Install this to run OpenCL applications on ROCm with ASAN.
              - Components needed to run OpenCL with ASAN.
            * - ``amdrocm-core-sdk-asan10.0``
              - ROCm Full Suite
              - Install this if you need everything with ASAN.
              - The complete ROCm Core SDK including runtimes, compilers, development
                tools, and dependencies with ASAN.

      .. selected:: rhel-ver=10 rhel-ver=8

         .. list-table::
            :header-rows: 1
            :widths: 25 20 30 25

            * - Meta package name
              - Use case
              - Description
              - Contents
            * - ``amdrocm-asan10.0``
              - ROCm Base
              - Core runtime environment. Install this to run ROCm applications with
                ASAN instrumentation.
              - Runtimes, libraries, system control and monitoring tools, and other
                essential components with ASAN.
            * - ``amdrocm-core-dev-asan10.0``
              - ROCm Developer Essentials
              - Development environment. Install this to build ROCm applications with
                ASAN support.
              - ``amdrocm-asan10.0`` plus compilers, CMake configurations, static
                library files, and headers with ASAN.
            * - ``amdrocm-developer-tools-asan10.0``
              - ROCm Profiler
              - Install this to profile and optimize ROCm applications with ASAN.
              - Profilers and related tools with ASAN instrumentation.
            * - ``amdrocm-opencl-asan10.0``
              - ROCm OpenCL
              - Install this to run OpenCL applications on ROCm with ASAN.
              - Components needed to run OpenCL with ASAN.
            * - ``amdrocm-core-sdk-asan10.0``
              - ROCm Full Suite
              - Install this if you need everything with ASAN.
              - The complete ROCm Core SDK including runtimes, compilers, development
                tools, and dependencies with ASAN.

   .. selected:: os=sles

      .. selected:: sles-ver=16

         .. list-table::
            :header-rows: 1
            :widths: 25 20 30 25

            * - Meta package name
              - Use case
              - Description
              - Contents
            * - ``amdrocm-asan10.0``
              - ROCm Base
              - Core runtime environment. Install this to run ROCm applications with
                ASAN instrumentation.
              - Runtimes, libraries, system control and monitoring tools, and other
                essential components with ASAN.
            * - ``amdrocm-core-devel-asan10.0``
              - ROCm Developer Essentials
              - Development environment. Install this to build ROCm applications with
                ASAN support.
              - ``amdrocm-asan10.0`` plus compilers, CMake configurations, static
                library files, and headers with ASAN.
            * - ``amdrocm-developer-tools-asan10.0``
              - ROCm Profiler
              - Install this to profile and optimize ROCm applications with ASAN.
              - Profilers and related tools with ASAN instrumentation.
            * - ``amdrocm-opencl-asan10.0``
              - ROCm OpenCL
              - Install this to run OpenCL applications on ROCm with ASAN.
              - Components needed to run OpenCL with ASAN.
            * - ``amdrocm-core-sdk-asan10.0``
              - ROCm Full Suite
              - Install this if you need everything with ASAN.
              - The complete ROCm Core SDK including runtimes, compilers, development
                tools, and dependencies with ASAN.

      .. selected:: sles-ver=15

         .. list-table::
            :header-rows: 1
            :widths: 25 20 30 25

            * - Meta package name
              - Use case
              - Description
              - Contents
            * - ``amdrocm-asan10.0``
              - ROCm Base
              - Core runtime environment. Install this to run ROCm applications with
                ASAN instrumentation.
              - Runtimes, libraries, system control and monitoring tools, and other
                essential components with ASAN.
            * - ``amdrocm-core-dev-asan10.0``
              - ROCm Developer Essentials
              - Development environment. Install this to build ROCm applications with
                ASAN support.
              - ``amdrocm-asan10.0`` plus compilers, CMake configurations, static
                library files, and headers with ASAN.
            * - ``amdrocm-developer-tools-asan10.0``
              - ROCm Profiler
              - Install this to profile and optimize ROCm applications with ASAN.
              - Profilers and related tools with ASAN instrumentation.
            * - ``amdrocm-opencl-asan10.0``
              - ROCm OpenCL
              - Install this to run OpenCL applications on ROCm with ASAN.
              - Components needed to run OpenCL with ASAN.
            * - ``amdrocm-core-sdk-asan10.0``
              - ROCm Full Suite
              - Install this if you need everything with ASAN.
              - The complete ROCm Core SDK including runtimes, compilers, development
                tools, and dependencies with ASAN.

   .. selected:: os=ubuntu os=debian os=oracle-linux os=rocky-linux

      .. list-table::
         :header-rows: 1
         :widths: 25 20 30 25

         * - Meta package name
           - Use case
           - Description
           - Contents
         * - ``amdrocm-asan10.0``
           - ROCm Base
           - Core runtime environment. Install this to run ROCm applications with
             ASAN instrumentation.
           - Runtimes, libraries, system control and monitoring tools, and other
             essential components with ASAN.
         * - ``amdrocm-core-dev-asan10.0``
           - ROCm Developer Essentials
           - Development environment. Install this to build ROCm applications with
             ASAN support.
           - ``amdrocm-asan10.0`` plus compilers, CMake configurations, static
             library files, and headers with ASAN.
         * - ``amdrocm-developer-tools-asan10.0``
           - ROCm Profiler
           - Install this to profile and optimize ROCm applications with ASAN.
           - Profilers and related tools with ASAN instrumentation.
         * - ``amdrocm-opencl-asan10.0``
           - ROCm OpenCL
           - Install this to run OpenCL applications on ROCm with ASAN.
           - Components needed to run OpenCL with ASAN.
         * - ``amdrocm-core-sdk-asan10.0``
           - ROCm Full Suite
           - Install this if you need everything with ASAN.
           - The complete ROCm Core SDK including runtimes, compilers, development
             tools, and dependencies with ASAN.

   .. note::

      All ASAN meta packages follow the naming convention
      ``amdrocm-<component>-asan10.0`` or ``amdrocm-<component>-asan10.0-gfx<XYZ>``
      for architecture-specific builds.

.. ================================================================== TARBALL ==

.. selected:: i=tar
   :heading: Create the installation directory
   :heading-level: 3

   Run the following command in your desired location to create your
   installation directory:

   .. code-block:: bash

      mkdir therock-tarball-asan && cd therock-tarball-asan

   .. important::

      Subsequent commands assume you're working with the ``therock-tarball-asan``
      directory. If you choose a different directory name, adjust the commands
      accordingly.

.. selected:: i=tar
   :heading: Download and unpack the tarball
   :heading-level: 3

   Use the following commands to download and untar the ROCm ASAN tarball for
   your target GPU architecture.

   .. selected:: fam=all

      .. code-block:: bash

         wget https://stable.repo.amd.com/rocm/core/tarball-asan/therock-dist-linux-multiarch-10.0.0.tar.gz
         mkdir install
         tar -xf *.tar.gz -C install

   .. selected:: gfx=gfx942

      .. code-block:: bash

         wget https://stable.repo.amd.com/rocm/core/tarball-asan/therock-dist-linux-gfx94X-dcgpu-10.0.0.tar.gz
         mkdir install
         tar -xf *.tar.gz -C install

   .. selected:: gfx=gfx950

      .. code-block:: bash

         wget https://stable.repo.amd.com/rocm/core/tarball-asan/therock-dist-linux-gfx950-dcgpu-10.0.0.tar.gz
         mkdir install
         tar -xf *.tar.gz -C install

.. _rocm-asan-install-post:

Post-installation
=================

After installing ROCm ASAN 10.0.0, complete these post-installation steps to
configure your system and validate the installation.

.. selected:: i=pkgman i=tar
   :heading: Configure your environment
   :heading-level: 3

   Configure environment variables so that ROCm ASAN libraries and tools are
   available either to all users on the system or only to your user account.

   .. tab-set::

      .. tab-item:: System-wide setup

         .. selected:: i=tar

            Create a profile script so that all users inherit the ROCm ASAN
            environment variables when they start a shell session. Make sure
            you're in the ``therock-tarball-asan`` directory before proceeding.

            .. code-block:: bash

               # Configure ROCM_ASAN_PATH to the ASan install tree (tarball extract path)
               ROCM_ASAN_INSTALL_PATH=$(pwd)/install
               sudo tee /etc/profile.d/set-rocm-asan-env.sh << EOF
               # ROCm ASan Configuration
               export ROCM_ASAN_PATH=$ROCM_ASAN_INSTALL_PATH

               # Add ROCm bin to PATH
               export PATH=\$PATH:\$ROCM_ASAN_PATH/bin

               # Enable XNACK for device-side GPU instrumentation
               # Without this, only host-side (CPU) errors will be detected
               export HSA_XNACK=1

               # Locate ASan runtime directory and append instrumented library directories
               ASAN_LIB_PATH=\$(amdclang --print-file-name=libclang_rt.asan-x86_64.so 2>/dev/null || echo "")
               export LD_LIBRARY_PATH="\$LD_LIBRARY_PATH:\${ASAN_LIB_PATH%/*}:\${ROCM_ASAN_PATH}/lib:\${ROCM_ASAN_PATH}/lib/llvm/lib:\${ROCM_ASAN_PATH}/lib/rocm_sysdeps/lib/"
               EOF
               sudo chmod +x /etc/profile.d/set-rocm-asan-env.sh
               source /etc/profile.d/set-rocm-asan-env.sh

         .. selected:: i=pkgman

            Create a profile script so that all users inherit the ROCm ASAN
            environment variables when they start a shell session.

            .. code-block:: bash

               # Configure ROCM_ASAN_PATH to the ASan install tree
               sudo tee /etc/profile.d/set-rocm-asan-env.sh << 'EOF'
               # ROCm ASan Configuration
               export ROCM_ASAN_PATH=/opt/rocm/core-asan-10.0

               # Enable XNACK for device-side GPU instrumentation
               # Without this, only host-side (CPU) errors will be detected
               export HSA_XNACK=1

               # Locate ASan runtime directory and append instrumented library directories
               ASAN_LIB_PATH=$(amdclang --print-file-name=libclang_rt.asan-x86_64.so 2>/dev/null || echo "")
               export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:${ASAN_LIB_PATH%/*}:${ROCM_ASAN_PATH}/lib:${ROCM_ASAN_PATH}/lib/llvm/lib:${ROCM_ASAN_PATH}/lib/rocm_sysdeps/lib/"
               EOF
               sudo chmod +x /etc/profile.d/set-rocm-asan-env.sh
               source /etc/profile.d/set-rocm-asan-env.sh

      .. tab-item:: User setup

         Configure the ROCm ASAN environment for your user by updating your shell
         startup configuration file.

         .. selected:: i=tar

            Use the following commands to update your shell configuration file
            (``~/.bashrc`` or ``~/.profile``) and add ROCm ASAN to your PATH.
            Before proceeding, make sure you're in the ``therock-tarball-asan``
            directory so the install path resolves correctly.

         .. selected:: i=pkgman

            Use the following commands to update your shell configuration file
            (``~/.bashrc`` or ``~/.profile``) and add ROCm ASAN to your PATH.

         .. tab-set::

            .. tab-item:: .bashrc
               :sync: bashrc

               .. selected:: i=tar

                  .. code-block:: bash

                     # Configure ROCM_ASAN_PATH to the ASan install tree (tarball extract path)
                     ROCM_ASAN_INSTALL_PATH=$(pwd)/install
                     tee --append ~/.bashrc << EOF
                     # BEGIN ROCm ASan Configuration
                     export ROCM_ASAN_PATH=$ROCM_ASAN_INSTALL_PATH

                     # Add ROCm bin to PATH
                     export PATH=\$PATH:\$ROCM_ASAN_PATH/bin

                     # Enable XNACK for device-side GPU instrumentation
                     # Without this, only host-side (CPU) errors will be detected
                     export HSA_XNACK=1

                     # Locate ASan runtime directory and append instrumented library directories
                     ASAN_LIB_PATH=\$(amdclang --print-file-name=libclang_rt.asan-x86_64.so 2>/dev/null || echo "")
                     export LD_LIBRARY_PATH="\$LD_LIBRARY_PATH:\${ASAN_LIB_PATH%/*}:\${ROCM_ASAN_PATH}/lib:\${ROCM_ASAN_PATH}/lib/llvm/lib:\${ROCM_ASAN_PATH}/lib/rocm_sysdeps/lib/"
                     # END ROCm ASan Configuration
                     EOF
                     source ~/.bashrc

               .. selected:: i=pkgman

                  .. code-block:: bash

                     # Configure ROCM_ASAN_PATH to the ASan install tree
                     tee --append ~/.bashrc << 'EOF'
                     # BEGIN ROCm ASan Configuration
                     export ROCM_ASAN_PATH=/opt/rocm/core-asan-10.0

                     # Enable XNACK for device-side GPU instrumentation
                     # Without this, only host-side (CPU) errors will be detected
                     export HSA_XNACK=1

                     # Locate ASan runtime directory and append instrumented library directories
                     ASAN_LIB_PATH=$(amdclang --print-file-name=libclang_rt.asan-x86_64.so 2>/dev/null || echo "")
                     export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:${ASAN_LIB_PATH%/*}:${ROCM_ASAN_PATH}/lib:${ROCM_ASAN_PATH}/lib/llvm/lib:${ROCM_ASAN_PATH}/lib/rocm_sysdeps/lib/"
                     # END ROCm ASan Configuration
                     EOF
                     source ~/.bashrc

            .. tab-item:: .profile
               :sync: profile

               .. selected:: i=tar

                  .. code-block:: bash

                     # Configure ROCM_ASAN_PATH to the ASan install tree (tarball extract path)
                     ROCM_ASAN_INSTALL_PATH=$(pwd)/install
                     tee --append ~/.profile << EOF
                     # BEGIN ROCm ASan Configuration
                     export ROCM_ASAN_PATH=$ROCM_ASAN_INSTALL_PATH

                     # Add ROCm bin to PATH
                     export PATH=\$PATH:\$ROCM_ASAN_PATH/bin

                     # Enable XNACK for device-side GPU instrumentation
                     # Without this, only host-side (CPU) errors will be detected
                     export HSA_XNACK=1

                     # Locate ASan runtime directory and append instrumented library directories
                     ASAN_LIB_PATH=\$(amdclang --print-file-name=libclang_rt.asan-x86_64.so 2>/dev/null || echo "")
                     export LD_LIBRARY_PATH="\$LD_LIBRARY_PATH:\${ASAN_LIB_PATH%/*}:\${ROCM_ASAN_PATH}/lib:\${ROCM_ASAN_PATH}/lib/llvm/lib:\${ROCM_ASAN_PATH}/lib/rocm_sysdeps/lib/"
                     # END ROCm ASan Configuration
                     EOF
                     source ~/.profile

               .. selected:: i=pkgman

                  .. code-block:: bash

                     # Configure ROCM_ASAN_PATH to the ASan install tree
                     tee --append ~/.profile << 'EOF'
                     # BEGIN ROCm ASan Configuration
                     export ROCM_ASAN_PATH=/opt/rocm/core-asan-10.0

                     # Enable XNACK for device-side GPU instrumentation
                     # Without this, only host-side (CPU) errors will be detected
                     export HSA_XNACK=1

                     # Locate ASan runtime directory and append instrumented library directories
                     ASAN_LIB_PATH=$(amdclang --print-file-name=libclang_rt.asan-x86_64.so 2>/dev/null || echo "")
                     export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:${ASAN_LIB_PATH%/*}:${ROCM_ASAN_PATH}/lib:${ROCM_ASAN_PATH}/lib/llvm/lib:${ROCM_ASAN_PATH}/lib/rocm_sysdeps/lib/"
                     # END ROCm ASan Configuration
                     EOF
                     source ~/.profile

Verify your installation
------------------------

Use the following ROCm tools to verify that ROCm ASAN is correctly installed
and that your AMD devices are visible to the system.

Use ``rocminfo`` to list detected AMD GPUs and confirm that the ROCm runtimes
and drivers are correctly installed and loaded:

.. code-block:: bash

   rocminfo

Use the AMD SMI CLI ``amd-smi`` to validate system information:

.. code-block:: bash

   amd-smi version

----

.. _rocm-asan-uninstall:

Uninstall ROCm ASAN
===================

.. ========================================================== PACKAGE MANAGER ==

.. selected:: i=pkgman
   :heading: Uninstall ROCm ASAN packages
   :heading-level: 3

   1. Use your package manager to remove the installed packages.

      .. selected:: os=ubuntu os=debian

         .. selected:: fam=all

            .. code-block:: bash

               sudo apt autoremove amdrocm-asan10.0

         .. selected:: gfx=gfx942

            .. code-block:: bash

               sudo apt autoremove amdrocm-asan10.0-gfx942

         .. selected:: gfx=gfx950

            .. code-block:: bash

               sudo apt autoremove amdrocm-asan10.0-gfx950

      .. selected:: os=rhel os=oracle-linux os=rocky-linux

         .. selected:: fam=all

            .. code-block:: bash

               sudo dnf remove amdrocm-asan10.0

         .. selected:: gfx=gfx942

            .. code-block:: bash

               sudo dnf remove amdrocm-asan10.0-gfx942

         .. selected:: gfx=gfx950

            .. code-block:: bash

               sudo dnf remove amdrocm-asan10.0-gfx950

      .. selected:: os=sles

         .. code-block:: bash

            sudo zypper remove amdrocm-*-asan10.0*

   2. Remove ROCm repositories.

      .. selected:: os=ubuntu os=debian

         .. code-block:: bash

            sudo rm -f /etc/apt/sources.list.d/amdrocm-stable.sources

            # Clear the cache and clean the system
            sudo rm -rf /var/cache/apt/*
            sudo apt clean all
            sudo apt update

      .. selected:: os=rhel os=oracle-linux os=rocky-linux

         .. code-block:: bash

            sudo rm -f /etc/yum.repos.d/amdrocm-stable.repo*

            # Clear the cache and clean the system
            sudo rm -rf /var/cache/dnf
            sudo dnf clean all

      .. selected:: os=sles

         .. code-block:: bash

            sudo zypper removerepo "amdrocm-stable"

            # Clear the cache and clean the system
            sudo zypper clean --all
            sudo zypper refresh

   3. Remove the ROCm environment variables from your configuration.

      .. tab-set::

         .. tab-item:: System-wide setup

            .. code-block:: bash

               sudo rm -f /etc/profile.d/set-rocm-asan-env.sh

         .. tab-item:: User setup

            Remove the ROCm environment configuration block from your shell
            configuration file (``~/.bashrc`` or ``~/.profile``).

.. ================================================================== TARBALL ==

.. selected:: i=tar
   :heading: Uninstall the ROCm ASAN tarball
   :heading-level: 3

   1. Remove the directory containing the ROCm ASAN installation:

      .. important::

         The following command assumes you're working with the
         ``therock-tarball-asan`` directory. If you chose a different directory
         name when `installing ROCm <https://rocm.docs.amd.com/en/latest/install/rocm.html#rocm-install>`_, adjust the
         command accordingly.

      .. code-block:: bash

         rm -rf therock-tarball-asan

   2. Remove the ROCm environment variables from your configuration.

      .. tab-set::

         .. tab-item:: System-wide setup

            .. code-block:: bash

               sudo rm -f /etc/profile.d/set-rocm-asan-env.sh

         .. tab-item:: User setup

            Remove the ROCm environment configuration block from your shell
            configuration file (``~/.bashrc`` or ``~/.profile``).

----

.. _rocm-asan-next-steps:

Next steps
==========

To run applications with ASAN instrumentation, ensure the following:

- Linux kernel ≥ 5.6 with HMM enabled
- ``HSA_XNACK=1``
- ``-fsanitize=address``
- Instrumented runtimes

For more information, see the `GPU sanitizer guide
<https://github.com/ROCm/TheRock/blob/main/docs/development/sanitizers.md#using-asan-instrumented-libraries>`_.
