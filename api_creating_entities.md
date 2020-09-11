---
uid: ecs-api-entity-manager
---
# using `EntityManager`
<!-- 
> Topics to add
> * 
-->

A World's `EntityManager` has methods to create or destroy entities in the World and to add or remove their components. Methods which make such 'structural changes' can only be called from the main thread.

In many cases, rather than use an `EntityManager` directly, it is better to create entities and make structural changes by using an `EntityCommandBuffer` or by loading a **subscene**. An `EntityCommandBuffer` lets us record structural changes in jobs to be later 'played back' *via* an `EntityManager`; a subscene lets us define entities in scenes and lets us efficiently create many entities upon scene load.

Also be aware that adding or removing components of an existing entity changes its archetype, meaning the entity gets moved to a different chunk. While this is a trivial expense for a small number of entities, moving *many* entities per frame should generally be avoided when possible.

## **getting an EntityManager**

The `EntityManager` property of a `World` returns its `EntityManager`.

In a system, we should get an `EntityManager` *via* the system itself to ensure we get the `EntityManager` of the `World` to which the system belongs:

```csharp
// (in a system)
EntityManager manager = this.EntityManager;
```

## **creating archetypes**

To create an `EntityArchetype`, which represents a set of component types:

```csharp
// Creates an EntityArchetype consisting of components Foo and Bar
EntityArchetype archetype = manager.CreateArchetype(typeof(Foo), typeof(Bar));
```

## **creating queries**

To create an `EntityQuery`:

```csharp
// Creates a query matching all chunks with components Foo and Bar.
EntityQuery query = manager.CreateEntityQuery(typeof(Foo), typeof(Bar));
```

We generally, though, want the queries used in a system to be registered with the system, so we should usually instead create queries with `GetEntityQuery()` of the system instead of *via* the `EntityManager`.

## **creating entities**

- `CreateEntity()` returns `Entity`
- `CreateEntity(params ComponentType[])` returns `Entity`
- `CreateEntity(EntityArchetype)` returns `Entity`
- `CreateEntity(EntityArchetype, int, Allocator)` returns `NativeArray<Entity>`
- `CreateEntity(EntityArchetype, NativeArray<Entity>)`

To create a new entity with no components:

```csharp
// Create a new entity with no components.
Entity e = manager.CreateEntity();
```

To create a new entity with a set of components, pass any numuber of `ComponentType` arguments:

```csharp
// Create a new entity with components Foo and Bar.
// (The Type arguments get coerced into ComponentType values.)
Entity e = manager.CreateEntity(typeof(Foo), typeof(Bar));
```

We can also pass an `EntityArchetype`, which is convenient if we already have one handy:

```csharp
EntityArchetype archetype = manager.CreateArchetype(typeof(Foo), typeof(Bar));

// Create a new entity with components Foo and Bar.
Entity e = manager.CreateEntity(types);
```

To create multiple entities with a set of components:

```csharp
EntityArchetype archetype = manager.CreateArchetype(typeof(Foo), typeof(Bar));

// Create 5 new entities.
// We are responsible for disposing of the array when we're done with it.
NativeArray<Entity> ents = manager.CreateEntity(archetype, 5, Allocator.Temp);
```

Alternatively, if we already have a `NativeArray<Entity>`, we can pass it to be filled:

```csharp
EntityArchetype archetype = manager.CreateArchetype(typeof(Foo), typeof(Bar));

// We are responsible for disposing of the array when we're done with it.
NativeArray<Entity> ents = new NativeArray<Entity>(5, Allocator.Temp);

// Create 5 new entities (because the array length is 5).
// The new entity id's are stored in the array.
manager.CreateEntity(archetype, ents);
```

## **getting and setting component values**

*While it is possible to get and set components directly through `EntityManager`, it is usually not optimal or appropriate to do so. Instead, we should more commonly get and set components in jobs iterating over the entities matching a query. This is covered elsewhere with `IJobChunk` and `Entities.ForEach`.*

- `GetComponentData<T>(Entity)` returns `T`
- `SetComponentData<T>(Entity, T)`

To get the value of an entity's component:

```csharp
Entity e = manager.CreateEntity(typeof(Foo));

// Get the Foo component value of the entity.
Foo foo = manager.GetComponentData<Foo>(e);
```

To set the value of an entity's component:

```csharp
Entity e = manager.CreateEntity(typeof(Foo));
Foo foo;
// ...(not shown) init the value of foo

// Set the Foo component value of the entity.
manager.SetComponentData<Foo>(e, foo);
```

## **checking if an entity has a component**

- `HasComponent<T>()` returns `bool`

