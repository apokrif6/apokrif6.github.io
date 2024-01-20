---
title: Affiliation and Team in Unreal Engine
---
<h3>Problem</h3>
You start creating in-game AI, and learning about perception.
And as soon you start using ***Sensing***, you are faced with ***Detection by Affiliation***.

But if you are using blueprints, you will never know what it really is.
And just mark all checkboxes, without understanding what does it really mean.

So what is it?

Unfortunately, ***Affiliation*** could be set only with C++, but it's very powerful concept

Every controller in game has affiliation towards another controller.
It means that having two controllers, you can make them ***hostile*** to each other in context of Unreal.

That perception sense with hostile affiliation will detect those "enemies" or "friends", if you corresponding checkbox.
And you could write your own systems which are using this.

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

Let's look into default implementation of affiliation checking:

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

You define team of your controller with an ID.
And after it, your controller checks attitude with another actor. Not necessarily controller, because you can implement
**IGenericTeamAgentInterface** everywhere. But I strongly recommend use controller for characters :)

<h3>Let's dive into real example</h3>
In your RPG game you have player, squad allies and enemies.
Player and allies are friendly to each other, enemies are hostile to both of them.

Start with enum:

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

Next one will be PlayerController:

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

And finish with AIController:

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

Don't forget that you can set ID with this method, if you need:
```c++
SetGenericTeamId(StaticCast<uint8>(Team));
```


<h3>Conclusion</h3>
So now, our **Player** is friend for **Ally** and hostile for **Enemy**.

And Ally is friendly for Player and another Allies, and hostile for Enemies.

Now your perception finally working as you expected.

But it's not end.
You could have so many ***"teams"***, and use them by overriding ***GetTeamAttitudeTowards*** as you with.
For example, you can have a "berserk" mode when ally could attack everybody. Or some enemies could be Neutral by
default.
Your fantasy is only the limit.

