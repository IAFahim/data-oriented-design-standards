# Chapter 3: Return Values – The TryX Pattern

> **"Make success and failure explicit. Avoid sentinel values. Use bool + out parameters."**

---

## Table of Contents

1. [The Problem with Traditional Return Values](#the-problem-with-traditional-return-values)
2. [What is the TryX Pattern?](#what-is-the-tryx-pattern)
3. [The Rules](#the-rules)
4. [Anatomy of a TryX Method](#anatomy-of-a-tryx-method)
5. [Real-World Examples](#real-world-examples)
6. [TryX vs Exceptions](#tryx-vs-exceptions)
7. [Common Mistakes](#common-mistakes)

---

## The Problem with Traditional Return Values

In C#, there are too many ways to say "it failed".

### 1. Sentinel Values (The "Magic Number" Problem)
```csharp
// ❌ BAD
public int FindIndex(int value) {
    // ...
    return -1; // Does -1 mean error? Or just not found?
}
```
You have to remember what `-1` means. Is it `0`? `null`? `int.MinValue`? It's messy.

### 2. Null Returns
```csharp
// ❌ BAD
public Player FindPlayer(int id) {
    return null; // Hope you remember to check for null!
}
```
This is how you get `NullReferenceException`.

### 3. Exceptions for Logic
```csharp
// ❌ BAD
public int Parse(string input) {
    if (invalid) throw new Exception(); // 💥 Expensive!
    return result;
}
```
Exceptions are incredibly slow (thousands of CPU cycles). They allocate memory (garbage). They are not Burst-compatible. **Never use exceptions for game logic.**

---

## What is the TryX Pattern?

The **TryX Pattern** is standard in .NET (think `int.TryParse`). It's the cleanest way to handle operations that might fail.

**The Contract:**
1.  Return `bool` (true = success, false = failure).
2.  Output the result via an `out` parameter.

---

## The Rules

### 1. Always Return `bool`
The return value tells you **only** if the operation succeeded.

```csharp
// ✅ CORRECT
public static bool TryParse(string input, out int result)
```

### 2. Always Initialize `out` Parameters
You must assign a value to `out` parameters before returning, even on failure.

```csharp
public static bool TryGet(out int value)
{
    value = default; // Do this first!
    // ...
}
```

### 3. Name it "Try..."
Prefix your method with `Try`. It tells the developer: "This might fail, and you need to handle it."

---

## Anatomy of a TryX Method

```csharp
[MethodImpl(MethodImplOptions.AggressiveInlining)]
public static bool TryCalculate(
    // Inputs
    float a, float b,
    // Outputs
    out float result)
{
    // 1. Initialize defaults
    result = 0f;
    
    // 2. Validate (Early Exit)
    if (a < 0) return false;
    if (b < 0) return false;
    
    // 3. Do the work
    result = a * b;
    
    // 4. Return success
    return true;
}
```

---

## Real-World Examples

### Example 1: Inventory

```csharp
public static bool TryAddItem(ref int count, int max, int amount, out int added)
{
    added = 0;
    
    if (amount <= 0) return false;
    if (count >= max) return false;
    
    int space = max - count;
    added = Math.Min(amount, space);
    count += added;
    
    return added == amount; // True only if we added everything
}
```

### Example 2: Safe Division

```csharp
public static bool TryDivide(float a, float b, out float result)
{
    result = 0f;
    
    if (b == 0f) return false; // Prevent crash
    
    result = a / b;
    return true;
}
```

---

## TryX vs Exceptions

| Feature | TryX Pattern | Exceptions |
|---------|-------------|------------|
| **Speed** | ~5 cycles (Fast) | ~1000+ cycles (Slow) |
| **Allocations** | Zero | Allocates Exception object + Stack Trace |
| **Burst** | ✅ Compatible | ❌ Not Compatible |
| **Control Flow** | Explicit (`if`) | Implicit (`catch`) |

**Rule of Thumb:** Use Exceptions only for things that should **never** happen (like a corrupted file or missing hardware). For everything else (gameplay logic, parsing, validation), use TryX.

---

## Common Mistakes

### 1. Returning True on Partial Success
If the method didn't do exactly what was asked, return `false`.

```csharp
// ❌ WRONG
public bool TryAdd(int amount) {
    // Added half the amount...
    return true; // Confusing!
}
```

### 2. Using `out` for Input
`out` parameters are for output. If you need to read the value and then modify it, use `ref`.

```csharp
// ❌ WRONG
public bool TryModify(out int val) {
    val += 1; // Error: Use of unassigned out parameter
}

// ✅ CORRECT
public bool TryModify(ref int val) {
    val += 1;
}
```

---

**Next:** [Chapter 4: Performance - Burst Compatibility →](./04-burst-compatibility.md)
