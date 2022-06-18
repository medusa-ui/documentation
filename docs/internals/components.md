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

A session refers to a loaded page with an active open RSocket channel.

A session contains:
- Which template was rendered
- Which controller was used
- Last rendered HTML
- Last used parameters to render
- Tags: for bi-directionality

A session is stored in either local memory (when set up as a single instance) or redis (when set up as a multi-instance cluster).

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
Internally it uses XMLUnit's DiffEngine for this difference calculation. More on XMLUnit here: https://github.com/xmlunit/user-guide/wiki/Comparing-XML#monitoring-the-comparison-process

Diffs generated get mapped onto either an ADDITION, REMOVAL or EDIT POJO called JSReadyDiff. This corresponds with how the JS should interpret it.

Addition: Appends a row after the provided XPath. That means that the xpath always points to the previous sibling. If no previous sibling exists, the pointer is the parent entry with ::first. At that point the parent entry gets looked up and the entry added as a child.
Removal: XPath is found and removed
Edit: Morphdom is executed on the found XPath. This is essentially the same as element.innerHTML = "", except it is capable of dealing with nuances around caret positions, etc.

The core concept is thus quite simple. Of course, there are nuances:
- As you change elements, the XPaths can change. For example, if you have 3 list items and you have a pointer to change the 3rd one, but then also a removal of the 2nd one, then the order of execution would start to matter. As such, we do not send a diff one at a time, but we send it in batches (ie 1 batch per action, which can contain multiple diffs). The batch interpretation then goes in 2 phases. Phase 1: Iterate over all diffs and resolve their xpaths to an element. Phase 2: Execute them based on the already looked up elements.
- This comes with another problem, if the addition diff refers to the previous element, and there are multiple elements added in the same change, there can be inter-dependencies. For example, if you have a list with 1 item and a change causes 2 items to be added to the list, the first element will have a previous element of an existing item, but the second item will point to the newly created item. As such, specifically with additions, we allow for a last-chance lookup at execution time in case the element was not found during initial lookup in phase 1.
- Not all diffs are represented (yet). For example, we don't care if the child node list length is different - because we only care about the specific nodes.