```csharp
Entity e = manager.CreateEntity(typeof(Foo));

// Will return true.
bool hasFoo = manager.HasComponent<Foo>(e);

// Will return false.
bool hasBar = manager.HasComponent<Bar>(e);
```

If the component type that we want to check for is only known at runtime, we can use `ComponentType`:

```csharp
Entity e = manager.CreateEntity(typeof(Foo));

// Will return true.
// (The Type will be coerced to ComponentType)
bool hasFoo = manager.HasComponent(e, typeof(Foo));

// Will return false.
// (The Type will be coerced to ComponentType)
bool hasBar = manager.HasComponent(e, typeof(Bar));
```

## **copying entities**

*Copying entities from one World to another is covered with Worlds.*

- `Instantiate(Entity)` returns `Entity`
- `Instantiate(Entity, int, Allocator)` returns `NativeArray<Entity>`
- `Instantiate(Entity, NativeArray<Entity>)`
- `Instantiate(NativeArray<Entity>, NativeArray<Entity>)`
- `CopyEntities(NativeArray<Entity>, NativeArray<Entity>)`

To make a copy of an entity:

```csharp
Entity e = manager.CreateEntity(typeof(Foo));

// Creates a new entity with all the same components and values as the original.
Entity copy = manager.Instantiate(e);
```

(All variants of `Instantiate()` do not copy the special `Prefab` component; all other components are copied as normal.)


To make multiple copies of an entity:

```csharp
Entity e = manager.CreateEntity(typeof(Foo));

// Create 5 new entities that are all copies of e.
// We are responsible for disposing of the array when we're done with it.
NativeArray<Entity> ents = manager.Instantiate(e, 5, Allocator.Temp);
```

Alternatively, if we already have a `NativeArray<Entity>`, we can pass it to be filled:

```csharp
Entity e = manager.CreateEntity(typeof(Foo));

// We are responsible for disposing of the array when we're done with it.
NativeArray<Entity> ents = new NativeArray<Entity>(5, Allocator.Temp);

// Create 5 new entities that are all copies of e.
// The new entity id's are stored in the array.
manager.Instantiate(e, ents);
```

To make a single copy of multiple entities:

```csharp
NativeArray<Entity> originals = new NativeArray<Entity>(5, Allocator.Temp);
// ...(not shown) init originals

NativeArray<Entity> copies = new NativeArray<Entity>(5, Allocator.Temp);

// Make one copy each of the entities in the originals array. 
// The new entity id's are stored in the copies array.
manager.Instantiate(originals, copies);
```

The `CopyEntities(NativeArray<Entity>, NativeArray<Entity>)` method works like `Instantiate(NativeArray<Entity>, NativeArray<Entity>)`, except it will...

- ...copy the `Prefab` component.
- ...*not* copy system state components.
- ...modify `Entity` values stored in the components: a reference to an entity in the source array will be changed to reference its corresponding new clone.

## **destroying entities**

- `DestroyEntity(Entity)`
- `DestroyEntity(NativeArray<Entity>)`
- `DestroyEntity(NativeSlice<Entity>)`
- `DestroyEntity(EntityQuery)`

To destroy a single entity:

```csharp
Entity e = manager.CreateEntity();

// Destroy the entity.
manager.DestroyEntity(e);
```

To destroy multiple entities, specified by id's in an array:

```csharp
EntityArchetype archetype = manager.CreateArchetype(typeof(Foo), typeof(Bar));
NativeArray<Entity> ents = manager.CreateEntity(archetype, 5, Allocator.Temp);

// Destroy all entities of the array.
manager.DestroyEntity(ents);
```

We can also use a `NativeSlice<Entity>`. (A slice represents a range of indexes within an array. Be careful not to use a slice once its underlying array has been disposed.):

```csharp
EntityArchetype archetype = manager.CreateArchetype(typeof(Foo), typeof(Bar));
NativeArray<Entity> ents = manager.CreateEntity(archetype, 100, Allocator.Temp);

// A slice representing indexes 20 through 29 of the array.
NativeSlice<Entity> slice = new NativeSlice<Entity>(ents, 20, 10)

// Destroy all entities of the slice.
manager.DestroyEntity(slice);
```

Lastly, we can destroy all entities matching a query:

```csharp
EntityQuery query = manager.CreateEntityQuery(typeof(Foo), typeof(Bar));

// Destroy all entities having both Foo and Bar components.
manager.DestroyEntity(query);
```

## **adding and removing components**

*This page only covers adding and removing regular (IComponentData) components. All other types of components are covered elsewhere.*

