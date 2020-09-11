# guide to the `Unity.Entities` API.
<!-- 
> Topics to add
> * 
-->

This guide is meant to walk users through the Entities API, in rough order of most-commmonly used features to least-commonly used. The interspersed samples demonstrate how to put things together.

The guide assumes familiarity with the [basic Entities concepts](), and some details are left to the [API reference]().

|||
| --- | --- |
|  using [`EntityManager`](api_creating_entities.md)  | Create and destroy entities, and add and remove the components of entities.  | 
|  using [`EntityQuery`]()  | Find all entities with a specified set of component types.  | 
|  using [`SystemBase`]()  | Systems are the core units of code in ECS. |
| **sample: [HelloCube]()** | Displays a rotating cube using `Unity.Transforms` and `Unity.Rendering.Hybrid`.  |
|  using [subscenes and conversion]()| Create entities from scenes. |
|  using [`EntityCommandBuffer`]()  | Record structural changes for playback later.  |
|  using [`IJobChunk`]()  | Access entities in a job.  |
|  using [`Entities.ForEach`]()  | A more convenient (but less powerful) way to create and schedule `IJobChunk` jobs. |
|  using [`World`]()  |  A World is comprised of a set of entities and a set of system instances.  |  
|  using [`SystemGroup`]()  | Organize the systems of a `World` into a hierarchy that determines the update order of the systems.  |
| using [fixed time step]() |
|  using [shared components]()  |    |
|  using [chunk components]()  |    |
|  using [system state components]()  |    |
|  using [dynamic buffer components]()  |    |
|  using [blob assets]()  |    |
|  using [versioning]()  |    |
|  using [write groups]()  |    |
|  using [entity transactions]()  |    |
