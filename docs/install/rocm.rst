.. meta::
   :description: How to install AMD ROCm for Instinct GPUs, Radeon GPUs, and Ryzen AI APUs
   :keywords: linux, distro, windows, install, download, setup, quick, start, amdgpu-install, pkg, package, meta, ubuntu, debian, red, hat, rhel, suse, sles, enterprise, server, oracle, azure, centos, rocky, fedora, void, arch, cachy, pop, mint, tar

:selector-toc2: Installation environment
:selector-toc2-icon: fa-solid fa-computer

*******************************
Install AMD ROCm |ROCM_VERSION|
*******************************

.. _rocm-install-methods:

.. include:: ./include/000-install-methods.rst

----

.. _rocm-install-selector:

Use the following selector to choose your installation method for your
supported AMD GPU or APU and operating system. For system requirements and
support information, see the :doc:`Compatibility matrix
</compatibility/compatibility-matrix>`. To learn more about changes introduced
in ROCm |ROCM_VERSION|, see the :doc:`Release notes </about/release-notes>`.

.. note::

   If your GPU is not listed, it might be community-enabled through TheRock
   nightly builds. For more information, see `TheRock supported GPUs
   <https://github.com/ROCm/TheRock/blob/main/SUPPORTED_GPUS.md>`__. For
   installation guidance, see `TheRock releases
   <https://github.com/ROCm/TheRock/blob/main/RELEASES.md>`__.

.. datatemplate:yaml:: /data/gpus.yaml
   :template: fam-selector.rst.jinja

.. selected:: fam=all fam=radeon fam=ryzen

   .. selector:: Use case
      :key: w

      .. selector-option:: Compute
         :value: compute
         :width: 50%

      .. selector-option:: Compute + graphics
         :value: graphics
         :width: 50%

.. selected:: w=compute

   .. datatemplate:yaml:: /data/gpus.yaml
      :template: gpu-selector.rst.jinja

   .. datatemplate:yaml:: /data/gpus.yaml
      :template: os-selector.rst.jinja

.. selected:: w=graphics

   .. datatemplate:yaml:: /data/gpus.yaml
      :template: misc/os-selector-graphics-workloads.rst.jinja

.. datatemplate:yaml:: /data/gpus.yaml
   :template: os-version-selector.rst.jinja

.. selected:: w=graphics

   .. selected:: os=windows

      .. datatemplate:yaml:: /data/gpus.yaml
         :template: gpu-selector.rst.jinja

   .. selector:: Installation method
      :key: i
      :show-cond: os=ubuntu os=rhel

      .. selector-option:: amdgpu-install
         :value: amdgpu-install
         :width: 12

.. selected:: w=compute

   .. selected:: fam=all

      .. selector:: Installation method
         :show-cond: os=ubuntu os=debian
         :key: i

         .. selector-option:: apt
            :value: pkgman
            :width: 3

         .. selector-option:: pip
            :value: pip
            :width: 3

         .. selector-option:: Tarball
            :value: tar
            :width: 3

         .. selector-option:: Runfile
            :value: runfile
            :width: 3

      .. selector:: Installation method
         :show-cond: os=rhel os=oracle-linux os=rocky-linux
         :key: i

         .. selector-option:: dnf
            :value: pkgman
            :width: 3

         .. selector-option:: pip
            :value: pip
            :width: 3

         .. selector-option:: Tarball
            :value: tar
            :width: 3

         .. selector-option:: Runfile
            :value: runfile
            :width: 3

      .. selector:: Installation method
         :show-cond: os=sles
         :key: i

         .. selector-option:: zypper
            :value: pkgman
            :width: 3

         .. selector-option:: pip
            :value: pip
            :width: 3

         .. selector-option:: Tarball
            :value: tar
            :width: 3

         .. selector-option:: Runfile
            :value: runfile
            :width: 3

   .. selected:: fam=instinct fam=radeon fam=ryzen

      .. selector:: Installation method
         :show-cond: os=ubuntu os=debian
         :key: i

         .. selector-option:: apt
            :value: pkgman
            :width: 3

         .. selector-option:: pip
            :value: pip
            :width: 3

         .. selector-option:: Tarball
            :value: tar
            :width: 3

         .. selector-option:: Runfile
            :value: runfile
            :width: 3

      .. selector:: Installation method
         :show-cond: os=rhel os=oracle-linux os=rocky-linux
         :key: i

         .. selector-option:: dnf
            :value: pkgman
            :width: 3

         .. selector-option:: pip
            :value: pip
            :width: 3

         .. selector-option:: Tarball
            :value: tar
            :width: 3

         .. selector-option:: Runfile
            :value: runfile
            :width: 3

      .. selector:: Installation method
         :show-cond: os=sles
         :key: i

         .. selector-option:: zypper
            :value: pkgman
            :width: 3

         .. selector-option:: pip
            :value: pip
            :width: 3

         .. selector-option:: Tarball
            :value: tar
            :width: 3

         .. selector-option:: Runfile
            :value: runfile
            :width: 3

.. selector:: Installation method
   :show-cond: os=windows
   :key: i

   .. selector-option:: pip
      :value: pip
      :width: 6

   .. selector-option:: Tarball
      :value: tar
      :width: 6

----

.. _rocm-install-about:

.. include:: ./include/050-intro.rst

----

.. _rocm-prerequisites:

.. include:: ./include/100-prerequisites.rst

.. include:: ./include/150-runfile-quick-start-config-options.rst

----

.. _rocm-install:

.. include:: ./include/200-install.rst

----

.. _rocm-post-install:

.. include:: ./include/300-post-install.rst

----

.. _rocm-uninstall:

.. include:: ./include/400-uninstall.rst

|
|
|
|
|
|
|
|
|

.. raw:: html

   <script>
     document.addEventListener("DOMContentLoaded", () => {
       const nextLink = document.querySelector("footer.prev-next-footer a.right-next");
       const nextTitle = nextLink.querySelector(".prev-next-title");
       nextTitle.textContent = "Install HPC SDK";
       nextLink.href = "./hpc-sdk.html";
     });
   </script>
