---
title: Enums as name
---
<h3>Enums</h3>
Technically, enum type is a set of **predefined constants**.

So it is a good practice, that you use it as set of options.
But there is a problem in Unreal, that a lot of things could be used only with strings.
For example, sockets.

Imagine, you have actor that do some stuff related do components and sockets.
So you create enum:
```c++
UENUM(BlueprintType)
enum class EMyComponentSocket : uint8
{
    BottomSocket,
    MiddleSocket,
    TopSocket,
};
```
And now you need attach something to socket with name **TopSocket**. Also, you would like to use name as map key in blueprints properties. So you want to use enum, instead of hardcoded string.
But how to?

<h3>GetAuthoredNameStringByValue()</h3>
The best way to do it - to use **authored** functions.

They returns the unlocalized logical name originally assigned to the enum at creation.
By default it is short name, but it could be overriden in child classes with some internal names.
This name is consistent in cooked and editor builds.
So it will guarantee that it wil returns same name whole time.
```c++
FString EnumString = StaticEnum<EMyComponentSocket>()->GetAuthoredNameStringByValue(static_cast<int64>(EMyComponentSocket::TopSocket));

FName EnumName = FName(*StaticEnum<EMyComponentSocket>()->GetAuthoredNameStringByValue(static_cast<int64>(EMyComponentSocket::TopSocket)));

FString FoundEnumName;
bool EnumNameWasFound = StaticEnum<EMyComponentSocket>()->FindAuthoredNameStringByValue(FoundEnumName, static_cast<int64>(EMyComponentSocket::TopSocket));

//They all returns "TopSocket"
```

Also there are also no-authored versions. But you should **AVOID** them, if you want to use your enums as strings, because they could be changed by reflection.
```c++
StaticEnum<EMyComponentSocket>()->GetNameStringByValue(static_cast<int64>(EMyComponentSocket::TopSocket));

//You can never be sure that it will return the same value in build and editor
```

<h3>Examples</h3>
```c++
FName SocketName = FName(*StaticEnum<EMyComponentSocket>()->GetAuthoredNameStringByValue(static_cast<int64>(EMyComponentSocket::TopSocket)));

MyComponent->AttachToComponent(RootComponent, FAttachmentTransformRules::KeepRelativeTransform, SocketName);
```

```c++
UPROPERTY(EditDefaultsOnly, Category = "Sockets")
TMap<EMyComponentSocket, TSubclassOf<AActor>> SocketsActors;

for (TTuple<EMyComponentSocket, TSubclassOf<AActor>> SocketsActor : SocketsActors)
{
    AActor* SpawnedActor = GetWorld()->SpawnActor(SocketsActor.Value);
    
    FName SocketName = FName(*StaticEnum<EMyComponentSocket>()->GetAuthoredNameStringByValue(static_cast<int64>(EMyComponentSocket::TopSocket)));
    SpawnedActor->AttachToActor(this, FAttachmentTransformRules::SnapToTargetNotIncludingScale, SocketName);
}
```

And many many others.