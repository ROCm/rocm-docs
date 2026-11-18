.. meta::
   :description: Learn how to use AMD ROCm for high-performance computing (HPC).
   :keywords: ROCm, HPC, high performance computing, scientific computing, GPU acceleration, AMD Instinct, InfinityHub

****************
AMD ROCm HPC SDK
****************

The AMD ROCm™ open-source software stack is optimized to extract high-performance
computing (HPC) workload performance from AMD Instinct™ GPUs while maintaining
compatibility with industry software frameworks.

ROCm enhances support and access for developers by providing streamlined and improved
tools that significantly increase productivity. Being open-source, ROCm fosters
innovation, differentiation, and collaboration within the developer community, making it
a powerful and accessible solution for leveraging the full potential of AMD GPUs'
capabilities in diverse computational applications.

Some of the most popular HPC frameworks are part of the ROCm HPC SDK, including those
that help parallelize operations across multiple GPUs and servers, handle memory
hierarchies, and solve linear systems. The HPC application catalog provides a vast set of platform-compatible HPC applications,
including those for astrophysics, climate and weather, computational chemistry,
computational fluid dynamics, earth science, genomics, geophysics, molecular dynamics,
and physics computing.

See :ref:`hpc-install` to install the ROCm HPC SDK.

.. datatemplate:yaml:: /data/components-current.yaml

    {%- set defaults = load("/data/components-default.yaml").rocm_hpc_sdk.components -%}
    {%- set current = data.rocm_hpc_sdk.components -%}
    {%- set slug = data.rocm_hpc_sdk.meta.rtd_version_slug -%}
    {%- set tag = data.rocm_hpc_sdk.meta.release_tag -%}
    {%- for name, comp in defaults.items() | sort(attribute="0") -%}
    {%-     set cur = current.get(name, {}) -%}
    {%-     set ver_label = " " + cur.version|string if cur.version is defined else "" -%}
    {%-     set desc = " -- " + comp.description if comp.description is defined else "" -%}
    {%-     if comp.xref is defined and comp.xref.rtd_project is defined -%}
    {%-         set url = comp.xref.rtd_project | replace("${rtd_version_slug}", slug) | replace("${release_tag}", tag) %}
    * `{{ name }}{{ ver_label }} <{{ url }}>`__{{ desc }}
    {%-     elif comp.xref is defined and comp.xref.github_repo is defined -%}
    {%-         set url = comp.xref.github_repo | replace("${rtd_version_slug}", slug) | replace("${release_tag}", tag) %}
    * `{{ name }}{{ ver_label }} <{{ url }}>`__{{ desc }}
    {%-     else %}
    * {{ name }}{{ ver_label }}{{ desc }}
    {%-     endif %}
    {%- endfor %}
