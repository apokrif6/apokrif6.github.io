---
title: How to compare strings with conditional breakpoint in Unreal Engine
---

<h3>Problem</h3>
While debugging in Unreal Engine, you might want to break execution only when a specific string condition is met.
For example, you have 100 NPCs in your game, and you want to break execution only when the name of the NPC is `BP_NPC_Test`.

You might think that the natural way to do this is to use a conditional breakpoint, so you create condition like:
```c++
GetString() == "String"
```
but this won’t work as expected. Depends on your debugger, it will freeze, throw an error, or behave like a regular breakpoint and trigger every time.

The reason is because of debugger evaluator can't parse the string comparison correctly. Unreal Engine uses `FString` for strings, which is a custom type that the debugger doesn't handle.
```
Couldn't parse conditional expression:
warning: result of comparison against a string literal is unspecified
error: invalid operands to binary expression ('FString' and 'const char [7]')
candidate function not viable: no known conversion from 'FString' to 'UE::CoreUObject::Private::FObjectHandlePrivate' for 1st argument
candidate function not viable: no known conversion from 'FString' to 'const FIoHash' for 1st argument
candidate function not viable: no known conversion from 'FString' to 'const wchar_t *' for 1st argument
candidate function not viable: no known conversion from 'FString' to 'const FLazyName' for 1st argument
...
```

<h3>Solution</h3>
The best way to compare strings here is to use `wcscmp` method:
```c++
wcscmp(str1, str2) == 0
```

This is comparing two wide-character strings (`wchar_t*`). It's like `strcmp`, but for wide strings.

`wcscmp(str1, str2)` returns:
- `0` if the strings are identical
- `< 0` if str1 is lexicographically less than str2 
- `> 0` if str1 is greater

Technically, you can use `strcmp`, but `wchar_t*` format is used for Unicode (wide) strings. Mixing them would result in a type mismatch or incorrect behavior.
Unreal Engine uses wide strings everywhere with `TEXT` macro, so we should too.

The `TEXT` macro expands at compile time and isn't available in runtime debugger expressions, so you can use `L` prefix for wide strings in C++:
```c++
L"Test"    // wide string (wchar_t*)
"Test"     // regular narrow string (char*)
```

We can't use `FString` directly because it is a custom type in Unreal Engine, and the debugger doesn't know how to handle it in a conditional breakpoint expression. So we will get raw char data from `FString` via:
```c++
StringVariable.Data.AllocatorInstance.Data
```

So the final conditional breakpoint expression will look like this:
```c++
wcscmp((wchar_t*)StringVariable.Data.AllocatorInstance.Data, L"StringToCompare") == 0
```