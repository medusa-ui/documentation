---
sidebar_position: 2
---

# Internal flow of rendering

As part of Medusa v1.0.0, these are the steps the code takes to render a page. Relevant to note is that at this point, [at-startup initialization](/docs/internals/at-startup-initialization.md) has completed. 

This does not account for Hydra, which is described in the [Hydra flow](/docs/internals/hydra-flow.md). and describes its flow at more of an architectural high-level point. 

## Initial render

We start with a new request being made in the browser to our server. This request is picked up by the [request handler](/docs/internals/components.md#req-handler).

Here, we find the relevant controller. Using the controller, we find the combination of referenced raw Thymeleaf HTML and initial parameters. This also starts a [session](/docs/internals/components.md#session).

Said combination is used to [render](/docs/internals/components.md#renderer) the page. Within the page, we establish an RSocket channel. 

From that point on, any interaction with the page is done through the RSocket channel - at least until a page change, which counts as a new request.  

## Subsequence actions

As an action happens, we want to trigger a controller method server-side. Each action triggered through a Medusa tag, 
will be triggering a Javascript function which builds up an event and sends it through the RSocket channel.

The event is retrieved by the [Action Handler](/docs/internals/components.md#action-handler) component. 
This component retrieves the [session](/docs/internals/components.md#session), which can tell us which template was rendered.

Using this combination of information we can trigger the controller method and retrieve a set of [DOMChanges](/docs/internals/components.md#dom-changes). 

This set of dom changes can be merged with the last used attributes from the session, to then trigger a [re-render](/docs/internals/components.md#renderer) of the template.

This results in a new HTML. 

We then route this HTML though the [Diff Engine](/docs/internals/components.md#diff-engine), together with the last [rendered](/docs/internals/components.md#renderer) HTML from the [session](/docs/internals/components.md#session).

We then send the diffs back through the RSocket Channel. These diffs contain the changed HTML, the action (add, delete or edit a node) and the relevant xpath to the changing DOM node.

In the JS we then apply the diff in the browser via [morphdom](https://github.com/patrick-steele-idem/morphdom). 

## Server-to-client actions

Communication in Medusa is bidirectional, meaning changes can happen not only due to actions from the browser, but also from the server.

Key to this is access to the [session](/docs/internals/components.md#session) a user has. Each session can get tags assigned to it.

Developers can also add tags to sessions programmatically. 

Some such tags are also automatically added to the sessions, like the current page they are on and the kind of roles they have.

With the tags, these sessions can then be queried. With a list of sessions, these can then be used to send events.