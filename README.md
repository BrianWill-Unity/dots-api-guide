# guide to the `Unity.Entities` API.
<!-- 
> Topics to add
> * 
-->

This guide is meant to walk users through the Entities API, in rough order of most-commmonly used features to least-commonly used. The interspersed samples demonstrate how to put things together.

The guide assumes familiarity with the [basic Entities concepts](), and some details are left to the [API reference]().

|||
| --- | --- |
|  [`EntityManager`](api_creating_entities.md)  | Create and destroy entities, and add and remove the components of entities.  | 
|  [`EntityQuery`](api_entity_query.md)  | Find all entities with a specified set of component types.  | 
|  [`SystemBase`](api_subscenes.md)  | Systems are the core units of code in ECS. |
|  [`SystemGroup` and system ordering]()  | Organize the systems of a `World` into a hierarchy that determines the update order of the systems.  |
| **sample: [HelloCube]()** | Displays a rotating cube using `Unity.Transforms` and `Unity.Rendering.Hybrid`.  |
|  [subscenes and conversion](api_subscenes.md)| Create entities from scenes. |
|  [`EntityCommandBuffer`](api_entity_comand_buffer.md)  | Record structural changes for playback later.  |
|  [`IJobChunk`](api_IJobChunk.md)  | Access entities in a job.  |
|  [`Entities.ForEach`](api_entities_foreach.md)  | A more convenient (but less powerful) way to create and schedule `IJobChunk` jobs. |
|  [`World`]()  |  A World is comprised of a set of entities and a set of system instances.  |  
|  [shared components]()  |    |
|  [chunk components]()  |    |
|  [system state components]()  |    |
|  [dynamic buffer components]()  |    |
|  [blob assets]()  |    |
|  [versioning]()  |    |
|  [write groups]()  |    |
| [fixed time step]() |
| [entity transactions]()  |    |
