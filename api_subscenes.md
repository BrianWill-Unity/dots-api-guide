# using subscenes
<!-- 
> Topics to add
> * 
-->

Scenes cannot directly include entities, but we can create entities upon scene load using **subscenes**.

A `SubScene` is defined by a GameObject in a scene which has a `SubScene` component that references another scene asset. This other scene asset lists everything that makes up the subscene.

At build time, conversion systems process each subscene, creating an entity 'equivalent' for each GameObject of the subscene. These build-time entities are serialized and then deserialized into the World at runtime when the scene loads.

The correspondance between a subscene GameObject and the entity 'equivalent' produced from it is flexible: for any GameObject, we can customize the conversion logic to produce as many entities as we like with whatever set of entity components as we like.