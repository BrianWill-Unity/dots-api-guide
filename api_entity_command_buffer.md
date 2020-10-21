# `EntityCommandBuffer`
<!-- 
> Topics to add
> * 
-->

Because jobs are not usually run on the main thread, a job cannot directly make structural changes. A job can, however, use an `EntityCommandBuffer` to record changes it would like to be made later. After the job completes, the main thread can 'play back' the commands recorded in the job.

We also sometimes might want to record changes on the main thread because:

- It can be more efficient to play back many changes together in one group.
- We want to delay our changes.
- We want to play back our changes multiple times.

A command buffer can be flagged to allow either single playback or multiple playback. Attempting to play back a single-playback buffer more than once throws an exception.

After playback of a command buffer, attempting to record more changes throws an exception.

When recording in a parallelized job, we need a variant called `EntityCommandBuffer.ParallelWriter`, which allows for concurrent recording. (Playback of all command buffers is always single-threaded.)

## creating an `EntityCommandBuffer`

- `new EntityCommandBuffer(Allocator)`
- `new EntityCommandBuffer(Allocator, PlaybackPolicy)`
- `AsParallelWriter()` returns `EntityCommandBuffer.ParlalleWriter`

If we're going to play back and dispose of our command buffer manually, we create it ourselves directly:

```csharp
EntityCommandBuffer ecb = new EntityCommandBuffer(Allocator.TempJob);
```

If, though, we want an `EntityCommandBufferSystem` to play back and dispose the command buffer for us:

```csharp
// (in a system)
// (assume an EntityCommandBufferSystem named MyECBSys)
EntityCommandBufferSystem sys = this.World.GetExistingSystem<MyECBSys>();

// Create a command buffer that will be played back and disposed by MyECBSys.
EntityCommandBuffer ecb = sys.CreateCommandBuffer();
```

To make our command buffer into a `ParallelWriter`:

```csharp
EntityCommandBuffer ecb = new EntityCommandBuffer(Allocator.TempJob);
EntityCommandBuffer.ParallelWriter ecbParallel = ecb.AsParallelWriter();
ecb.Dispose();    // dispose of the underlying EntityCommandBuffer, not the ParallelWriter
```

## creating, destroying, and modifying entities

These methods mirror those of `EntityManager`, but not all `EntityManager` methods have an `EntityCommandBuffer` 'equivalent' because an equivalent isn't possible in some cases (and in other cases an equivalent wouldn't be useful).

Be clear again that none of these methods immediately enact any changes. Instead these methods record an *intention of change* to be enacted later when the `EntityCommandBuffer` is played back.

- `CreateEntity(EntityArchetype)` returns `Entity`
- `DestroyEntity(Entity)`
- `DestroyEntity(EntityQuery)`
- `SetComponent<T>(Entity, T)` 
- `AddComponent<T>(Entity)`
- `AddComponent<T>(Entity, T)` *(add and set initial value)*
- `AddComponent<T>(EntityQuery)`
- `AddComponent(Entity, ComponentType)`
- `AddComponent(EntityQuery, ComponentType)`
- `RemoveComponent<T>(Entity)`
- `RemoveComponent<T>(EntityQuery)`
- `RemoveComponent(Entity, ComponentType)`
- `RemoveComponent(EntityQuery, ComponentType)`

For an `EntityCommandBuffer.ParallelWriter`, all of these methods have an additional `sortKey` parameter, which is necessary to [make playback determinsitic](). An `Entities.ForEach` or `IJobChunk` provides the appropriate `sortKey` value, and we simply need to pass it along:

```csharp
// ForEach provides the sortKey we need
Entities.ForEach( (Entity entity, int sortKey, in Health health) => {
    if (health.Value < 0)
        // assume this is an EntityCommandBuffer.ParallelWriter
        ecb.DestroyEntity(sortKey, entity);  
    }
}).ScheduleParallel();
```

## using `EntityCommandBufferSystem`

Rather than manually play back and dispose a command buffer ourselves, we can have an `EntityCommandBufferSystem` do those things for us:

1. We get the command buffer system we want to do the play back.
2. We create a command buffer *via* the system.
3. We schedule a job that will record to the command buffer.
4. We register the scheduled job with the system.
5. In its next update, the system will complete the job (to ensure the job has finished recording), play back the command buffer, and then dispose of the command buffer.

We can create an `EntityCommandBufferSystem` ourselves like a regular system, except we never override its `OnUpdate()` (because the base `OnUpdate` of an `EntityCommandBufferSystem` already does everything we want it to do).

```csharp
[UpdateInGroup(typeof(SimulationSystemGroup))]
public class MyEntityCommandBufferSystem : EntityCommandBufferSystem
{
    // ... always leave the body empty
}
```

However, we most commonly only need one of the five `EntityCommandBufferSystem`'s created for us in the default world:

- `BeginInitializationEntityCommandBufferSystem`
- `EndInitializationEntityCommandBufferSystem`
- `BeginSimulationEntityCommandBufferSystem`
- `EndSimulationEntityCommandBufferSystem`
- `BeginPresentationEntityCommandBufferSystem`

As their names imply, these are updated at the beginings and ends of each of the three default world top-level system groups: `InitializationSystemGroup`, `SimulationSystemGroup`, and `PresentationSystemGroup`. (The `EndPresentationEntityCommandBufferSystem` was removed because making structural changes related to rendering proved to be too error prone.)

So for example:

```csharp
// (in a system update)
EntityCommandBufferSystem sys = GetExistingSystem<EndSimulationEntityCommandBufferSystem>();
EntityCommandBuffer ecb = sys.CreateCommandBuffer();

// ... record changes
ecb.CreateEntity();
```

Above, we don't play back and dispose of the `EntityCommandBuffer` because the `EntityCommandBufferSystem` will do it for us.

If an `EntityCommandBuffer` created *via* an `EntityCommandBufferSystem` records changes during a job, we don't want the system to play back the buffer until the job has completed (because otherwise it may not have finished recording its changes). We can tell the system to complete the job before playback with `AddJobHandleForProducer()`:

```csharp
// The EntityCommandBufferSystem ('sys') will complete the job ('someJob')
// before playing back its EntityCommandBuffers.
sys.AddJobHandleForProducer(someJob);
```
