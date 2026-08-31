<div align="center">
  <img src="docs/images/amd-rocm-logo.png" width="200px" alt="ROCm logo">
  <h3 align="center">
    Source for the ROCm documentation website
  </h3>
  <p align="center">
    <a href="https://rocm.docs.amd.com/en/latest/">
      <b>rocm.docs.amd.com</b>
    </a>
  </p>
</div>

# ROCm documentation

This repository contains the source for the top-level
[ROCm documentation](https://rocm.docs.amd.com/en/latest/) site. It builds the
landing pages, installation and compatibility guides, and the navigation that
ties the per-project documentation together.

It is a documentation-only repository: it holds no ROCm source code. Individual
ROCm components keep their own documentation in their own repositories, which
this site links out to.

## Building the documentation

The site is built with [Sphinx](https://www.sphinx-doc.org/) and
[`rocm-docs-core`](https://github.com/ROCm/rocm-docs-core). To build it locally:

```bash
cd docs/sphinx
pip install -r requirements.txt
python -m sphinx -T -b html -d _build/doctrees -D language=en ../.. _build/html
```

The rendered site is written to `docs/sphinx/_build/html`. Open `index.html`
there to preview it. See [Contributing to ROCm docs](https://rocm.docs.amd.com/en/latest/contribute/contributing.html)
for authoring guidelines, style, and the review process.

## Contribute

To propose documentation changes, open a pull request against this repository.
See [CONTRIBUTING.md](CONTRIBUTING.md) and the
[ROCm documentation contribution guide](https://rocm.docs.amd.com/en/latest/contribute/contributing.html).

For contribution guidelines in the wider ROCm project, see:

- [TheRock](https://github.com/ROCm/TheRock/blob/main/CONTRIBUTING.md)
- [ROCm Systems](https://github.com/ROCm/rocm-systems/blob/develop/CONTRIBUTING.md)
- [ROCm Libraries](https://github.com/ROCm/rocm-libraries/blob/develop/CONTRIBUTING.md)

## Security

To report a security vulnerability, see [SECURITY.md](SECURITY.md). Do not
report vulnerabilities through public GitHub issues.

## Licensing

See the [LICENSE](LICENSE) file for this repository. For ROCm-wide licensing,
see the [ROCm licenses](https://rocm.docs.amd.com/en/latest/about/license.html)
page.
