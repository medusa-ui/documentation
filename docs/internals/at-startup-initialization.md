---
sidebar_position: 1
---

# At startup initialization

Before any requests are served, the app goes through its startup initialization.

This is a sequence of classes that get executed at the BeanPostProcessor#postProcessBeforeInitialization level.

They are used for anything that is build-dependent. As such, things like determining which routes/controllers are 
possible is not something that happens at runtime.

It all starts with the RootDetector. This is the class that kicks off any initialization. That's all it does. 
Effectively this kicks off before each bean is initialized.

In addition to routes, all the HTML files are also scanned for Refs and Fragments, so it is known up-front which fragments are relevant, saving time during rendering.