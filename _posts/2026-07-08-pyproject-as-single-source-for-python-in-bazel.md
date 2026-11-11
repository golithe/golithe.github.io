---
layout: post
title: "One Python version for Bazel: pyproject.toml as the single source of truth"
---

I have built a lot of ML pipelines. Sometimes with fancy models, mostly with boring models. Model work is the part people post about. But whether any of it survives production depends on something much less shiny.

Reproducibility.

If you work with Python, you know that Python makes reproducibility tough in a lot of cases. When you use Python in Bazel, however, you might know that Bazel is one of the few ecosystems that treats the CPython interpreter itself as a pinned build input rather than whatever is on the host machine. But even there, the Python version typically lives in three places: `pyproject.toml`, the toolchain registration, and `pip.parse`. Nothing keeps them in sync and you have to handle these three versions manually, which is annoying and leads to errors.

But your days of endless crying are over.

With the next rules_python release, [my recently merged PR](https://github.com/bazel-contrib/rules_python/pull/3514) ships. Point the build at your `pyproject.toml` and the `requires-python` field becomes the single source of truth. The interpreter Bazel fetches, the registered toolchain, and the pip resolution all follow that one file. No need for `python_version` in BUILD files or `.python-version` files in your project root. A wrong Python version in the build is no longer a state the system can reach.

The details: the toolchain is auto-registered, `pyproject.toml` is parsed with a pure-Starlark TOML decoder (reading the version needs no host Python), and the file is watched. Which makes automated upgrades trivial: e.g. Renovate just bumps `requires-python`, and that single-line PR upgrades interpreter, toolchain and resolved deps together.

By design it expects exact version pinning (`==X.Y.Z`). That fits application repos where you want exact, reproducible and hermetic builds (as opposed to published packages where you might be more lenient with version ranges) and is what makes the automation clean.

I know this all sounds incredibly boring. But in my opinion boring and reliable wins over shiny and unreliable.
