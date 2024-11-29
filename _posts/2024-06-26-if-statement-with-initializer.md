---
title: If statement with initializer
---
<h3>What it is</h3>
This feature was added in [C++17](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0305r0.html). What does it mean for us?
We can initialize a variable inside an if statement, and perform the actual conditional check after initialization.

| Statement | Equivalent | Iterations |
|----------|----------|----------|
| if (init; cond) E;    | { init; while(cond) { E; break; } }   | Once while cond holds   |
| if (init; cond) E; else F;   | (more complex)   | Once   |

"New" if statement looks like that:

```c++
if (init; condition)
```

<h3>Guard clause</h3>
Here is an example of guard clause:

```c++
AMyPlayerCharacter* PC = UMyFunctionLibrary::GetMyPlayerCharacter(this);
if (PC)
{
  PC->DoStuff();
}

if (AMyPlayerCharacter* PC = UMyFunctionLibrary::GetMyPlayerCharacter(this))
{
  PC->DoStuff();
}
```
Second one is one line shorter. It is just an if statement with simple condition which is permitted to be:
> declaration of a single non-array variable with a brace-or-equals initializer

Techically, there is an implicit condition. But **it is not an initializer with a coniditonal**!
You can use guard clause in previous versions of C++.

<h3>Initializer with condition</h3>
For C++17 if statement with initializer, you must provide explicit condition:

```c++
if (int32 RandomNumber = FMath::RandRange(0, 100); RandomNumber < MinimalChance)
{
  DoRandomStuff();
}
```


Also, you can combine guard clause with condition:

```c++
if (AMyPlayerCharacter* PC = UMyFunctionLibrary::GetMyPlayerCharacter(this);
    PC && PC->CanDoStuff())
{
  PC->DoStuff();
}
```
Note that is your responsibility to perform `nullptr` checks, because **first statement is not a condition anymore, it is just an initializer**.

Remember, that conditional scope accept any condition, so you can combine it any way you want.

<h3>Conclusion</h3>
Personally, using features provided by technology is an absolutely good practice. Being up-to-date is a good practice as well.
To conclude, `if statement with initializers` will make your code more *modern* and cleaner.
