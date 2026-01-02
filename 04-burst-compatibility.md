# Chapter 4: Performance – Burst Compatibility & Primitive Logic

> **"Write C# that compiles to C++ performance. Pass primitives, inline everything, avoid managed types."**

---

## Table of Contents

1. [What is Burst?](#what-is-burst)
2. [The Burst Compatibility Rules](#the-burst-compatibility-rules)
3. [Why Primitives Only?](#why-primitives-only)
4. [Forbidden Constructs](#forbidden-constructs)
5. [Method Inlining](#method-inlining)
6. [Struct vs Primitive Parameters](#struct-vs-primitive-parameters)
7. [Burst Inspector Deep Dive](#burst-inspector-deep-dive)

---

## What is Burst?

Burst is a compiler that takes your C# code and turns it into highly optimized native machine code. It's not magic—it just has strict rules. If you follow them, you get C++ speeds (or faster). If you break them, it won't compile.

**The Transformation:**
1.  **You write:** High-level C# (IL).
2.  **Burst compiles:** Optimized LLVM IR.
3.  **You get:** Native Assembly (x64, ARM) with SIMD instructions.

**The Payoff:**
- Vector math: **10x faster**
- Complex logic: **25-50x faster**

---

## The Burst Compatibility Rules

### 1. Static Methods Only
Burst doesn't like `this`. It doesn't want to know about your object instance, vtables, or inheritance. It just wants a function.

```csharp
// ❌ BAD
public class MyLogic {
    public void Calculate() { } // Instance method = hidden 'this' pointer
}

// ✅ GOOD
public static class MyLogic {
    public static void Calculate() { } // Pure function
}
```

### 2. No Managed Types
If the Garbage Collector has to touch it, Burst hates it.

**Forbidden:** `string`, `class`, `object`, `List<T>`, `int[]`.
**Allowed:** `int`, `float`, `bool`, `struct`, `NativeArray<T>`, `FixedString`.

```csharp
// ❌ BAD
public static void Log(string msg) { }

// ✅ GOOD
public static void Log(FixedString64Bytes msg) { }
```

### 3. Primitives Only in Logic
This is the most important architectural rule. **Do not pass your structs into your logic methods.** Pass the fields (primitives) instead.

```csharp
// ❌ BAD - Passing the whole struct
public static void Tick(ref Cooldown c, float dt) {
    c.Current -= dt;
}

// ✅ GOOD - Passing primitives
public static void Tick(ref float current, float dt) {
    current -= dt;
}
```

---

## Why Primitives Only?

You might think, "Why not just pass the struct? It's cleaner."

**The reason is SIMD (Single Instruction, Multiple Data).**

When you pass `ref float current`, the Burst compiler sees a stream of floats. It can pack 4 or 8 of them into a single CPU register and process them all at once.

When you pass `ref Cooldown c`, the compiler sees a struct with mixed fields (maybe a float, an int, a bool). It can't easily vectorize that because the data isn't uniform.

**The Difference:**
- **Struct parameter:** Scalar processing (1 item at a time).
- **Primitive parameter:** Vector processing (4-8 items at a time).

---

## Forbidden Constructs

### 1. Try-Catch Blocks
Exceptions are managed. Burst doesn't support them. Use the **TryX Pattern** instead.

### 2. Virtual Methods / Interfaces
Burst needs to know exactly which function to call at compile time. Virtual calls require a runtime lookup.

### 3. Boxing
`object x = 5;` creates an object on the heap. That's an allocation. Burst forbids allocations.

---

## Method Inlining

Function calls have overhead. They involve jumping memory addresses and pushing/popping the stack.

For small, frequently called methods, we want to eliminate that overhead entirely.

```csharp
[MethodImpl(MethodImplOptions.AggressiveInlining)]
public static float Square(float x)
{
    return x * x;
}
```

**What happens:** The compiler takes the code inside `Square` and pastes it directly into the calling function. No jump, no stack push. Just raw math.

**Rule of Thumb:** If a method is small (less than 10 lines) and called inside a loop, inline it.

---

## Struct vs Primitive Parameters

Let's look at a real example.

### The "Clean" (But Slow) Way
```csharp
public static void Translate(ref Transform2D t, float dx, float dy)
{
    t.X += dx;
    t.Y += dy;
}
```
Burst has to load the struct, modify fields, and store it back. It's hard to vectorize because `X` and `Y` might be interleaved with other data.

### The "Fast" Way
```csharp
public static void Translate(ref float x, ref float y, float dx, float dy)
{
    x += dx;
    y += dy;
}
```
Burst loads `x` and `y` into registers. If you run this in a loop over arrays of `x` and `y`, Burst will use AVX instructions to update 8 positions simultaneously.

---

## Burst Inspector Deep Dive

Want to see if your code is actually fast?

1.  **Window** → **Analysis** → **Burst Inspector**.
2.  Select your job.
3.  Look at the Assembly.

**What to look for:**
- **Green/Blue lines:** Good.
- **Yellow/Red lines:** Bad (scalar fallback).
- **`vaddps`, `vmulps`:** Vector instructions (Good!).
- **`call`:** Function calls (Bad if inside a tight loop).

---

**Next:** [Chapter 5: Unity Integration - Thin Adapter Pattern →](./05-thin-adapter.md)
