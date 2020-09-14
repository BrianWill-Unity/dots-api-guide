# `EntityQuery`
<!-- 
> Topics to add
> * 
-->

An `EntityQuery` let's us efficiently get all chunks which have (or don't have) a set of specified component types.

Queries can also filter on two other criteria:

- The value of shared components.
- Whether values of specified component values may have changed since the last run of the query.

## **creating queries**

To create an `EntityQuery`:

```csharp
// From an EntityManager, create a query matching all chunks with components Foo and Bar.
EntityQuery query = manager.CreateEntityQuery(typeof(Foo), typeof(Bar));
```

For a query with more complex criteria, we need an `EntityQueryDesc`, which has three separate arrays of component types:

```csharp
EntityQueryDesc desc = new EntityQueryDesc {
    // Query only matches chunks with both Red and Green components.
    All = new ComponentTypes[] {typeof(Red), typeof(Green)},     

    // Query also only matches chunks with at least one of Purple, Orange, or Pink components.
    Any = new ComponentTypes[] {typeof(Purple), typeof(Orange), typeof(Pink)},

    // Query does not match chunks with a Blue component.
    None = new ComponentTypes[] {typeof(Blue)},
};
EntityQuery query = manager.CreateEntityQuery(desc);
```

We generally want the queries used in a system to be registered with a system, so we should usually instead create queries *via* `GetEntityQuery()` of a system instead of *via* the `EntityManager`.

```csharp
// (in a system)
// Retrieves cached query (or creates the query if it didn't exist already).
EntityQuery query = GetEntityQuery(typeof(Foo), typeof(Bar));
```

When we use `CreateEntityQuery()`, we are responsible for disposing of the query. When we use `GetEntityQuery()`, the query is permanently registered with the system, and so the system itself is responsible for disposing of the query.

## get all chunks matching a query

```csharp
// Create an array of all the chunks matching the query.
NativeArray<ArchetypeChunk> chunks = query.CreateArchetypeChunkArray(Allocator.Persistent);

// We are responsible for disposing of the array when done with it.
chunks.Dispose();
```

(An `ArchetypeChunk` is basically just a wrapper around a pointer to a chunk.)