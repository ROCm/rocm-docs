.. selected:: rocm-ver=10.1.0

   .. selected:: i=tar

      The standard ROCm uninstallation process can be followed to uninstall HPC
      SDK. No additional steps are required to remove the HPC SDK separately.
      Refer to `Uninstalling ROCm 10.1.0
      <https://rocm.docs.amd.com/en/docs-10.1.0/install/rocm.html#uninstalling>`__
      section and select **Tarball** from the installation environment
      selector.

   .. selected:: i=pkgman

      .. selected:: fam=all

         Use the following command to uninstall HPC SDK for all GPU architectures:

         .. selected:: os=ubuntu os=debian

            .. code-block:: bash

                sudo apt autoremove amdrocm-hpc10.1 amdrocm-hpc-sdk10.1

         .. selected:: os=rhel os=rocky-linux os=oracle-linux

            .. code-block:: bash

                sudo dnf remove amdrocm-hpc10.1 amdrocm-hpc-sdk10.1

         .. selected:: os=sles

            .. code-block:: bash

                sudo zypper remove amdrocm-hpc10.1 amdrocm-hpc-sdk10.1

      .. selected:: gfx=gfx950

         Use the following command to uninstall HPC SDK for your ``gfx950`` GPU:

         .. selected:: os=ubuntu os=debian

            .. code-block:: bash

                sudo apt autoremove amdrocm-hpc10.1-gfx950 amdrocm-hpc-sdk10.1-gfx950

         .. selected:: os=rhel os=rocky-linux os=oracle-linux

            .. code-block:: bash

                sudo dnf remove amdrocm-hpc10.1-gfx950 amdrocm-hpc-sdk10.1-gfx950

         .. selected:: os=sles

            .. code-block:: bash

                sudo zypper remove amdrocm-hpc10.1-gfx950 amdrocm-hpc-sdk10.1-gfx950

      .. selected:: gfx=gfx942

         Use the following command to uninstall HPC SDK for your ``gfx942`` GPU:

         .. selected:: os=ubuntu os=debian

            .. code-block:: bash

                sudo apt autoremove amdrocm-hpc10.1-gfx942 amdrocm-hpc-sdk10.1-gfx942

         .. selected:: os=rhel os=rocky-linux os=oracle-linux

            .. code-block:: bash

                sudo dnf remove amdrocm-hpc10.1-gfx942 amdrocm-hpc-sdk10.1-gfx942

         .. selected:: os=sles

            .. code-block:: bash

                sudo zypper remove amdrocm-hpc10.1-gfx942 amdrocm-hpc-sdk10.1-gfx942

      .. selected:: gfx=gfx90a

         Use the following command to uninstall HPC SDK for your ``gfx90a`` GPU:

         .. selected:: os=ubuntu os=debian

            .. code-block:: bash

                sudo apt autoremove amdrocm-hpc10.1-gfx90a amdrocm-hpc-sdk10.1-gfx90a

         .. selected:: os=rhel os=rocky-linux os=oracle-linux

            .. code-block:: bash

                sudo dnf remove amdrocm-hpc10.1-gfx90a amdrocm-hpc-sdk10.1-gfx90a

         .. selected:: os=sles

            .. code-block:: bash

                sudo zypper remove amdrocm-hpc10.1-gfx90a amdrocm-hpc-sdk10.1-gfx90a

      .. selected:: gfx=gfx908

         Use the following command to uninstall HPC SDK for your ``gfx908`` GPU:

         .. selected:: os=ubuntu os=debian

            .. code-block:: bash

                sudo apt autoremove amdrocm-hpc10.1-gfx908 amdrocm-hpc-sdk10.1-gfx908

         .. selected:: os=rhel os=rocky-linux os=oracle-linux

            .. code-block:: bash

                sudo dnf remove amdrocm-hpc10.1-gfx908 amdrocm-hpc-sdk10.1-gfx908

         .. selected:: os=sles

            .. code-block:: bash

                sudo zypper remove amdrocm-hpc10.1-gfx908 amdrocm-hpc-sdk10.1-gfx908

      .. selected:: gfx=gfx1200

         Use the following command to uninstall HPC SDK for your ``gfx1200`` GPU:

         .. selected:: os=ubuntu os=debian

            .. code-block:: bash

                sudo apt autoremove amdrocm-hpc10.1-gfx1200 amdrocm-hpc-sdk10.1-gfx1200

         .. selected:: os=rhel os=rocky-linux os=oracle-linux

            .. code-block:: bash

                sudo dnf remove amdrocm-hpc10.1-gfx1200 amdrocm-hpc-sdk10.1-gfx1200

         .. selected:: os=sles

            .. code-block:: bash

                sudo zypper remove amdrocm-hpc10.1-gfx1200 amdrocm-hpc-sdk10.1-gfx1200

      .. selected:: gfx=gfx1201

         Use the following command to uninstall HPC SDK for your ``gfx1201`` GPU:

         .. selected:: os=ubuntu os=debian

            .. code-block:: bash

                sudo apt autoremove amdrocm-hpc10.1-gfx1201 amdrocm-hpc-sdk10.1-gfx1201

         .. selected:: os=rhel os=rocky-linux os=oracle-linux

            .. code-block:: bash

                sudo dnf remove amdrocm-hpc10.1-gfx1201 amdrocm-hpc-sdk10.1-gfx1201

         .. selected:: os=sles

            .. code-block:: bash

                sudo zypper remove amdrocm-hpc10.1-gfx1201 amdrocm-hpc-sdk10.1-gfx1201

      .. selected:: gfx=gfx1100

         Use the following command to uninstall HPC SDK for your ``gfx1100`` GPU:

         .. selected:: os=ubuntu os=debian

            .. code-block:: bash

                sudo apt autoremove amdrocm-hpc10.1-gfx1100 amdrocm-hpc-sdk10.1-gfx1100

         .. selected:: os=rhel os=rocky-linux os=oracle-linux

            .. code-block:: bash

                sudo dnf remove amdrocm-hpc10.1-gfx1100 amdrocm-hpc-sdk10.1-gfx1100

         .. selected:: os=sles

            .. code-block:: bash

                sudo zypper remove amdrocm-hpc10.1-gfx1100 amdrocm-hpc-sdk10.1-gfx1100

      .. selected:: gfx=gfx1101

         Use the following command to uninstall HPC SDK for your ``gfx1101`` GPU:

         .. selected:: os=ubuntu os=debian

            .. code-block:: bash

                sudo apt autoremove amdrocm-hpc10.1-gfx1101 amdrocm-hpc-sdk10.1-gfx1101

         .. selected:: os=rhel os=rocky-linux os=oracle-linux

            .. code-block:: bash

                sudo dnf remove amdrocm-hpc10.1-gfx1101 amdrocm-hpc-sdk10.1-gfx1101

         .. selected:: os=sles

            .. code-block:: bash

                sudo zypper remove amdrocm-hpc10.1-gfx1101 amdrocm-hpc-sdk10.1-gfx1101

      .. selected:: gfx=gfx1102

         Use the following command to uninstall HPC SDK for your ``gfx1102`` GPU:

         .. selected:: os=ubuntu os=debian

            .. code-block:: bash

                sudo apt autoremove amdrocm-hpc10.1-gfx1102 amdrocm-hpc-sdk10.1-gfx1102

         .. selected:: os=rhel os=rocky-linux os=oracle-linux

            .. code-block:: bash

                sudo dnf remove amdrocm-hpc10.1-gfx1102 amdrocm-hpc-sdk10.1-gfx1102

         .. selected:: os=sles

            .. code-block:: bash

                sudo zypper remove amdrocm-hpc10.1-gfx1102 amdrocm-hpc-sdk10.1-gfx1102

      .. selected:: gfx=gfx1103

         Use the following command to uninstall HPC SDK for your ``gfx1103`` GPU:

         .. selected:: os=ubuntu os=debian

            .. code-block:: bash

                sudo apt autoremove amdrocm-hpc10.1-gfx1103 amdrocm-hpc-sdk10.1-gfx1103

         .. selected:: os=rhel os=rocky-linux os=oracle-linux

            .. code-block:: bash

                sudo dnf remove amdrocm-hpc10.1-gfx1103 amdrocm-hpc-sdk10.1-gfx1103

         .. selected:: os=sles

            .. code-block:: bash

                sudo zypper remove amdrocm-hpc10.1-gfx1103 amdrocm-hpc-sdk10.1-gfx1103

      .. selected:: gfx=gfx1151

         Use the following command to uninstall HPC SDK for your ``gfx1151`` GPU:

         .. selected:: os=ubuntu os=debian

            .. code-block:: bash

                sudo apt autoremove amdrocm-hpc10.1-gfx1151 amdrocm-hpc-sdk10.1-gfx1151

         .. selected:: os=rhel os=rocky-linux os=oracle-linux

            .. code-block:: bash

                sudo dnf remove amdrocm-hpc10.1-gfx1151 amdrocm-hpc-sdk10.1-gfx1151

         .. selected:: os=sles

            .. code-block:: bash

                sudo zypper remove amdrocm-hpc10.1-gfx1151 amdrocm-hpc-sdk10.1-gfx1151

      .. selected:: gfx=gfx1150

         Use the following command to uninstall HPC SDK for your ``gfx1150`` GPU:

         .. selected:: os=ubuntu os=debian

            .. code-block:: bash

                sudo apt autoremove amdrocm-hpc10.1-gfx1150 amdrocm-hpc-sdk10.1-gfx1150

         .. selected:: os=rhel os=rocky-linux os=oracle-linux

            .. code-block:: bash

                sudo dnf remove amdrocm-hpc10.1-gfx1150 amdrocm-hpc-sdk10.1-gfx1150

         .. selected:: os=sles

            .. code-block:: bash

                sudo zypper remove amdrocm-hpc10.1-gfx1150 amdrocm-hpc-sdk10.1-gfx1150

      .. selected:: gfx=gfx1152

         Use the following command to uninstall HPC SDK for your ``gfx1152`` GPU:

         .. selected:: os=ubuntu os=debian

            .. code-block:: bash

                sudo apt autoremove amdrocm-hpc10.1-gfx1152 amdrocm-hpc-sdk10.1-gfx1152

         .. selected:: os=rhel os=rocky-linux os=oracle-linux

            .. code-block:: bash

                sudo dnf remove amdrocm-hpc10.1-gfx1152 amdrocm-hpc-sdk10.1-gfx1152

         .. selected:: os=sles

            .. code-block:: bash

                sudo zypper remove amdrocm-hpc10.1-gfx1152 amdrocm-hpc-sdk10.1-gfx1152

      .. selected:: gfx=gfx1153

         Use the following command to uninstall HPC SDK for your ``gfx1153`` GPU:

         .. selected:: os=ubuntu os=debian

            .. code-block:: bash

                sudo apt autoremove amdrocm-hpc10.1-gfx1153 amdrocm-hpc-sdk10.1-gfx1153

         .. selected:: os=rhel os=rocky-linux os=oracle-linux

            .. code-block:: bash

                sudo dnf remove amdrocm-hpc10.1-gfx1153 amdrocm-hpc-sdk10.1-gfx1153

         .. selected:: os=sles

            .. code-block:: bash

                sudo zypper remove amdrocm-hpc10.1-gfx1153 amdrocm-hpc-sdk10.1-gfx1153


