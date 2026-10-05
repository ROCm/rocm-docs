.. meta::
   :description: AMD ROCm Core SDK - list of libraries and tools
   :keywords: component, tool, lib, library, dnn, algorithm, cli, end, machine, learning, optimization, optimize, primitive, api, binding, wrapper

************************
ROCm Core SDK components
************************

The ROCm Core SDK is the foundation of the ROCm software stack. It provides the
libraries, runtimes, compilers, and tools needed to develop and run GPU-accelerated
applications on AMD hardware.

.. datatemplate:yaml:: /data/components-current.yaml

    {%- set defaults = load("/data/components-default.yaml").rocm_core_sdk.components -%}
    {%- set current = data.rocm_core_sdk.components -%}
    {%- set slug = data.rocm_core_sdk.meta.rtd_version_slug -%}
    {%- set tag = data.rocm_core_sdk.meta.release_tag -%}
    {%- set group_order = [
        "Math and compute libraries",
        "Communication libraries",
        "Runtime and compilers",
        "Profiling and debugging tools",
        "Control and monitoring tools",
        "Media and vision libraries",
        "Storage libraries",
        "Threading libraries"
    ] -%}

    {# Collect groups not in the predefined order so they appear at the end. #}
    {%- set extra_groups = [] -%}
    {%- for name, comp in defaults.items() -%}
    {%-     if comp.group is defined and comp.group not in group_order and comp.group not in extra_groups -%}
    {%-         set _ = extra_groups.append(comp.group) -%}
    {%-     endif -%}
    {%- endfor -%}

    {%- for group in group_order + extra_groups -%}
    {%-     set group_items = [] -%}
    {%-     for name, comp in defaults.items() -%}
    {%-         if comp.group is defined and comp.group == group -%}
    {%-             set _ = group_items.append((name, comp)) -%}
    {%-         endif -%}
    {%-     endfor -%}
    {%-     if group_items %}

    {{ group }}
    {{ "=" * group | length }}
    {%- if group == "Math and compute libraries" %}

    A comprehensive set of GPU-accelerated math libraries covering dense and sparse
    linear algebra, FFTs, random number generation, and more.

    * Libraries prefixed with ``roc*`` are native, high-performance
      implementations written in HIP specifically for AMD GPUs.

    * Libraries prefixed with ``hip*`` are portable wrappers that implement
      NVIDIA CUDA-equivalent APIs, allowing CUDA applications to be ported to AMD GPUs
      with minimal code changes.

    Libraries include:
    {% endif %}
    {%- for name, comp in group_items | sort(attribute="0") -%}
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
    {%-     endif -%}
    {%- endfor %}
