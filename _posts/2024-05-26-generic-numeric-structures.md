---
title: Generic numeric structures
---
<h3>Problem</h3>
There are a lot of cases, when you need to use some common structures with same pattern.

For example, if you are working on Ability System with magic spells, you would like to define minimal and maximal damage.
Of course, you could create two variables:
```c++
int32 MinDamage = 10;

int32 MaxDamage = 20;
```
But there are better ways. One of them - use **TInterval**.
```c++
TInterval<int32> Damage {10, 20};
```
But there is one issue. Very critical issue. You can't expose this variable to Blueprints.

It is nonsense. Designers should have easy access to this data to modify them. But blueprints don't supports templates.

So what the solution you can think about?

Define your own structure!
```c++
USTRUCT(BlueprintType)
struct FDamageStruct
{
	GENERATED_BODY()

	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
	int32 MinDamage = 10;
	
	UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
	int32 MaxDamage = 20;
};

///////////////////////////

UPROPERTY(EditDefaultsOnly)
FDamageStruct Damage;
```
Technically, it works.

![Damage struct](https://apokrif6.github.io/assets/images/generic_numeric_structures/damage_struct.png)

But imagine case. Now you need to make same with character's speed.
Will you define `FSpeedStruct`? Definitely not. We need more generics!

<h3>Interval for Blueprints</h3>
There are:
```c++
DEFINE_INTERVAL_WRAPPER_STRUCT(FFloatInterval, float)
DEFINE_INTERVAL_WRAPPER_STRUCT(FDoubleInterval, double)
DEFINE_INTERVAL_WRAPPER_STRUCT(FInt32Interval, int32)
```
It is special wrappers to support exposing to blueprints:
```c++
UPROPERTY(EditDefaultsOnly)
FInt32Interval Damage {10, 20};
```

![F32Interval](https://apokrif6.github.io/assets/images/generic_numeric_structures/fint32_interval.png)

So creating your own interval structures has no sense because of `FXInterval`

<h3>MathFwd</h3>
There is also one thing, you could use.
Look into:
`Engine\Source\Runtime\Core\Public\Math\MathFwd.h`

And find a bunch of forward declarations.
Whole engine uses it.
When you see vector of something, it is:
```c++
FVector == UE::Math::TVector<double> == template<typename T> struct TVector
```
So, technically templates could be exposed to blueprints this way.

And we can use it.

<h3>Templated Numeric Point</h3>
Let's check, is there something interesting for us in **MathFwd**. Remember, we need generic min-max template.

There is something called `TIntPoint`. Name suggests it is used as templated int.
It is totally okay for our min-max to be numeric X-Y (because it is point).
There are few declarations for this:
```c++
// Int points
using FInt32Point = UE::Math::TIntPoint<int32>;
using FInt64Point = UE::Math::TIntPoint<int64>;
using FUint32Point = UE::Math::TIntPoint<uint32>;
using FUint64Point = UE::Math::TIntPoint<uint64>;
```
Signed and unassigned ints for 32 and 64 bits.
We should try it:
```c++
UPROPERTY(EditDefaultsOnly)
FInt32Point Damage {10, 20};
```

And yeah, this works:

![Generic int struct](https://apokrif6.github.io/assets/images/generic_numeric_structures/generic_int_struct.png)

The only issue, there are `X` and `Y` names of variables, because of generic.
But from my perspective, it is fair price for ability to declare generic X-Y struct.

<h3>Templated Numeric Rect</h3>
From other hand, `FInt32Rect` could not be exposed to blueprints. I have no idea why.
Nobody doesn't add support for it :(

But if you want to declare some generic numeric stuff, just look into `MathFwd`.
Maybe somebody add it, and add support for blueprints!
