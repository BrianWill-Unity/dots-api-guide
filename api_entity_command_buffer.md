---
uid: ecs-api-entity-command-buffer
---
# using `EntityCommandBuffer`
<!-- 
> Topics to add
> * 
-->

Because jobs are not usually run on the main thread, a job cannot directly make structural changes. A job can, however, use an `EntityCommandBuffer` to record changes it would like to be made later. After the job completes, the main thread can 'play back' the commands recorded in the job.

We also sometimes might want to record changes on the main thread because:

- it can be more efficient to play back many changes in one lump
- we want to delay our changes
- we want to play back our changes multiple times

A command buffer can support either single playback or multiple playback. Attempting to playback a single-playback buffer more than once throws an exception.

After playback of a command buffer, attempting to record more changes throws an exception.

When recording in a parallelized job, we need a variant called `EntityCommandBuffer.ParallelWriter`, which allows for concurrent recording. (Playback of all command buffers is always single-threaded.)

Rather than manually play back and dispose a command buffer ourselves, we can have an `EntityCommandBufferSystem` do those things for us:

1. Get the command buffer system we want to do the play back.
2. Create a command buffer *via* the system.
3. Schedule a job that will record to the command buffer.
4. Register the scheduled job with the system.
5. In its next update, the system will complete the job (to ensure the job has finished recording), play back the command buffer, and then dispose of the command buffer.

[why can't (shouldn't) we reuse the same ecb to record across multiple jobs?]


## creating an `EntityCommandBuffer`

- `new EntityCommandBuffer(Allocator)`
- `new EntityCommandBuffer(Allocator, PlaybackPolicy)`
- `AsParallelWriter()` returns `EntityCommandBuffer.ParlalleWriter`
- 

If we're going to play back and dispose of our command buffer manually, we create it ourselves directly:

```
EntityCommandBuffer ecb = new EntityCommandBuffer(Allocator.TempJob);
```

If, though, we want an `EntityCommandBufferSystem` to play back and dispose the command buffer for us:

```
// (in a system)
// (assume an EntityCommandBufferSystem named MyECBSys)
EntityCommandBufferSystem sys = this.World.GetExistingSystem<MyECBSys>();

// Create a command buffer that will be played back and disposed by MyECBSys.
EntityCommandBuffer ecb = sys.CreateCommandBuffer();
```


To make our command buffer into a `ParallelWriter`:

```
EntityCommandBuffer.ParallelWriter ecbParallel = ecb.AsParallelWriter();
```




## creating, destroying, and modifying entities

*These methods mirror those of `EntityManager`, but not all `EntityManager` methods have an `EntityCommandBuffer` 'equivalent' because an equivalent isn't possible in some cases, and in other cases an equivalent wouldn't be useful.*

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

For an `EntityCommandBuffer.ParallelWriter`, all of these methods have an additional `sortKey` parameter, which is necessary to [make play back determinsitic](). An `Entities.ForEach` or `IJobChunk` provides the appropriate sort key value, and we simply need to pass it along:

```


```
