---
title: Enums as name
---
<h3>Enums</h3>
Technically, enum type is a set of **predefined constants**.

It is considered a good practice to use them as set of options.
However, many things in Unreal are usable with strings only,
take sockets for example.

Let's imagine that you have an actor that does some stuff related to components and sockets.
You decide to create enum:
```c++
UENUM(BlueprintType)
enum class EMyComponentSocket : uint8
{
    BottomSocket,
    MiddleSocket,
    TopSocket,
};
```
Now something needs to be attached to a socket named **TopSocket**. Also, said enum must be used as map key in blueprint properties.
So, you want to use enum for both cases and avoid hardcoded string, but don't know how.

<h3>GetAuthoredNameStringByValue()</h3>
The best way to do this is to use **authored** functions.

These functions return the unlocalized logical name originally assigned to the enum at creation.
By default it is the short name, but it could be overriden in child classes to return some internal names.
Additionally, returned name remains identical in both cooked and editor builds.
Authored functions guarantee to return the same name every time.
```c++
FString EnumString = StaticEnum<EMyComponentSocket>()->GetAuthoredNameStringByValue(static_cast<int64>(EMyComponentSocket::TopSocket));

FName EnumName = FName(*StaticEnum<EMyComponentSocket>()->GetAuthoredNameStringByValue(static_cast<int64>(EMyComponentSocket::TopSocket)));

FString FoundEnumName;
bool EnumNameWasFound = StaticEnum<EMyComponentSocket>()->FindAuthoredNameStringByValue(FoundEnumName, static_cast<int64>(EMyComponentSocket::TopSocket));

//All return "TopSocket"
```

Also, there are non-authored versions. You should **AVOID** them when you want to use your enums as strings, as they might be changed by reflection.
```c++
StaticEnum<EMyComponentSocket>()->GetNameStringByValue(static_cast<int64>(EMyComponentSocket::TopSocket));

//Don't use it
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