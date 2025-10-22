---
title: TNonNullPtr — Non-Nullable Raw Pointers in Unreal Engine
---

<h3>What is TNonNullPtr</h3>

`TNonNullPtr<T>` is a **non-nullable, non-owning raw pointer** that **guarantees** the pointer is never `nullptr`. It wraps a `T*` but **bans null** at compile-time and runtime via `ensureMsgf`.

```c++
TNonNullPtr<AActor> ActorPtr(SomeValidActor);  // OK
TNonNullPtr<AActor> Bad(nullptr);              // Banned + ensure failure
```

<h3>Problem</h3>
Raw pointers require constant null checks:

```c++
void DoSomething(ATestActor* TestActor)
{
    if (TestActor)
    {
        TestActor->DoMagic();
    }
}
```

This leads to:
- Boilerplate null checks
- Runtime crashes if someone accidentally passes nullptr
- Cognitive overhead: "Wait, can this be null or not?"

<h3>Solution</h3>

`TNonNullPtr<T>` wraps a raw pointer but forbids null:

```c++
void DoSomething(ATestActor* TestActor)
{
    TestActor->DoMagic(); // No null check needed!
}
```

If anyone tries to pass nullptr literal, it fails at compile-time. Passing nullptr via a variable triggers `ensureMsgf` at runtime:
```c++
ATestActor* TestActorRaw = nullptr;
TNonNullPtr<ATestActor> Safe(TestActorRaw); // ensureMsgf: "Tried to initialize TNonNullPtr with a null pointer!"
```

<h3>Key features</h3>

<h4>Construction</h4>

```c++
ATestActor* ValidTestActor = GetTestActor();
TNonNullPtr<ATestActor> TestActor(ValidTestActor);  // OK

TNonNullPtr<ATestActor> TestActor(nullptr);         // Compile-time banned + ensure in debug
```

<h4>Construction</h4>

```c++
ATestActor* ValidTestActor = GetTestActor();
TNonNullPtr<ATestActor> TestActor(ValidTestActor);  // OK

TNonNullPtr<ATestActor> TestActor(nullptr);         // Compile-time banned + ensure in debug
```

<h4>Assignment</h4>

```c++
TestActor = AnotherValidTestActor;      // OK
TestActor = nullptr;                    // Banned
```

<h4>Functions</h4>

```c++
TArray<TNonNullPtr<ATestActor>> GetTestActors()
{
    TArray<TNonNullPtr<ATestActor>> TestActors;

    //...

    return TestActors;
}
```

<h4>Works with TOptional</h4>

```c++
TOptional<TNonNullPtr<ATestActor>> OptionalActor;
OptionalActor.Emplace(SomeValidActor);

if (OptionalActor)
{
    OptionalActor->DoMagic();
}
```

<h4>Implicit Conversion</h4>

```c++
ATestActor* RawTestActor = TestActor; // OK — but only after ensure
```

<h3>Best Practice</h3>

Use `TNonNullPtr` only when you control both sides and can guarantee validity.

```c++
// API contract: Caller MUST pass valid pointer
void TestFunction(TNonNullPtr<ATestActor> TestActor)
{
    // SAFE to use directly — IF caller respects contract
    TestActor->DoMagic();
}
```

<h3>When to use it?</h3>

| Scenario                                             | Use `TNonNullPtr<T>`?                                    |
|------------------------------------------------------|----------------------------------------------------------|
| Function parameter guaranteed non-null               | Yes                                                      |
| Return value from a function that never returns null | Yes                                                      |
| Class member that is always valid after construction | Yes                                                      |
| Temporary pointer that might be null                 | No → use `TObjectPtr<T>`, raw pointer, or `TOptional<T>` |
