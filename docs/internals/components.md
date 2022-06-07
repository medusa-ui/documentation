---
sidebar_position: 3
---

# Components

### <a name="dom-changes"></a> DOM Changes
io.medusa.medusa.core.domchanges
TODO

### <a name="req-handler"></a> Request handler
io.getmedusa.medusa.core.router.request

The initial request only resolves if it matches a route set up initially in the RouteSetup class. This is a class that connects the routes loaded during at-startup initialization with a process to handle its rendering.

It then proceeds onto the RequestStreamHandler - the goal of which is to trigger the actual response and rendering of HTML. 
This is one of two implementations of IRequestStreamHandler: The distinction lies between whether an actual Spring Security context is added or not.

Regardless, a session is built up and associated with the request. The actual rendering itself is handled by the renderer.

TODO

### <a name="session"></a> Session
io.getmedusa.medusa.core.session
TODO
Contains:
- Which template was rendered
- Which controller was used
- Last rendered HTML
- Last used parameters to render
- Tags: for bi-directionality

### <a name="renderer"></a> Renderer
io.getmedusa.medusa.core.render
TODO

### <a name="action-handler"></a> Action Handler
io.getmedusa.medusa.core.router.action
TODO

### <a name="diff-engine"></a> Diff Engine
io.getmedusa.medusa.core.diffengine
TODO
