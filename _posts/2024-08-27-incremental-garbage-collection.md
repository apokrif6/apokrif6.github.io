---
title: Incremental Garbage Collection
---
<h3>Be aware</h3>
It is 5.4, and it is experimental feature. So learn about it, try it. But remember that feature is not production ready.

<h3>Problem</h3>
Unreal Engine uses `Garbage Collector` to manage memory usage of `UObject`. But there is one major issue with it.

`Garbage Collection` runs every X seconds. But before "collect" memory, `GC` should define which objects should be collected.
It is called `reachability analysis`. The main idea is check references, and mark objects which should be cleaned.
But if you have a lot of objects loaded, this process may take a while, or even longer. This leads to lags or stutters.

There are few ways to avoid this problem, disabling `Garbage Collection` during complex gameplay, pooling objects, etc.

<h3>Incremental Reachability Analysis</h3>
But 5.4 introduces new technique. Now we can split `reachability analysis` across multiple frames, instead of running analysis in single frame.


First of all, it doesn't work with `UPROPERTY` and raw pointers. If you want use this feature, you should use `TObjectPtr` (if you don't know what is it, read [here](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-5-migration-guide))

As soon as you mark `TObjectPtr` as `UPROPERTY`, it becomes trackable by engine.
Unreal's source code is almost full compatible with `TObjectPtr`, so you should prefer them to raw pointers.

Secondly, to enable `IGC`, add to your `DefaultEngine.ini`:

```c++
[ConsoleVariables]
gc.AllowIncrementalReachability=1 ; enables Incremental Reachability Analysis
gc.AllowIncrementalGather=1 ; enables Incremental Gather Unreachable Objects
gc.IncrementalReachabilityTimeLimit=0.002 ; sets the soft time limit to 2ms;
```

<h3>Performance</h3>
On this screen from Unreal Insight, we see how `analysis` is split across few frames, and there is no spike.
And that's how we could avoid hitches with this feature.

![Performance](https://apokrif6.github.io/assets/images/incremental_garbage_collection/unreal_insight_test.png)
