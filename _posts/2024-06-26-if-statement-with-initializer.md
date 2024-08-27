---
title: If statement with initializer
---
<h3>What it is</h3>
This feature was added in [C++17](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0305r0.html). What does it mean for us?
We can initialize a variable inside if statement, and perform the actual conditional check after initialization.

| Statement | Equivalent | Iterations |
|----------|----------|----------|
| if (init; cond) E;    | { init; while(cond) { E; break; } }   | Once while cond holds   |
| if (init; cond) E; else F;   | (more complex)   | Once   |

Tehnically, new if looks like:

```c++
if (init; condition)
```

<h3>Guard clause</h3>
Here is example of guard clause:

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
Second one is one-line less. It is just if with simple condition, because condition is permitted to be:
> declaration of a single non-array variable with a brace-or-equals initializer

So techically, there is implicit condition. But **it is not initializer with coniditonal**!
And you can use it in previous versions of C++.

<h3>Initializer with condition</h3>
If you want use C++17's if statement with initializer, you should provide explicit condition:

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
Note, you should care about checking on `nullptr`. Because **first statement is not condition anymore, it is just initializer**.

Remember, that conditional scope accept any condition, so you can combine it any way you would like to.

<h3>Conclusion</h3>
For me, using features provided by technology is absolutely good practice. And be up-to-date is a good practice too.
So using `if statement with initializers` will make your code cleaner and more *modern*.
