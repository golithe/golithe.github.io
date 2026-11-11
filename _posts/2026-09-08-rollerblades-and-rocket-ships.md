---
layout: post
title: "Rollerblades and rocket ships: keeping an early inference pipeline ready for scale"
---

For the geospatial models I am currently building at Dira GeoSystems, our entire runtime fits on one GPU, fed by a sequential queue. But it is all lined up to serve at scale.

My argument is simple: precisely because this is all very early and nobody runs these at scale yet, the temptation to cut corners is strongest.

Why do I say that?

Because in the last couple of years I built roughly 50 prototypes and the main lesson is that it takes much more time to redesign a rocket ship when you originally blueprinted a pair of rollerblades. Usually the rollerblades were already sold as a rocket ship, gold paint and two bottle rockets duct taped on.

I am still a huge fan of YAGNI. But avoiding premature optimization is not the same as ignoring architectural integrity. You do not need to build every future feature now. But you do want to avoid today's prototype assumptions that make tomorrow incredibly expensive to change.

Three decisions that keep our inference pipeline clean:

- **Clients ask for a target, like "buildings", never a model version.** Internally that maps to an adapter, a checkpoint, and a queue. Swap the model, and the client contract does not move.
- **The orchestrator cannot see the models.** It knows tiles, batches, and progress, nothing about neural networks. CI checks read the syntax tree and block model imports from leaking past the adapter layer.
- **Loading fails loud on a version mismatch.** Every checkpoint carries its provenance. The adapter checks it against the installed package and refuses incompatible weights, so a silent mismatch never gets blamed on the model.

The honest constraint: today the API hands the worker a local file path, so both need the same filesystem. Moving to shared object storage is the main thing standing between us and workers we can distribute freely. But that change sits at the infrastructure boundary. It does not touch the client contract or the core pipeline.

My opinion: good architectural boundaries make scale far less painful. But scale is not a free byproduct of good architecture.
