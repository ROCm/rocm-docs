.. selected:: gpu=mi355x gpu=mi350x gpu=mi350p gpu=mi325x gpu=mi300x gpu=mi210
   :heading: GPU virtualization support

   .. selected:: gpu=mi355x gpu=mi350x gpu=mi350p gpu=mi325x gpu=mi300x gpu=mi210

      Supported SR-IOV configurations require the GPU-IOV Module (GIM) driver
      9.1.0.K -- see the `AMD Instinct Virtualization Driver documentation
      <https://instinct.docs.amd.com/projects/virt-drv/en/mainline-9.1.0.k/>`__ to
      get started.


   .. matrix::

      .. matrix-row::
         :header:

         .. matrix-cell:: Hypervisor

         .. matrix-cell:: Virtualization technology

         .. matrix-cell:: Virtualization driver

         .. matrix-cell:: Host OS

         .. matrix-cell:: Guest OS

      .. matrix-row::
         :show-cond: gpu=mi355x

         .. matrix-cell:: KVM
            :rowspan: 5

         .. matrix-cell:: Passthrough
            :rowspan: 2

         .. matrix-cell:: —
            :rowspan: 2

         .. matrix-cell:: Ubuntu 26.04

         .. matrix-cell:: Ubuntu 26.04

      .. matrix-row::
         :show-cond: gpu=mi355x

         .. matrix-cell:: Ubuntu 24.04

         .. matrix-cell:: Ubuntu 24.04

      .. matrix-row::
         :show-cond: gpu=mi355x

         .. matrix-cell:: SR-IOV
            :rowspan: 3

         .. matrix-cell::
            :rowspan: 3

            `GIM 9.1.0.K <https://github.com/amd/MxGPU-Virtualization/releases/tag/9.1.0.K>`__

         .. matrix-cell:: Ubuntu 24.04
            :rowspan: 3

         .. matrix-cell:: Ubuntu 24.04

      .. matrix-row::
         :show-cond: gpu=mi355x

         .. matrix-cell:: RHEL 10.0

      .. matrix-row::
         :show-cond: gpu=mi355x

         .. matrix-cell:: RHEL 9.6

      .. matrix-row::
         :show-cond: gpu=mi355x

         .. matrix-cell:: ESXi

         .. matrix-cell:: SR-IOV

         .. matrix-cell:: —

         .. matrix-cell:: ESXi 9.1

         .. matrix-cell:: Ubuntu 24.04

      .. matrix-row::
         :show-cond: gpu=mi350x

         .. matrix-cell:: KVM
            :rowspan: 5

         .. matrix-cell:: Passthrough
            :rowspan: 2

         .. matrix-cell:: —
            :rowspan: 2

         .. matrix-cell:: Ubuntu 26.04

         .. matrix-cell:: Ubuntu 26.04

      .. matrix-row::
         :show-cond: gpu=mi350x

         .. matrix-cell:: Ubuntu 24.04

         .. matrix-cell:: Ubuntu 24.04

      .. matrix-row::
         :show-cond: gpu=mi350x

         .. matrix-cell:: SR-IOV
            :rowspan: 3

         .. matrix-cell::
            :rowspan: 3

            `GIM 9.1.0.K <https://github.com/amd/MxGPU-Virtualization/releases/tag/9.1.0.K>`__

         .. matrix-cell:: Ubuntu 24.04
            :rowspan: 3

         .. matrix-cell:: Ubuntu 24.04

      .. matrix-row::
         :show-cond: gpu=mi350x

         .. matrix-cell:: RHEL 10.0

      .. matrix-row::
         :show-cond: gpu=mi350x

         .. matrix-cell:: RHEL 9.6

      .. matrix-row::
         :show-cond: gpu=mi350x

         .. matrix-cell:: ESXi

         .. matrix-cell:: SR-IOV

         .. matrix-cell:: —

         .. matrix-cell:: ESXi 9.1

         .. matrix-cell:: Ubuntu 24.04

      .. matrix-row::
         :show-cond: gpu=mi350p

         .. matrix-cell:: KVM

         .. matrix-cell:: Passthrough

         .. matrix-cell:: —

         .. matrix-cell:: Debian 13

         .. matrix-cell:: Ubuntu 24.04

      .. matrix-row::
         :show-cond: gpu=mi325x

         .. matrix-cell:: KVM
            :rowspan: 3

         .. matrix-cell:: Passthrough
            :rowspan: 2

         .. matrix-cell:: —
            :rowspan: 2

         .. matrix-cell:: Ubuntu 26.04

         .. matrix-cell:: Ubuntu 26.04

      .. matrix-row::
         :show-cond: gpu=mi325x

         .. matrix-cell:: Ubuntu 22.04

         .. matrix-cell:: Ubuntu 22.04

      .. matrix-row::
         :show-cond: gpu=mi325x

         .. matrix-cell:: SR-IOV

         .. matrix-cell::

            `GIM 9.1.0.K <https://github.com/amd/MxGPU-Virtualization/releases/tag/9.1.0.K>`__

         .. matrix-cell:: Ubuntu 22.04

         .. matrix-cell:: Ubuntu 22.04

      .. matrix-row::
         :show-cond: gpu=mi300x

         .. matrix-cell:: KVM
            :rowspan: 6

         .. matrix-cell:: Passthrough
            :rowspan: 2

         .. matrix-cell:: —
            :rowspan: 2

         .. matrix-cell:: Ubuntu 26.04

         .. matrix-cell:: Ubuntu 26.04

      .. matrix-row::
         :show-cond: gpu=mi300x

         .. matrix-cell:: Ubuntu 22.04

         .. matrix-cell:: Ubuntu 22.04

      .. matrix-row::
         :show-cond: gpu=mi300x

         .. matrix-cell:: SR-IOV
            :rowspan: 4

         .. matrix-cell::
            :rowspan: 4

            `GIM 9.1.0.K <https://github.com/amd/MxGPU-Virtualization/releases/tag/9.1.0.K>`__

         .. matrix-cell:: Ubuntu 24.04

         .. matrix-cell:: Ubuntu 24.04

      .. matrix-row::
         :show-cond: gpu=mi300x

         .. matrix-cell:: Ubuntu 22.04

         .. matrix-cell:: Ubuntu 22.04

      .. matrix-row::
         :show-cond: gpu=mi300x

         .. matrix-cell:: RHEL 9.4
            :rowspan: 2

         .. matrix-cell:: Ubuntu 22.04

      .. matrix-row::
         :show-cond: gpu=mi300x

         .. matrix-cell:: RHEL 9.4

      .. matrix-row::
         :show-cond: gpu=mi210

         .. matrix-cell:: KVM
            :rowspan: 4

         .. matrix-cell:: Passthrough
            :rowspan: 2

         .. matrix-cell:: —
            :rowspan: 2

         .. matrix-cell:: Ubuntu 26.04

         .. matrix-cell:: Ubuntu 26.04

      .. matrix-row::
         :show-cond: gpu=mi210

         .. matrix-cell:: RHEL 9.4

         .. matrix-cell:: Ubuntu 22.04

      .. matrix-row::
         :show-cond: gpu=mi210

         .. matrix-cell:: SR-IOV
            :rowspan: 2

         .. matrix-cell::
            :rowspan: 2

            `GIM 9.1.0.K <https://github.com/amd/MxGPU-Virtualization/releases/tag/9.1.0.K>`__

         .. matrix-cell:: RHEL 9.4
            :rowspan: 2

         .. matrix-cell:: Ubuntu 22.04

      .. matrix-row::
         :show-cond: gpu=mi210

         .. matrix-cell:: RHEL 9.4

   See the :ref:`release notes <release-virtualization-support>`
   for the full list of supported configurations.

