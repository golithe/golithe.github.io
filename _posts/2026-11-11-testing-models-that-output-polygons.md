---
layout: post
title: "Testing a model that outputs polygons: how strict is strict enough?"
---

If you test models that output geometry: how do you set your tolerances?

Here's my take. One of our models at Dira GeoSystems takes an aerial image and returns polygons. Testing that the service still returns the right polygons after a code change or a library update was more challenging than I thought.

The first idea was a snapshot test: I thought I could just run a few fixed tiles, commit the output, and let tests fail if anything changes. I found out quickly that that doesn't even work on CPU (yes, CPU, not GPU: our CI runners don't have GPUs). Floating point details differ between machines, so the same code gives slightly different coordinates on my laptop and on the GitHub runner.

So I realized that the simplest possible test probably would just be to compare polygon by polygon. The number of polygons has to match exactly, and every expected polygon needs its own counterpart with an IoU of at least 0.999. Strict enough to catch a real change, loose enough to survive a different CPU.

On GPU, even that fails. We run inference in mixed precision: on the GPU, most of the math runs in 16-bit floats instead of 32-bit, which is faster but only keeps about three significant digits. Each GPU generation uses its own optimised kernels, which add numbers up in a different order, so the results differ slightly. Same model, same tile: an RTX 5090 and an RTX 6000 Ada gave a merged-area IoU between 0.988 and 0.9995, and one polygon more or less. So depending on the card, you get a different number of polygons.

That's why the GPU test compares the total area instead: IoU of at least 0.98, and the count may differ by two.

A few more things I ran into:

- One fixture is a JPEG-compressed tile. JPEG decoding depends on the GDAL version, so a library update can change the input pixels before the model even sees them.
- The GPU test is expensive (build the image, start a pod, run the tiles), so it doesn't run on every change (more on that in a future post).
- When the strict CPU test fails after an update, it's usually a tiny shift. If the benchmark moves by around 0.1 percentage points, I accept it and recapture the fixtures. No big analysis.

The fixtures have one more simple but very important job. A lot of the service code is written by AI assistants, and the fixtures are the one thing the assistant is not allowed to change. Only a script regenerates them, and only when I decide the output should change.

If you do it differently, I'd like to hear how.
