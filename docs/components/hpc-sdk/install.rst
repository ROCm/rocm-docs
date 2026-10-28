:selector-toc2: Installation environment
:selector-toc2-icon: fa-solid fa-computer

.. meta::
   :description: Install ROCm to run high-performance computing (HPC) workloads.
   :keywords: ROCm, HPC, install, installation, Linux, AMD Instinct

.. _hpc-install:

********************
Install ROCm HPC SDK
********************

AMD ROCm HPC SDK provides high-performance computing libraries and tools for
AMD Instinct GPUs. This guide walks you through installing the HPC SDK
alongside ROCm installation on a supported Linux distribution.

The ROCm for HPC applications and containers run on a standard ROCm
installation. See the :ref:`Compatibility matrix <compat-matrix>` for details
on supported hardware and operating systems.

The HPC application containers are published through
`AMD InfinityHub-CI <https://github.com/amd/InfinityHub-CI>`_. Each container
provides parameters to specify source code branches and release versions of ROCm,
OpenMPI, UCX, and Ubuntu.

.. selector:: Device family
   :key: fam

   .. selector-option:: All
      :value: all
      :width: 6

   .. selector-option:: AMD Instinct™
      :value: instinct
      :width: 6
      :toc-label: AMD Instinct

.. datatemplate:yaml:: /data/gpus.yaml
   :template: gpu-selector.rst.jinja

.. datatemplate:yaml:: /data/gpus.yaml
   :template: misc/os-selector-no-windows.rst.jinja

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

Before installing the HPC SDK, make sure your system meets the ROCm hardware,
software, and driver requirements. For instructions, see :ref:`Install AMD ROCm <rocm-install-selector>`. Use the
selector panel on that page to view instructions appropriate for your system
environment.

HPC SDK includes `hipTensor <https://github.com/ROCm/rocm-libraries/tree/develop/projects/hiptensor>`_ and `rocALUTION <https://github.com/ROCm/rocm-libraries/tree/develop/projects/rocalution>`_ packaged as part of the installation.

Install HPC SDK
---------------

.. selected:: i=tar

   The standard ROCm tarball installation includes the HPC SDK. No
   additional steps are required. For details on ROCm tarball installation,
   refer to `Install AMD ROCm 10.0.0
   <https://rocm.docs.amd.com/en/docs-10.0.0/install/rocm.html>`__ and
   select **Tarball** installation method from installation environment
   selector.

.. selected:: i=pkgman

   1. :doc:`Install ROCm </install/rocm>`. Remember to complete the :ref:`ROCm
      installation prerequisites <rocm-prerequisites>` to install dependencies
      and configure GPU access permissions.

   .. selected:: fam=all

      2. Use the following command to install HPC SDK for all GPU architectures:

         .. selected:: os=ubuntu os=debian

            .. code-block:: bash

                sudo apt install amdrocm-hpc10.0 amdrocm-hpc-sdk10.0

         .. selected:: os=rhel os=rocky-linux os=oracle-linux

            .. code-block:: bash

                sudo dnf install amdrocm-hpc10.0 amdrocm-hpc-sdk10.0

         .. selected:: os=sles

            .. code-block:: bash

                sudo zypper install amdrocm-hpc10.0 amdrocm-hpc-sdk10.0

   .. selected:: gfx=gfx950

      2. Use the following command to install HPC SDK for your ``gfx950`` GPU:

         .. selected:: os=ubuntu os=debian

            .. code-block:: bash

                sudo apt install amdrocm-hpc10.0-gfx950 amdrocm-hpc-sdk10.0-gfx950

         .. selected:: os=rhel os=rocky-linux os=oracle-linux

            .. code-block:: bash

                sudo dnf install amdrocm-hpc10.0-gfx950 amdrocm-hpc-sdk10.0-gfx950

         .. selected:: os=sles

            .. code-block:: bash

                sudo zypper install amdrocm-hpc10.0-gfx950 amdrocm-hpc-sdk10.0-gfx950

   .. selected:: gfx=gfx942

      2. Use the following command to install HPC SDK for your ``gfx942`` GPU:

         .. selected:: os=ubuntu os=debian

            .. code-block:: bash

                sudo apt install amdrocm-hpc10.0-gfx942 amdrocm-hpc-sdk10.0-gfx942

         .. selected:: os=rhel os=rocky-linux os=oracle-linux

            .. code-block:: bash

                sudo dnf install amdrocm-hpc10.0-gfx942 amdrocm-hpc-sdk10.0-gfx942

         .. selected:: os=sles

            .. code-block:: bash

                sudo zypper install amdrocm-hpc10.0-gfx942 amdrocm-hpc-sdk10.0-gfx942

   .. selected:: gfx=gfx90a

      2. Use the following command to install HPC SDK for your ``gfx90a`` GPU:

         .. selected:: os=ubuntu os=debian

            .. code-block:: bash

                sudo apt install amdrocm-hpc10.0-gfx90a amdrocm-hpc-sdk10.0-gfx90a

         .. selected:: os=rhel os=rocky-linux os=oracle-linux

            .. code-block:: bash

                sudo dnf install amdrocm-hpc10.0-gfx90a amdrocm-hpc-sdk10.0-gfx90a

         .. selected:: os=sles

            .. code-block:: bash

                sudo zypper install amdrocm-hpc10.0-gfx90a amdrocm-hpc-sdk10.0-gfx90a

   .. selected:: gfx=gfx908

      2. Use the following command to install HPC SDK for your ``gfx908`` GPU:

         .. selected:: os=ubuntu os=debian

            .. code-block:: bash

                sudo apt install amdrocm-hpc10.0-gfx908 amdrocm-hpc-sdk10.0-gfx908

         .. selected:: os=rhel os=rocky-linux os=oracle-linux

            .. code-block:: bash

                sudo dnf install amdrocm-hpc10.0-gfx908 amdrocm-hpc-sdk10.0-gfx908

         .. selected:: os=sles

            .. code-block:: bash

                sudo zypper install amdrocm-hpc10.0-gfx908 amdrocm-hpc-sdk10.0-gfx908

