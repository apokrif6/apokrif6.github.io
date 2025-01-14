---
title: Generic numeric structures
---
<h3>Problem</h3>
There are a lot of cases when you need to use some common structures that follow the same pattern.

For example, when you are working on Ability System with magic spells, you would like to define minimal and maximal damage.
Of course, you can create two variables:
```c++
int32 MinDamage = 10;

int32 MaxDamage = 20;
```
Still, there are better ways. One of them - **TInterval**.
```c++
TInterval<int32> Damage {10, 20};
```
There is one issue. Very critical one in fact. You can't expose this variable to Blueprints.

It is nonsense. Blueprints doesn't support templates denying designers an easy access to data modification.

So what is the solution you can think about?

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

This solution is also flawed. Imagine that now you need to create a similar structure for character's speed.
Will you define `FSpeedStruct`? Definitely not. We need a more generic solution!

<h3>Interval for Blueprints</h3>
There are:
```c++
DEFINE_INTERVAL_WRAPPER_STRUCT(FFloatInterval, float)
DEFINE_INTERVAL_WRAPPER_STRUCT(FDoubleInterval, double)
DEFINE_INTERVAL_WRAPPER_STRUCT(FInt32Interval, int32)
```
These are special wrappers that expose intervals to blueprints:
```c++
UPROPERTY(EditDefaultsOnly)
FInt32Interval Damage {10, 20};
```

![F32Interval](https://apokrif6.github.io/assets/images/generic_numeric_structures/fint32_interval.png)

That means that your own interval structures are already implemented by `FXInterval`.

<h3>MathFwd</h3>
There is also another interesting solution.
Look into:
`Engine\Source\Runtime\Core\Public\Math\MathFwd.h`

There you can find a bunch of forward declarations.
Whole engine uses them.
When you see vector of something, it is:
```c++
FVector == UE::Math::TVector<double> == template<typename T> struct TVector
```
So, technically templates could be exposed to blueprints this way.

We can use this.

<h3>Templated Numeric Point</h3>
Let's see if **MathFwd** contains a generic min-max template that we are interested in.

There is something called `TIntPoint`. Name suggests it is used as a templated int.
It is totally fine for our min-max template to be numeric X-Y (it can be represented by a point).
There are few point declarations:
```c++
// Int points
using FInt32Point = UE::Math::TIntPoint<int32>;
using FInt64Point = UE::Math::TIntPoint<int64>;
using FUint32Point = UE::Math::TIntPoint<uint32>;
using FUint64Point = UE::Math::TIntPoint<uint64>;
```
Signed and unsigned 32 and 64 bit integers.
We should try it:
```c++
UPROPERTY(EditDefaultsOnly)
FInt32Point Damage {10, 20};
```

And yeah, this works:

![Generic int struct](https://apokrif6.github.io/assets/images/generic_numeric_structures/generic_int_struct.png)

The only issue is generic `X` and `Y` variable names.
However, from my perspective, it is a fair price to pay for ability to declare a generic X-Y struct.

<h3>Templated Numeric Rect</h3>
On the other hand, `FInt32Rect` cannot be exposed to blueprints. I have no idea why.
Nobody doesn't want to add support for it :(

Look into `MathFwd` in case you want to declare some generic numeric stuff.
Maybe somebody will add it together with support for blueprints!
