---
sidebar_position: 4
---

# Hydra flow
## Goal
The goal of Hydra is to provide coupling with Medusa in order to allow for micro-frontends. What we mean by that is the ability to develop and deploy frontends as independent services and then combine them into a singular user experience for the user.

Some potential usecases:
- Content can be made dynamically available based on the availability of a service. For example, if the ordering system is down, the informational part of the site could still keep working.
- The user only has to log in once, even if different services require different logins.
- New versions of features can be tested by only conditionally exposing it to users via weights (10% of users see new feature, 90% as-is)
- Etc.

## Flow
The way the flow works is mostly based on Spring Cloud Gateway. We derive the available services at run-time due to RSocket connections. 

So in other words, every Medusa app deployed points to a Hydra cluster. This starts a heartbeat RSocket channel.

The channel's basic connection / disconnect works as a way to know which services are available to the Hydra cluster. We spread the information to the cluster by simply storing it in memory for singular instances and Redis for clustered deploys.

As the channel is active, the two-way connection can be used to pass along fragment data.

### Heartbeat channel

Hydra is not the initiator of registry. It simply listens for incoming requests.

An app with Medusa will have a pointer to a Hydra instance. During startup initializations, once all routes are found, it shoots a post message to /h/discovery/{publicKey}/registration. 
It provides its application name, its available paths and its available resources.

Any Medusa app only connects to 1 Hydra instance. Hydra shares its information with other Hydra instances via Redis, or keeps it in-memory in case of a single instance. 
Which Hydra instance it connects to should not matter, to allow for load balancing access.

Hydra accepts the message and uses this incoming information of Medusa's routes/resources. A few things happen:
- It stores the routes/resources in shared memory (thus sharing it with its cluster, if applicable) and changes its overallRouteHashKey in shared memory.
- It returns an overview of the currently active services. This lets Medusa immediately boot up with Hydra capabilities such as dynamic menu items.

Every second, Hydra checks this routeHashKey to see if it has changed. If it has, it will reload all routes/resources from memory and provision Spring Cloud Gateway routes to them. 
Since this data is shared, any Hydra instance in the cluster will pick these up within 1 second, even if they have no direct channel connection.

If this gets detected, it updates all its connected Medusa instances as well with this new overview.

To stay active, the Medusa app sends an isAlive every 3 seconds. If no isAlive is received for more than 5 seconds, it is assumed the service is inactive.

On the Medusa side, a connection termination will trigger an infinite connection retry every second.

## Fragments
Fragments are the core of combined page micro-frontends. The idea is to have a system that can reliably combine parts of pages from different microservices (aka fragments) into a core page.

So the routing works as normal for the core page, but during rendering it will find fragment tags and resolve them by asking Hydra for its info. If no such info is available, it shows the fallback.

A core page with a fragment tag would look like this:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

<h1>Hello, this is a core page</h1>

<m:fragment service="orderService" ref="checkout" exports="postalZipCode, addressLine2 as address" imports="dob, countryOrigin as country">
    Fallback info
</m:fragment>

</body>
</html>
```
Fragments must be executed on their respective servers and HTML is returned to the root server where it is 'fragmented'. The servers connect to Hydra anyway, so Hydra can use that connection to ask to render certain HTML.

This is illustrated with exports and imports to be as expansive as possible. By default, fragments and their attributes are a fully closed system. In other words, fragments do not interact with the core page or other fragments by default.

If such coordination is required, it should be explicitly pointed out.

- An export means that an Attribute with matching keyName will be made available to the core page. In case of a conflict, an exception will be thrown at compile-time. Developers can specify an alias via 'as my-alias' to specify a non-conflicting name.
- An import means that an Attribute available to the core page will be made available to the fragment. These can also be aliased in case of conflict.
- Service / Ref define which Medusa UI app (service) and which page fragment (ref) get used.

### Fragment render requests via channel

Upon registration, Hydra knows how to reconnect with the Medusa instances, which allows for communication from Hydra to connected services. 

Medusa will again initiate the flow, when rendering a root page containing one or more fragments. It sends a List of Fragments to Hydra via /h/discovery/{publicKey}/requestFragment.

Hydra will receive this and map these requests onto active services. Requests without such active services result in null, which means the fallback will be triggered on Medusa side.

If an active service is available, it will be used to forward the relevant Fragments to the relevant Medusa instances. Through the active instances, a POST call is forwarded to Medusa asking to render the Fragment.
It passes the request on to the renderer, which will render it locally and update the Session accordingly. The render and Session then get returned to Hydra.

Hydra combines the finished renders and updated Sessions, then sends a ResolvedFragments object back.

Medusa can then use this object to complete its rendering.