Uninstall HPC SDK
----------------------

.. selected:: i=tar

   The standard ROCm uninstallation process can be followed to uninstall HPC
   SDK. No additional steps are required to remove the HPC SDK separately.
   Refer to `Uninstalling ROCm 10.0.0
   <https://rocm.docs.amd.com/en/docs-10.0.0/install/rocm.html#uninstalling>`__
   section and select **Tarball** from the installation environment
   selector.

.. selected:: i=pkgman

   .. selected:: fam=all

      Use the following command to uninstall HPC SDK for all GPU architectures:

      .. selected:: os=ubuntu os=debian

         .. code-block:: bash

             sudo apt autoremove amdrocm-hpc10.0 amdrocm-hpc-sdk10.0

      .. selected:: os=rhel os=rocky-linux os=oracle-linux

         .. code-block:: bash

             sudo dnf remove amdrocm-hpc10.0 amdrocm-hpc-sdk10.0

      .. selected:: os=sles

         .. code-block:: bash

             sudo zypper remove amdrocm-hpc10.0 amdrocm-hpc-sdk10.0

   .. selected:: gfx=gfx950

      Use the following command to uninstall HPC SDK for your ``gfx950`` GPU:

      .. selected:: os=ubuntu os=debian

         .. code-block:: bash

             sudo apt autoremove amdrocm-hpc10.0-gfx950 amdrocm-hpc-sdk10.0-gfx950

      .. selected:: os=rhel os=rocky-linux os=oracle-linux

         .. code-block:: bash

             sudo dnf remove amdrocm-hpc10.0-gfx950 amdrocm-hpc-sdk10.0-gfx950

      .. selected:: os=sles

         .. code-block:: bash

             sudo zypper remove amdrocm-hpc10.0-gfx950 amdrocm-hpc-sdk10.0-gfx950

   .. selected:: gfx=gfx942

      Use the following command to uninstall HPC SDK for your ``gfx942`` GPU:

      .. selected:: os=ubuntu os=debian

         .. code-block:: bash

             sudo apt autoremove amdrocm-hpc10.0-gfx942 amdrocm-hpc-sdk10.0-gfx942

      .. selected:: os=rhel os=rocky-linux os=oracle-linux

         .. code-block:: bash

             sudo dnf remove amdrocm-hpc10.0-gfx942 amdrocm-hpc-sdk10.0-gfx942

      .. selected:: os=sles

         .. code-block:: bash

             sudo zypper remove amdrocm-hpc10.0-gfx942 amdrocm-hpc-sdk10.0-gfx942

   .. selected:: gfx=gfx90a

      Use the following command to uninstall HPC SDK for your ``gfx90a`` GPU:

      .. selected:: os=ubuntu os=debian

         .. code-block:: bash

             sudo apt autoremove amdrocm-hpc10.0-gfx90a amdrocm-hpc-sdk10.0-gfx90a

      .. selected:: os=rhel os=rocky-linux os=oracle-linux

         .. code-block:: bash

             sudo dnf remove amdrocm-hpc10.0-gfx90a amdrocm-hpc-sdk10.0-gfx90a

      .. selected:: os=sles

         .. code-block:: bash

             sudo zypper remove amdrocm-hpc10.0-gfx90a amdrocm-hpc-sdk10.0-gfx90a

   .. selected:: gfx=gfx908

      Use the following command to uninstall HPC SDK for your ``gfx908`` GPU:

      .. selected:: os=ubuntu os=debian

         .. code-block:: bash

             sudo apt autoremove amdrocm-hpc10.0-gfx908 amdrocm-hpc-sdk10.0-gfx908

      .. selected:: os=rhel os=rocky-linux os=oracle-linux

         .. code-block:: bash

             sudo dnf remove amdrocm-hpc10.0-gfx908 amdrocm-hpc-sdk10.0-gfx908

      .. selected:: os=sles

         .. code-block:: bash

             sudo zypper remove amdrocm-hpc10.0-gfx908 amdrocm-hpc-sdk10.0-gfx908
