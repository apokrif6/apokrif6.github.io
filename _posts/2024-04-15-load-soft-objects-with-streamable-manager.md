-
title: Load soft objects with StreamableManager
---
<h3>Soft Object</h3>
This post assumes understanding of soft objects, you can read more about them [here](https://dev.epicgames.com/community/learning/tutorials/kx/unreal-engine-all-about-soft-and-weak-pointers)

The main difference between hard and soft references:
- hard refs are automatically loaded into memory
- soft refs need to be loaded manually into memory

<h3>Loading</h3>
There are two ways how your object can be loaded. It is either Synchronous or Asynchronous loading.

In case you are loading data during initialization, for example at game start, synchronous loading probably will be the best option. It is easy and simple.
However, when you need to load anything during gameplay, you should be using asynchronous loading. Otherwise, synchronous loading will stall the thread, causing stutters to your game.

Consider the following cases:
- You have **HUD** widget in your game. Player always should see this HUD, so it is okay to load it synchronously and have it in memory.
- You have tutorial widget in your game. Player probably will only see tutorial for first few hours. After that, player can never open a tutorial in menu.

Only two cases, but I hope you get the idea. These assets should be loaded synchronously and asynchronously respectively.
You have to start the game with loaded **HUD** widget, while it is benefical to start the game without tutorial widget, so that your RAM can be used for something more important.

For synchronous loading, you can use ***LoadSynchronous()*** method.

```c++
TSubclassOf<MyClass> LoadedClass = MySoftClass.LoadSynchronous();
```
This will load your class into memory. If it is a large class and there is an action in your game, that will probably stutter.

We can avoid this with **StreamableManager**.
You can access it via:
```c++
FStreamableManager& StreamableManager = UAssetManager::GetStreamableManager();
```

It is a native class that manages asset streaming and keeping them in memory.

By the way, you can use ***RequestSyncLoad()*** if you want to.
But we will focus on ***RequestAsyncLoad()***. 
The main purpose of this method is to bind lambda to it.
```c++
FStreamableManager& StreamableManager = UAssetManager::GetStreamableManager();
StreamableManager.RequestAsyncLoad(MySoftClass.ToSoftObjectPath(), [this] {
    GetWorld()->SpawnActor(TestSubclass.Get());
});
```

Also, you can specify loading priority with **TAsyncLoadPriority**.
0 is default, and 100 is high priority.
With this, you can create your own loading sequence.
For example, when you need to load something with high priority, just pass a parameter:
```c++
//you don't need to store reference to StreamableManager, you can use it as singleton
UAssetManager::GetStreamableManager().RequestAsyncLoad(MySoftClass.ToSoftObjectPath(), [this] {
    GetWorld()->SpawnActor(TestSubclass.Get());
}, FStreamableManager::AsyncLoadHighPriority);
```

With this, lambda-callback will be called after asset is loaded.
If asset is in memory already, callback will be called immediately.

<h3>Conclusion</h3>
Avoiding hard references is considered a best practice.
But to use soft objects properly, you should care about how and when to load them.

Load synchronously something that you 100% need to be loaded before your gameplay. 

Load asynchronously anything you will probably need during gameplay, but don't need to store all the time, because there is probability that your player will never need this.
And in case player will need it, don't make them struggle with stutter.