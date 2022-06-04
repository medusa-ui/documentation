---
sidebar_position: 2
---

# Internal flow of rendering

As part of Medusa v1.0.0, these are the steps the code takes to render a page. Relevant to note is that at this point, [at-startup initialization](/docs/internals/at-startup-initialization.md) has completed. 

This does not account for Hydra, which is described in the [Hydra flow](/docs/internals/hydra-flow.md). and describes its flow at more of an architectural high-level point. 

## Initial render

We start with a new request being made in the browser to our server. This request is picked up by the [request handler](/docs/internals/components.md#req-handler).


## Subsequence actions
