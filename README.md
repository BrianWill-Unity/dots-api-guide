# Guide to the `Unity.Entities` API
<!-- 
> Topics to add
> * 
-->

This guide is meant to walk users through the essential parts of the Entities API. The interspersed samples demonstrate how to put things together.

The guide assumes familiarity with the [basic Entities concepts](), and some details are left to the [API reference]().

|||
| --- | --- |
|  [setting up a DOTS project](https://docs.google.com/document/d/13vXB43Rkleg-l4c0-MBh3oFW3b-wmhUoFCxVXp1KHjo/edit?usp=sharing)  | Create a DOTS project with the most commonly needed packages and settings.|
|  [`EntityManager`](api_entity_manager.md)  | Create and destroy entities. Also add and remove the components of entities.| 
|  [`EntityQuery`](api_entity_query.md)  | Find all entities with a specified set of component types.  | 
|  [`SystemBase`](api_systems.md)  | Systems are the core units of code in ECS. |
|  [`EntityCommandBuffer`](api_entity_command_buffer.md)  | Record intended changes to entities and their components for playback later.|
|  [`Entities.ForEach`](api_entities_foreach.md)  | A more convenient way to create and schedule jobs that read and write entities.|
|  [subscenes and conversion](api_subscenes.md)| Create entities from scenes. |
| **sample: [HelloCube]()** | Displays a rotating cube using `Unity.Transforms` and `Unity.Rendering.Hybrid`.|


<!--

|||
| --- | --- |
|  [`IJobChunk`](api_IJobChunk.md)  | Access entities in a job.  |
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


-->
