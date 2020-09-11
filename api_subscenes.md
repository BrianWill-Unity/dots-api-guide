---
uid: ecs-api-subscenes
---
# using subscenes
<!-- 
> Topics to add
> * 
-->

Scenes cannot directly include entities, but we can create entities upon scene load using **subscenes**. A `SubScene` GameObject component logically nests a scene asset inside another scene. At build time, conversion systems process each subscene, creating an entity 'equivalent' for each GameObject of the subscene. These build-time entities are serialized and then deserialized into the World [TODO can we control which World?] at runtime when the scene loads.

The correspondance between a subscene GameObject and the entity 'equivalent' produced from it is flexible: for any GameObject, we can customize the conversion logic to produce as many entities as we like with whatever set of entity components as we like.