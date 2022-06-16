---
sidebar_position: 3
---

# Components

### <a name="dom-changes"></a> Attributes
`io.medusa.medusa.core.domchanges`

Key/value pairs that get used as the model attributes during rendering.

### <a name="req-handler"></a> Request handler
`io.getmedusa.medusa.core.router.request`

The initial request only resolves if it matches a route set up initially in the RouteSetup class. This is a class that connects the routes loaded during at-startup initialization with a process to handle its rendering.

It then proceeds onto the RequestStreamHandler - the goal of which is to trigger the actual response and rendering of HTML. 
This is one of two implementations of IRequestStreamHandler: The distinction lies between whether an actual Spring Security context is added or not.

Regardless, a session is built up and associated with the request. The actual rendering itself is handled by the renderer.

### <a name="session"></a> Session
`io.getmedusa.medusa.core.session`

A session contains:
- Which template was rendered
- Which controller was used
- Last rendered HTML
- Last used parameters to render
- Tags: for bi-directionality

### <a name="renderer"></a> Renderer
`io.getmedusa.medusa.core.render`

Uses Thymeleaf to render the page with the provided Session's attributes.
We use Thymeleaf for its rendering speed, and it's ability to be extended with new tags.

After the render, websocket.js is injected and some JS variables are initialized.

### <a name="action-handler"></a> Action Handler
`io.getmedusa.medusa.core.router.action`

The Action Handler is a service that can take an operation call from JS (for example `doThisAction(1, 'example')`) and resolves it to an actual controller method. It does this via Spring's Expression language interpreter. 
The result is a set of Attributes, which get merged into the Session.

### <a name="diff-engine"></a> Diff Engine
`io.getmedusa.medusa.core.diffengine`

The diff engine takes two HTMLs and calculates the difference between them. The differences are exposed as a combination of xpath pointers and before/after HTML. 
Internally it uses XMLUnit for this difference calculation.
