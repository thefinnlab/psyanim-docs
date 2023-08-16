# Entity Prefabs

## 1. What's a `prefab`?

We've seen, in the previous tutorials, that `entities` are objects that live in a `scene`.

We've also learned that we can add state and behavior to an `entity` by attaching `components` to it.

The simplest `entities` may have no `components` or maybe just 1 `component` attached to them.  For these `entities`, it's easy enough to create more of them simply by attaching the same type of `component` to another `entity` in the same `scene` or in other `scenes`.

However, as soon as we start creating & using agents with more complex behaviors requiring many components, such as the `Playfight`, `Predator`, or `Prey`, attaching all the necessary `components` and configuring them all properly can become unecessarily repetitive and cumbersome.

// TODO: ...

An `Entity Prefab` is like a blueprint that captures the necessary components for creating an entity with specific state and behaviors.

In addition to capturing the necessary components, an `Entity Prefab` also handles the component configuration and exposes custom configuration parameters, providing a simpler interface for entity creation than the dealing with the individual components themselves.
 
// TODO: ...
