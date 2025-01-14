---
title: Cannot resolve symbol 'super'
---
<h3>Explanation</h3>
First of all, native C++ doesn't have ***Super***.

It is present in Java, Python, Ruby yet not in C++ where you have to use **ParentClassName** instead.

```c++
class Foo
{
public:
    void method() {
        //do stuff
    }    
}

class Bar : public Foo
{
public:
    void method() {
        //do another stuff
        Foo::method(); <- Here you call parent class
    }
}
```

But you probably saw something like:
```c++
void AGameNameCharacter::BeginPlay()
{
	Super::BeginPlay();
	//your code
}
```

<h3>Reflection</h3>
You can use ***Super::*** due to Unreal Engine reflection system.

Every class marked as **UCLASS** supports reflection and all its advantages.
Let's find the declaration of **UCLASS**:
```c++
#define DECLARE_CLASS( TClass, TSuperClass, TStaticFlags, TStaticCastFlags, TPackage, TRequiredAPI  ) \
private: \
    TClass& operator=(TClass&&);   \
    TClass& operator=(const TClass&);   \
	TRequiredAPI static UClass* GetPrivateStaticClass(); \
public: \
	/** Bitwise union of #EClassFlags pertaining to this class.*/ \
	static constexpr EClassFlags StaticClassFlags=EClassFlags(TStaticFlags); \
	/** Typedef for the base class ({{ typedef-type }}) */ \
	typedef TSuperClass Super;\
	/** Typedef for {{ typedef-type }}. */ \
	typedef TClass ThisClass;\
	/** Returns a UClass object representing this class at runtime */ \
	inline static UClass* StaticClass() \
	{ \
		return GetPrivateStaticClass(); \
	} \
	/** Returns the package this class belongs in */ \
	inline static const TCHAR* StaticPackage() \
	{ \
		return TPackage; \
	} \
	/** Returns the static cast flags for this class */ \
	inline static EClassCastFlags StaticClassCastFlags() \
	{ \
		return TStaticCastFlags; \
	} \
	/** For internal use only; use StaticConstructObject() to create new objects. */ \
	inline void* operator new(const size_t InSize, EInternal InInternalOnly, UObject* InOuter = (UObject*)GetTransientPackage(), FName InName = NAME_None, EObjectFlags InSetFlags = RF_NoFlags) \
	{ \
		return StaticAllocateObject(StaticClass(), InOuter, InName, InSetFlags); \
	} \
	/** For internal use only; use StaticConstructObject() to create new objects. */ \
	inline void* operator new( const size_t InSize, EInternal* InMem ) \
	{ \
		return (void*)InMem; \
	} \
	/* Eliminate V1062 warning from PVS-Studio while keeping MSVC and Clang happy. */ \
	inline void operator delete(void* InMem) \
	{ \
		::operator delete(InMem); \
	}
```
Here we have:
```c++
typedef TSuperClass Super
```
**[typedef](https://en.cppreference.com/w/cpp/language/typedef)** is a specifier which creates alias for type. In this case **Super** is an alias to **TSuperClass**.

Build tool creates our **UObject** based on the data from the macro:
```c++
DECLARE_CLASS(UObject,UObject,CLASS_Abstract|CLASS_Intrinsic|CLASS_MatchedSerializers,CASTCLASS_None,TEXT("/Script/CoreUObject"),COREUOBJECT_API)
```
That's why our **UObject** or Character, GameMode, Actor or any other class marked as **UCLASS** has **Super::** and **ThisClass::** by default.

<h3>Super for non UCLASS</h3>
Sometimes, when we work with low-level API we also work with derived classes. And intuitively, we use ***Super***.
This is invalid because no reflection means no ***Super***.

One way to solve this problem is to specify our own **typedef**.
For example:
```c++
//NON UCLASS
class FOurEditorDragDropAction final : public FGraphEditorDragDropAction
{
protected:
    //you can name your variable whatever you want, but keep in mind that Super is considered a convention
    typedef FGraphEditorDragDropAction Super;

    virtual void OnDrop(bool bDropWasHandled, const FPointerEvent& MouseEvent) override;
}
```

```c++
void FOurEditorDragDropAction::OnDrop(bool bDropWasHandled, const FPointerEvent& MouseEvent)
{
	GraphPanel->OnStopMakingConnection();
	GraphPanel->OnEndRelinkConnection();
	
	//without typedef, we must use FGraphEditorDragDropAction::OnDrop(bDropWasHandled, MouseEvent);
	Super::OnDrop(bDropWasHandled, MouseEvent);
}
```

Alternatively, use **[__super](https://learn.microsoft.com/en-us/cpp/cpp/super)** keyword. It is used by Visual Studio compiler, which is supported by Unreal Engine by default.
```c++
void FOurEditorDragDropAction::OnDrop(bool bDropWasHandled, const FPointerEvent& MouseEvent)
{
	GraphPanel->OnStopMakingConnection();
	GraphPanel->OnEndRelinkConnection();
	
	//in this case you don't need typedef
	__super::OnDrop(bDropWasHandled, MouseEvent);
}
```