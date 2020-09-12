# `Entities.ForEach`
<!-- 
> Topics to add
> * 
-->

Systems have a property `Entities` with a method `ForEach`. Through some special-to-Unity compilation post-processing of the Intermediate Language, `ForEach` generates a query and a job to process entities of the query.

## defining the query

The query is defined primarily by the parameters of the lambda passed to `ForEach`. Component parameters with the `ref` keyword are readwrite. Component parameters with the `in` keyword are readonly. (As a rule, any `ref` parameters must precede any `in` parameters in the list.)

```csharp
// a query matching Mass, Speed, and (readonly) Health
Entities.ForEach( (ref Mass, ref Speed, in Health) => {
    // ... body of lambda processes each entity
})
```

If we need the id's of the entities, we can add `Entity` as the first parameter:

```csharp
Entities.ForEach( (Entity entity, ref Mass, ref Speed, in Health) => {
    // ...
})
```

For components which should be part of the query but which we don't need to read or write, use `WithAll()`:

```csharp
Entities
    .WithAll<Mass>()    // Mass required by query, but we don't need to read or write Mass
    .ForEach( (ref Speed, in Health) => {
    // ...
})
```

Note above that `WithAll()` returns the same type as `Entities` and `ForEach` do, so we chain the method calls. Many other `ForEach`-related methods follow this pattern.

To exclude entities with a particular component, use `WithNone()`:

```csharp
Entities
    .WithNone<Mass>()    // query will not match entities with Mass
    .ForEach( (ref Speed, in Health) => {
    // ...
})
```

## schedule as a job

The value returned by `ForEach` represents a query and code to run on each entity of the query. From this, we can schedule a single-threaded job with `Schedule()`:

```csharp
// scheduled as a job
Entities.ForEach( (ref Mass, in Health) => {
    // ...
}).Schedule();
```

If instead we call `ScheduleParallel()`, the job is split into batches (one per chunk matching the query) and so may run parallelized across threads.

```csharp
// scheduled as a batched job
Entities.ForEach( (ref Mass, in Health) => {
    // ...
}).ScheduleParallel();
```

In a parallel job, we must use `EntityCommandBuffer.ParallelWriter` instead of a regular `EntityCommandBuffer`. (See [here](api_entity_command_buffer.md#using-EntityCommandBufferSystem).)

## run on the main thread

We also can call `Run()` to simply run the code on the main thread, in which case we can also use `WithStructuralChanges()` to allow the lambda to directly make structural changes. (Even when the lambda code is not run in a job, structural changes still would invalidate the chunk iterator produced from the query, so `ForEach` must do some extra compilation work to accomodate structural changes.)

```csharp
// will be scheduled as a batched job
Entities
    .WithStructuralChanges()
    .ForEach( (Entity entity, in Health) => {
        if (Health.Value <= 0) {
            EntityManager.DestroyEntity(entity);  // make a structural change
        }
}).Run();
```




