.. meta::
   :description: AMD ROCm profiling and debugging tools for GPU application performance analysis and fault diagnosis.
   :keywords: profiler, debugger, rocprofiler, ROCgdb, ROCm, performance, tracing

**********************************
ROCm profiling and debugging tools
**********************************

ROCm profiling and debugging tools help you measure GPU application performance,
identify bottlenecks, and diagnose execution faults.

For an overview of the profiling tools, their relationships, and how to use
them together, see :doc:`profiling-tools-overview`. For guidance on choosing the right tool for your performance investigation, see
:doc:`profiling-tools-selection`.

.. datatemplate:yaml:: /data/components-current.yaml

    {%- set defaults = load("/data/components-default.yaml").rocm_core_sdk.components -%}
    {%- set current = data.rocm_core_sdk.components -%}
    {%- set slug = data.rocm_core_sdk.meta.rtd_version_slug -%}
    {%- set tag = data.rocm_core_sdk.meta.release_tag -%}
    {%- for name, comp in defaults.items() | sort(attribute="0") -%}
    {%-     if comp.group == "Profiling and debugging tools" -%}
    {%-         set cur = current.get(name, {}) -%}
    {%-         set ver_label = " " + cur.version|string if cur.version is defined else "" -%}
    {%-         set desc = " -- " + comp.description if comp.description is defined else "" -%}
    {%-         if comp.xref is defined and comp.xref.rtd_project is defined -%}
    {%-             set url = comp.xref.rtd_project | replace("${rtd_version_slug}", slug) | replace("${release_tag}", tag) %}
    * `{{ name }}{{ ver_label }} <{{ url }}>`__{{ desc }}
    {%-         elif comp.xref is defined and comp.xref.github_repo is defined -%}
    {%-             set url = comp.xref.github_repo | replace("${rtd_version_slug}", slug) | replace("${release_tag}", tag) %}
    * `{{ name }}{{ ver_label }} <{{ url }}>`__{{ desc }}
    {%-         else %}
    * {{ name }}{{ ver_label }}{{ desc }}
    {%-         endif %}
    {%-     endif -%}
    {%- endfor %}

.. note::

   In addition to the profiling tools included in the ROCm Core SDK, AMD
   provides standalone visualization and analysis tools that help developers
   explore, interpret, and gain deeper insights from collected performance
   data. These tools are distributed separately and complement the ROCm
   profiling workflow.

   * :doc:`ROCm Optiq <roc-optiq:index>` is a unified tool for visualizing and
     analyzing performance data collected by ROCm Systems Profiler and ROCm
     Compute Profiler, providing insight into both system-level behavior and
     kernel-level performance for applications running on the ROCm stack. It
     is distributed separately as part of :doc:`ROCm Extras <extras>`.


   * :doc:`ROCprof Compute Viewer <rocprof-compute-viewer:index>` visualizes
     and analyzes GPU thread trace data collected using ``rocprofv3``,
     helping developers understand low-level GPU execution behavior,
     identify performance bottlenecks, and optimize kernel efficiency.

