.. meta::
    :description: Setting the number of CUs
    :keywords: CU, CUs, number of CUs, compute, unit

.. _settings-cus-reference:

*******************************
Set the number of compute units
*******************************

.. figure:: /images/shared/compute-unit.png

   Schematic representation of a compute unit in the CDNA 2 or CDNA 3 architecture.

The GPU driver provides two environment variables to set the number of compute units (CUs) used:

- ``HSA_CU_MASK``
- ``ROC_GLOBAL_CU_MASK``

The ``ROC_GLOBAL_CU_MASK`` variable sets the CU mask on queues created by HIP or OpenCL runtimes. The ``HSA_CU_MASK`` variable sets the mask on a lower level of queue creation in the driver. It also sets the mask on the queues being profiled.

.. tip::

   When using GPUs to accelerate compute workloads, it sometimes becomes necessary to configure the hardware's usage of compute units (CU). This is a more advanced option, so please read this page before experimentation.

The environment variables have the following syntax:

::

    ID = [0-9][0-9]*                         ex. base 10 numbers
    ID_list = (ID | ID-ID)[, (ID | ID-ID)]*  ex. 0,2-4,7
    GPU_list = ID_list                       ex. 0,2-4,7
    CU_list = 0x[0-F]* | ID_list             ex. 0x337F OR 0,2-4,7
    CU_Set = GPU_list : CU_list              ex. 0,2-4,7:0-15,32-47 OR 0,2-4,7:0x337F
    HSA_CU_MASK = CU_Set [; CU_Set]*         ex. 0,2-4,7:0-15,32-47; 3-9:0x337F

GPU indices refer to the GPU order after ``ROCR_VISIBLE_DEVICES`` reordering. ``HIP_VISIBLE_DEVICES`` and ``CUDA_VISIBLE_DEVICES`` don't change this numbering. For listed GPUs, the listed or masked CUs are enabled and the others are disabled. Unlisted GPUs aren't affected, and their CUs remain enabled.

.. note::

   A process started with ``HIP_VISIBLE_DEVICES=5`` uses GPU 5. ``HSA_CU_MASK=0:0-15`` restricts GPU 0, not GPU 5, to CUs 0-15, so the process's kernels can still use every CU of GPU 5. To limit this process to CUs 0-15 of GPU 5, set ``HSA_CU_MASK=5:0-15``, or select the GPU with ``ROCR_VISIBLE_DEVICES=5`` instead of ``HIP_VISIBLE_DEVICES`` and set ``HSA_CU_MASK=0:0-15``. With ``ROCR_VISIBLE_DEVICES=5``, the selected GPU is GPU 0 for both ROCr and HIP.

Variable parsing stops when a syntax error occurs, and ROCr reports no error or warning. The erroneous set and the following sets are ignored. Repeating GPU or CU IDs results in a syntax error. Specifying a mask with no usable CUs (``CU_list`` is ``0x0``) results in a syntax error. To exclude GPU devices, use ``ROCR_VISIBLE_DEVICES``.

.. note::

   These environment variables only affect ROCm software, not graphics applications.

Not all CU configurations are valid on all devices. For example, on devices where two CUs can be combined into a WGP (for kernels running in WGP mode), it’s not valid to disable only a single CU in a WGP.

