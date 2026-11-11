---
layout: default
---

ML engineer working on computer vision and GeoAI. Currently ML Lead at Dira GeoSystems in Zurich, where I build models that turn aerial imagery into maps, and the service that runs them.

Open-source contributions to [cog](https://github.com/replicate/cog), [rules_python](https://github.com/bazel-contrib/rules_python), [bazel-watcher](https://github.com/bazelbuild/bazel-watcher), [langfuse-python](https://github.com/langfuse/langfuse-python), [Building-Regulariser](https://github.com/DPIRD-DMA/Building-Regulariser) and [RustGIS](https://github.com/opengeos/RustGIS).

## Posts

{% for post in site.posts -%}
- {{ post.date | date: "%b %-d, %Y" }} · [{{ post.title }}]({{ post.url | relative_url }})
{% endfor %}

## Talks

- Jun 2026 · **A good data model is worth a complex neural network.** Oral presentation at GeoAI 2026, Ghent. [Slides]({{ "/assets/geoai-2026-slides.pdf" | relative_url }})

## Interviews

- Sep 2026 · **Every training label was lying, and the loss curve looked fine.** Interview with Ludovico Bessi for ML@SCALE. [Read](https://machinelearningatscale.substack.com/p/mlscale-11-every-training-label-was)

## Publications

- Zhou, Z., Winkler, J., Fu, C. and Weibel, R. (2026). **A good data model is worth a complex neural network: evidence from building generalization using deep learning.** Proceedings of the 1st International Conference on Geospatial Artificial Intelligence (GeoAI 2026). [Paper](https://zenodo.org/records/20287707)
