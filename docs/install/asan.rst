.. meta::
   :description: How to install AMD ROCm with AddressSanitizer (ASAN) instrumentation for AMD Instinct GPUs
   :keywords: linux, install, download, setup, asan, addresssanitizer, sanitizer, ubuntu, debian, red, hat, rhel, oracle, rocky, suse, sles, instinct, mi300a, mi300x, mi350x, gfx942, gfx950, package, manager, tarball

:selector-toc2: Installation environment
:selector-toc2-icon: fa-solid fa-computer

**********************************************
Install AMD ROCm with ASAN (AddressSanitizer)
**********************************************

.. _rocm-asan-install:

ASAN (AddressSanitizer) builds of ROCm are available for specific AMD Instinct
GPU architectures and can be installed using the package manager or a tarball.
ASAN-instrumented libraries help you detect memory errors such as out-of-bounds
accesses and use-after-free bugs in applications that use ROCm.

ASAN packages install to ``/opt/rocm/core-asan-10.0``, separate from a regular
ROCm installation at ``/opt/rocm/core-10.0``, so you can keep both on the same
system.

.. note::

   ASAN builds are only available for the ``gfx942`` and ``gfx950``
   architectures, plus a multiarch build (``all``) that supports both. ASAN
   packages use the naming convention ``amdrocm-<component>-asan10.0`` or
   ``amdrocm-<component>-asan10.0-gfx<XYZ>`` for architecture-specific builds.

Before installing ROCm ASAN, make sure your system meets the ROCm hardware,
software, and driver requirements. For instructions, see the
:ref:`ROCm installation prerequisites <rocm-prerequisites>`. For system
requirements and support information, see the :doc:`Compatibility matrix
</compatibility/compatibility-matrix>`.

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

      .. selector-option:: AMD Instinct MI350X (gfx950)
         :value: amd-instinct-mi350x gfx=gfx950

      .. selector-option:: AMD Instinct MI300X (gfx942)
         :value: amd-instinct-mi300x gfx=gfx942

      .. selector-option:: AMD Instinct MI300A (gfx942)
         :value: amd-instinct-mi300a gfx=gfx942

.. selector:: Operating system
   :key: os

   .. selector-option:: Ubuntu
      :value: ubuntu
      :width: 2

   .. selector-option:: Debian
      :value: debian
      :width: 2

   .. selector-option:: RHEL
      :value: rhel
      :width: 2
      :toc-label: Red Hat Enterprise Linux

   .. selector-option:: Oracle Linux
      :value: oracle-linux
      :width: 2

   .. selector-option:: Rocky Linux
      :value: rocky-linux
      :width: 2

   .. selector-option:: SLES
      :value: sles
      :width: 2
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

.. _rocm-asan-install-rocm:

Install ROCm ASAN
=================

Use the following instructions to install ROCm ASAN packages on your system.

.. ========================================================== PACKAGE MANAGER ==

.. selected:: i=pkgman
   :heading: Register ROCm repositories
   :heading-level: 3

   Register the ASAN ROCm repository with your system's package manager. This
   lets you install and update ROCm ASAN packages.

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

      mkdir therock-tarball && cd therock-tarball

   .. important::

      Subsequent commands assume you're working with the ``therock-tarball``
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

.. selected:: i=tar
   :heading: Post-installation
   :heading-level: 3

   After installing ROCm ASAN 10.0.0, complete these post-installation steps to
   configure your system and validate the installation.

   Configure environment variables so that ROCm ASAN libraries and tools are
   available either to all users on the system or only to your user account.

   .. tab-set::

      .. tab-item:: System-wide setup

         Create a profile script so that all users inherit the ROCm environment
         variables when they start a shell session. Make sure you're in the
         ``therock-tarball`` directory before proceeding.

         .. code-block:: bash

            ROCM_INSTALL_PATH=$(pwd)/install
            sudo tee /etc/profile.d/set-rocm-env.sh << EOF
            export ROCM_PATH=$ROCM_INSTALL_PATH
            export PATH=\$PATH:\$ROCM_PATH/bin
            export LD_LIBRARY_PATH=\$ROCM_PATH/lib
            EOF
            sudo chmod +x /etc/profile.d/set-rocm-env.sh
            source /etc/profile.d/set-rocm-env.sh

      .. tab-item:: User setup

         Configure the ROCm environment for your user by updating your shell
         startup configuration file (``~/.bashrc`` or ``~/.profile``). Make sure
         you're in the ``therock-tarball`` directory so the install path
         resolves correctly.

         .. code-block:: bash

            ROCM_INSTALL_PATH=$(pwd)/install
            tee --append ~/.bashrc << EOF

            # BEGIN ROCm environment configuration
            export ROCM_PATH=$ROCM_INSTALL_PATH
            export PATH=\$PATH:\$ROCM_PATH/bin
            export LD_LIBRARY_PATH=\$ROCM_PATH/lib
            # END ROCm environment configuration
            EOF
            source ~/.bashrc

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

            sudo zypper remove amdrocm*

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

.. ================================================================== TARBALL ==

.. selected:: i=tar
   :heading: Uninstall the ROCm ASAN tarball
   :heading-level: 3

   1. Remove the directory containing the ROCm ASAN installation:

      .. code-block:: bash

         rm -rf ~/therock-tarball

   2. Remove the ROCm environment variables from your configuration.

      .. tab-set::

         .. tab-item:: System-wide setup

            .. code-block:: bash

               sudo rm -f /etc/profile.d/set-rocm-env.sh

         .. tab-item:: User setup

            Edit your ``~/.bashrc`` file and remove the ROCm environment
            configuration section:

            .. code-block:: bash

               # BEGIN ROCm environment configuration
               export ROCM_PATH=$ROCM_INSTALL_PATH
               export PATH=$PATH:$ROCM_PATH/bin
               export LD_LIBRARY_PATH=$ROCM_PATH/lib
               # END ROCm environment configuration

   3. Reload your shell configuration:

      .. code-block:: bash

         source ~/.bashrc