- `AddComponent<T>(Entity)` returns `bool`
- `AddComponentData<T>(Entity, T)` returns `bool`
- `AddComponent<T>(NativeArray<Entity>)`
- `AddComponent<T>(EntityQuery)`
- `AddComponentData<T>(EntityQuery, NativeArray<Entity>)`
- `RemoveComponent<T>(Entity)` returns `bool`

When the component type to add or remove is known only at runtime, we can specify a `ComponentType` argument instead of a type parameter:

- `AddComponent(Entity, ComponentType)`
- `AddComponent(NativeArray<Entity>, ComponentType)`
- `AddComponent(EntityQuery, ComponentType)`
- `RemoveComponent(Entity, ComponentType)`
- `RemoveComponent(NativeArray<Entity>, ComponentType)`
- `RemoveComponent(EntityQuery, ComponentType)`

*(It doesn't matter for these methods whether a `ComponentType` is ReadWrite or ReadOnly. Also remember that `Type` can be coerced into `ComponentType`, *e.g.* `typeof(Foo)` can be written instead of `ComponentType.ReadWrite<Foo>()`.)*

For adding and removing multiple components:

- `AddComponent(Entity, ComponentTypes)`
- `RemoveComponent(EntityQuery, ComponentTypes)`
- `SetArchetype(e, archetype)`

*(The set of methods for multiple components probably seems incomplete. For instance, there is no `AddComponents(EntityQuery, ComponentTypes)`. These missing methods might be added in the future.)*


In the simplest case, we wish to add or remove a single component from a single entity, and the type is known at compile time:

```csharp
Entity e = manager.CreateEntity();

// Adds a Foo component to the entity.
// Returns false if the entity already has a Foo component.
bool added = manager.AddComponent<Foo>(e);

// Removes the Foo component from the entity.
// Returns false if the entity already has no Foo component.
bool removed = manager.RemoveComponent<Foo>(e);
```

We can specify an initial value for a component when adding it to an entity:

```csharp
Entity e = manager.CreateEntity();
Foo foo;
// ...(not shown) init value of foo

// Add Foo component with this specified value to the entity.
// Returns false if the entity already has a Foo component.
// The Foo value is still set regardless.
bool added = manager.AddComponentData<Foo>(e, foo);
```

We can add or remove a single component from multiple entities using a `NativeArray<Entity>`:

```csharp
EntityArchetype archetype = manager.CreateArchetype(typeof(Foo));
NativeArray<Entity> ents = manager.CreateEntity(archetype, 100, Allocator.Temp);

// Add a Bar component to all entities of the array.
// OK if some or all of the entities already have a Bar component.
manager.AddComponent<Bar>(ents);

// Remove the Bar component from all entities of the array.
// OK if some or all of the entities already have no Bar component.
manager.RemoveComponent<Bar>(ents);
```

We can also add or remove a component from all entities matching an `EntityQuery`:

```csharp
EntityQuery query = manager.CreateEntityQuery(typeof(Foo));

// Add the Bar component to all entities matching the query.
// OK if some or all of the entities already have a Bar component.
manager.AddComponent<Bar>(query);

// Remove the Bar component from all entities matching the query.
// OK if some or all of the entities already have no Bar component.
manager.RemoveComponent<Bar>(query);
```

When adding a component to the entities matching a query, we can specify their intial values with an array:

```csharp
EntityQuery query = manager.CreateEntityQuery(typeof(Foo));

// We are responsible for disposing of the array when we're done with it.
NativeArray<Bar> bars = new NativeArray<Bar>(100, Allocator.Temp);
// ...(not shown) init the values of bars

// The number of entities matching the query and the length of the array must be the same.
// Add a Bar component to each entity, using the values from the Bar array.
// Any entity which already has a Bar component gets its Bar value set.
manager.AddComponentData<Bar>(query, bars);
```

To add multiple components to a single entity:

```csharp
Entity e = manager.CreateEntity();
ComponentTypes types = new ComponentTypes(typeof(Foo), typeof(Bar));

// add Foo and Bar components to the entity
manager.AddComponent(e, types);
```

To remove multiple components from all entities matching a query:

```csharp
EntityQuery query = manager.CreateEntityQuery(typeof(Foo), typeof(Bar));
ComponentTypes types = new ComponentTypes(typeof(Foo), typeof(Bar));

// remove Foo and Bar components from all entities matching the query
manager.RemoveComponents(query, types);
```

Lastly, we can directly set the archetype of an entity:

```csharp
Entity e = manager.CreateEntity(typeof(A), typeof(B));
EntityArchetype archetype = manager.CreateArchetype(typeof(A), typeof(C));

// Remove component B and add component C.
// The data of component A is unchanged.
manager.SetArchetype(e, archetype);
```


## **accessing 'singleton' entities**




