---
title: Load soft objects with StreamableManager
---
<h3>Soft Object</h3>
I suppose that you know what soft object is, but if no, you could read it [here](https://dev.epicgames.com/community/learning/tutorials/kx/unreal-engine-all-about-soft-and-weak-pointers)

The main (but not only one) difference:
- hard refs are loaded in memory
- soft refs are not loaded, but could be loaded by yourself

<h3>Loading</h3>
There are two ways how to load your object. Synchronous and Asynchronous.

If you are loading data on initialization, for example at game start, synchronous loading probably will be the best option. It is easy and simple.
But if you need to load anything during gameplay, you should use asynchronous loading. Because synchronous loading will stop thread, and your game will stutter.

Imagine two cases.
- You have **HUD** widget in your game. Player always should see this HUD, so it is ok to load it synchronous and have it in memory.
- You have tutorial widget in your game. Player probably will see tutorial for first few hours. But after it player could never open a tutorial in menu.

Only two cases, but I hope you get the idea. These assets should be loaded synchronously and asynchronously respectively.
You can't start game without loaded **HUD** widget. And you can start game without tutorial widget, to use your RAM for something more important.

If you want load synchronous, you can use ***LoadSynchronous()*** method.

```c++
TSubclassOf<MyClass> LoadedClass = MySoftClass.LoadSynchronous();
```
This will load your class in memory. If it is big class and there is action in your game, that's probably will stutter.

But we can avoid it with **StreamableManager**.
You can access it via:
```c++
FStreamableManager& StreamableManager = UAssetManager::GetStreamableManager();
```

It is native class for managing streaming assets in and keeping them in memory.

By the way, you could use ***RequestSyncLoad()*** if you want to.
But we will focus on ***RequestAsyncLoad()***. 
The main purpose of this method is ability to bind lambda to it.
```c++
FStreamableManager& StreamableManager = UAssetManager::GetStreamableManager();
StreamableManager.RequestAsyncLoad(MySoftClass.ToSoftObjectPath(), [this] {
    GetWorld()->SpawnActor(TestSubclass.Get());
});
```

Also, you could define priority of loading with **TAsyncLoadPriority**.
0 is default, and 100 is high priority.
With this, you could create your own loading sequence.
So if you need load something with high priority, just pass parameter:
```c++
//you don't need to store reference to StreamableManager, you can use it as singleton
UAssetManager::GetStreamableManager().RequestAsyncLoad(MySoftClass.ToSoftObjectPath(), [this] {
    GetWorld()->SpawnActor(TestSubclass.Get());
}, FStreamableManager::AsyncLoadHighPriority);
```

With this, lambda-callback will be called after asset will be loaded.
If asset is in memory already, callback will be called immediately.

<h3>Conclusion</h3>
Avoiding hard references is best-practice.
But to use soft objects properly, you should care how and when load them.

Load synchronous that you 100% need to load before your gameplay. 

Load asynchronous anything you probably will be need during gameplay, but don't need to store it all the time, because there is probability that your player could never need this.
And if will, don't make player struggle with stutter.