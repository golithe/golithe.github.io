---
layout: post
title: "\"No kernel image is available\": catching torch and GPU mismatches before you ship"
---

If you work with NVIDIA GPUs you know the ecosystem is fiddly: drivers, CUDA toolkits, and then whatever framework you actually use on top, like PyTorch, to run training or inference.

Here's a situation I hit a lot. I pull in some repo I want to try, maybe from a paper. The authors published and moved on, so the repo is stale and the pinned versions (if they even pinned) are ancient. So I bump torch, numpy, python and see what breaks. You're stuck guessing at a version combo that works, and the trap is that a wrong guess still builds green.

Because if you're on a newer card, say a Blackwell 50-series, you can hit something nastier.

You run `torch.cuda.is_available()`, it returns `True`, everything looks fine. Then you actually run something and the process dies with:

```
no kernel image is available for execution on the device
```

What's going on: PyTorch wheels ship precompiled kernels for a fixed set of GPU architectures. If you pin an old torch and run it on a newer architecture (`sm_120` for Blackwell), the wheel just doesn't contain kernels for your card. It still builds. `is_available()` still says `True`. There's nothing to actually launch, so every CUDA op fails at runtime. PyTorch only warns about this with a `UserWarning`, which is easy to miss in logs. Worst case you don't find out until the container hits the GPU server and the job errors out.

I contributed [a PR to cog](https://github.com/replicate/cog/pull/3104) that catches this early. It's unglamorous infra work, but the kind that saves time and money: `cog doctor` now reads your GPU's compute capability, figures out the exact torch wheel cog will install, and compares that against a compatibility table. If they don't line up, you get a clear warning before you build instead of after you've shipped.

It doesn't fix the incompatibility. It just points it out early so you can react to it. For me, working somewhere without a big infra team where a lot of this is still manual, that's worth a lot.

Hope this helps some fellow ML engineers in the same boat. Thanks to the cog reviewer for a thorough pass.
