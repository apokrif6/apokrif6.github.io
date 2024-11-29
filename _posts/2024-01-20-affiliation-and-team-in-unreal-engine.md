---
title: Affiliation and Team in Unreal Engine
---
<h3>Problem</h3>
You start creating in-game AI and learning about perception.
Then, while using ***Sensing***, you are faced with ***Detection by Affiliation***.

Yet in blueprints, there is not much you can do to get a solid grasp on ***Affiliation***.
At best, you check all checkboxes, experiment and move on without understanding what it is.

So what is it?

***Affiliation*** is a very powerful concept which, unfortunately, could be set only in C++.

Every controller in game has affiliation towards another controller.
It means that two controllers can be made ***hostile*** to each other in context of Unreal.

Perception sense with hostile affiliation will recognize controllers as "enemies" or "friends", based on corresponding checkbox.
You can write your own systems utilizing this.

<h3>Teams</h3>
So, Unreal offers us three affiliations:

```c++
UENUM(BlueprintType)
namespace ETeamAttitude
{
	enum Type : int
	{
		Friendly,
		Neutral,
		Hostile,
	};
};
```

Let's look into the default implementation of affiliation checking:

```c++
virtual ETeamAttitude::Type GetTeamAttitudeTowards(const AActor& Other) const
{ 
    const IGenericTeamAgentInterface* OtherTeamAgent = Cast<const IGenericTeamAgentInterface>(&Other);
    return OtherTeamAgent ? FGenericTeamId::GetAttitude(GetGenericTeamId(), OtherTeamAgent->GetGenericTeamId())
        : ETeamAttitude::Neutral;
};
```

There we have another concept, teams:

```c++
USTRUCT(BlueprintType)
struct FGenericTeamId
{
	GENERATED_USTRUCT_BODY()

private:
	enum EPredefinedId
	{
		// if you want to change NoTeam's ID update FGenericTeamId::NoTeam

		NoTeamId = 255
	};

protected:
	UPROPERTY(Category = "TeamID", EditAnywhere, BlueprintReadWrite)
	uint8 TeamID;
	
	...
};
```

Team of your controller is defined by ID.
Afterwards, your controller checks attitude with another actor. It doesn't have to be a controller, because you can implement
**IGenericTeamAgentInterface** everywhere. That said, I strongly recommend using controller for characters :)

<h3>Let's dive into real life example</h3>
Imagine that in your RPG game you have a player, squad of allies and some enemies.
Player and their allies are friendly to each other, enemies are hostile to both of them.

Let's start with enum:

```c++
UENUM(BlueprintType)
enum class ETeam : uint8
{
	None UMETA(Hidden),
	Player,
	Ally,
	Enemy
};
```

Next one will be the PlayerController:

```c++
UCLASS()
class YOURGAME_API ARPGPlayerController : public APlayerController, public IGenericTeamAgentInterface
{
	GENERATED_BODY()
	
public:
	virtual FGenericTeamId GetGenericTeamId() const override
	{
		return FGenericTeamId(StaticCast<uint8>(ETeam::Player));
	}

	virtual ETeamAttitude::Type GetTeamAttitudeTowards(const AActor& Other) const override
	{
		const APawn* OtherPawn = Cast<APawn>(&Other);
		if (const IGenericTeamAgentInterface* TeamAgent = Cast<IGenericTeamAgentInterface>(OtherPawn->GetController()))
		{
			const FGenericTeamId OtherGenericTeamId = TeamAgent->GetGenericTeamId();

			if (OtherGenericTeamId == FGenericTeamId(StaticCast<uint8>(ETeam::Ally)))
				return ETeamAttitude::Friendly;

			if (OtherGenericTeamId == FGenericTeamId(StaticCast<uint8>(ETeam::Enemy)))
				return ETeamAttitude::Hostile;
		}

		return ETeamAttitude::Neutral;
	}
};
```

And finally AIController:

```c++
UCLASS()
class YOURGAME_API AAIRSGController : public AAIController
{
	GENERATED_BODY()

public:
	virtual FGenericTeamId GetGenericTeamId() const override
	{
		return FGenericTeamId(StaticCast<uint8>(Team));
	}

	virtual ETeamAttitude::Type GetTeamAttitudeTowards(const AActor& Other) const override
	{
		const APawn* OtherPawn = Cast<APawn>(&Other);

		if (const IGenericTeamAgentInterface* TeamAgent = Cast<IGenericTeamAgentInterface>(OtherPawn->GetController()))
		{
			const FGenericTeamId OtherGenericTeamId = TeamAgent->GetGenericTeamId();

			if (GetGenericTeamId() == OtherGenericTeamId)
				return ETeamAttitude::Friendly;

			if (GetGenericTeamId() == FGenericTeamId(StaticCast<uint8>(ETeam::Ally) &&
				OtherGenericTeamId == FGenericTeamId(StaticCast<uint8>(ETeam::Player))))
				return ETeamAttitude::Friendly;

			return ETeamAttitude::Hostile;
		}

		return ETeamAttitude::Neutral;
	}

protected:
	UPROPERTY(EditDefaultsOnly, Category = Team)
	ETeam Team;
};
```

ID can also be set with this method, when needed:

```c++
SetGenericTeamId(StaticCast<uint8>(Team));
```


<h3>Conclusion</h3>
Now, our **Player** is friendly to **Ally** and hostile to **Enemy**.

Mutually, Ally is friendly to Player and another Allies, while being hostile to Enemies.

Hopefully that clarifies how to make the perception work as expected.

But there is more.

You can define as many ***"teams"*** as you need, and use them by overriding ***GetTeamAttitudeTowards*** as you wish.
For example, you can add a "berserk" mode when an ally can attack everyone, or some enemies could be Neutral by
default.

Your fantasy is the only limit.
