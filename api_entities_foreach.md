# `Entities.ForEach`
<!-- 
> Topics to add
> * 
-->

Systems have a property `Entities` with a method `ForEach`. Through some special-to-Unity compilation post-processing of the Intermediate Language, `ForEach` generates a query and a job to process entities of the query.

## defining the query

The query is defined primarily by the parameters of the lambda passed to `ForEach`. Component parameters with the `ref` keyword are readwrite. Component parameters with the `in` keyword are readonly. (As a rule, any `ref` parameters must precede any `in` parameters in the list.)

```csharp
// A query matching Mass, Speed, and (readonly) Health.
Entities.ForEach( (ref Mass, ref Speed, in Health) => {
    // ... Body of lambda processes each entity.
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
    .WithAll<Mass>()    // Mass required by query, but we don't need to read or write Mass.
    .ForEach( (ref Speed, in Health) => {
    // ...
})
```

Note above that `WithAll()` returns the same type as `Entities` and `ForEach` do, so we chain the method calls. Many other `ForEach`-related methods follow this pattern.

To exclude entities with a particular component, use `WithNone()`:

```csharp
Entities
    .WithNone<Mass>()    // Query will not match entities with Mass.
    .ForEach( (ref Speed, in Health) => {
    // ...
})
```

## schedule as a job

The value returned by `ForEach` represents a query and code to run on each entity of the query. From this, we can schedule a single-threaded job with `Schedule()`:

```csharp
// Scheduled as a job.
Entities.ForEach( (ref Mass, in Health) => {
    // ...
}).Schedule();
```

If instead we call `ScheduleParallel()`, the job is split into batches (one per chunk matching the query) and so may run parallelized across threads.

```csharp
// Scheduled as a batched job.
Entities.ForEach( (ref Mass, in Health) => {
    // ...
}).ScheduleParallel();
```

In a parallel job, we must use `EntityCommandBuffer.ParallelWriter` instead of a regular `EntityCommandBuffer`. (See [here](api_entity_command_buffer.md#using-EntityCommandBufferSystem).)

## run on the main thread

We also can call `Run()` to simply run the code on the main thread, in which case we can also use `WithStructuralChanges()` to allow the lambda to directly make structural changes (but also disables Burst compilation). (Structural changes invalidate the chunk iterator produced from the query, so `ForEach` must do some extra compilation work to accomodate structural changes, even when the lambda is run on the main thread.)

```csharp
// Will be scheduled as a batched job.
Entities
    .WithStructuralChanges()
    .ForEach( (Entity entity, in Health) => {
        if (Health.Value <= 0) {
            EntityManager.DestroyEntity(entity);  // make a structural change
        }
}).Run();
```

## disabling Burst compilation

By default, a `ForEach` is Burst compiled, even if run on the main thread (unless using `WithStructuralChanges()`). When Burst compiled, the lambda can use instance fields and static methods of the enclosing system (but not static fields or instance methods).

The `WithoutBurst()` method disables Burst compilation of the `ForEach`:

```csharp
Entities
    .WithoutBurst()
    .ForEach( (Entity entity, ref Mass) => {
        // ... Code is not burst compiled.
}).Run();
```

## dispose containers upon job completion

If a `ForEach` reads from NativeContainers, we often want the NativeContainers to be disposed once the job completes. To do this, we can use `WithDisposeOnCompletion`:

```csharp
var arr = NativeArray<int>(100, Allocator.TempJob);
// ... Populate the array.

Entities
    .WithDisposeOnCompletion(arr)
    .ForEach( (Entity entity, ref Mass) => {
        // ... Use the array.
}).Schedule();
```

(If a `ForEach` *writes* to a NativeContainer, it generally doesn't make sense to dispose it upon completion because presumably some other code should *read* from the container after the job completes.)

## give the ForEach a name for more readable errors

The `WithName()` method gives the `ForEach` a name to make exception messages more readable:

```csharp
Entities
    .WithoutName("Update Mass")
    .ForEach( (Entity entity, ref Mass) => {
        // ...
}).Schedule();
```