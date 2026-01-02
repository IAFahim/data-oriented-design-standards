# Chapter 2: Control Flow – The Early Exit Pattern

> **"Flatten your code. Reduce complexity. Validate early, exit fast."**

---

## Table of Contents

1. [The Problem with Nested Logic](#the-problem-with-nested-logic)
2. [What is the Early Exit Pattern?](#what-is-the-early-exit-pattern)
3. [The Rules](#the-rules)
4. [Cyclomatic Complexity Explained](#cyclomatic-complexity-explained)
5. [Real-World Examples](#real-world-examples)
6. [Guard Clauses in Action](#guard-clauses-in-action)
7. [Performance Benefits](#performance-benefits)
8. [Common Mistakes](#common-mistakes)

---

## The Problem with Nested Logic

We've all seen "Arrow Code"—code that looks like a sideways pyramid. It's hard to read, hard to test, and hard to maintain.

```csharp
// ❌ NESTED NIGHTMARE
public static bool ProcessDamage(int health, int maxHealth, int damage, bool isInvincible)
{
    if (!isInvincible)
    {
        if (damage > 0)
        {
            if (health > 0)
            {
                health -= damage;
                if (health < 0)
                {
                    health = 0;
                }
                return health > 0;
            }
            else
            {
                return false;
            }
        }
        else
        {
            return true;
        }
    }
    else
    {
        return true;
    }
}
```

**Why this sucks:**
- You have to keep track of 5 levels of context in your head.
- It's almost impossible to see which `else` belongs to which `if`.
- Adding a new condition requires re-indenting the whole block.

---

## What is the Early Exit Pattern?

The **Early Exit Pattern** (or Guard Clause pattern) flips the logic. Instead of checking for the "happy path" and nesting, you check for failure and return immediately.

### The Mental Shift

**Old Way:** "If everything is okay, do the work."
**New Way:** "If anything is wrong, stop. Otherwise, do the work."

---

## The Rules

### 1. No `else` Blocks
Seriously. You almost never need them. If you return inside an `if`, you don't need an `else`.

```csharp
// ❌ WRONG
if (condition) {
    return true;
} else {
    return false;
}

// ✅ CORRECT
if (condition) return true;
return false;
```

### 2. Validate at the Top
Check all your invalid conditions at the very beginning of the function. These are your "Guard Clauses".

```csharp
public static void Process(ref int value)
{
    // Guards
    if (value < 0) return;
    
    // Main logic
    value *= 2;
}
```

### 3. Keep it Flat
If you find yourself nesting more than 2 levels deep, stop. Refactor.

---

## Real-World Examples

### Example 1: Timer Validation

#### ❌ Before (Nested)

```csharp
public static bool TryStartTimer(ref float current, float duration, bool canRestart)
{
    if (canRestart)
    {
        if (duration > 0)
        {
            if (current <= 0)
            {
                current = duration;
                return true;
            }
            else
            {
                return false; // Already running
            }
        }
        else
        {
            return false; // Invalid duration
        }
    }
    else
    {
        return false; // Cannot restart
    }
}
```

#### ✅ After (Early Exit)

```csharp
[MethodImpl(MethodImplOptions.AggressiveInlining)]
public static bool TryStartTimer(ref float current, float duration, bool canRestart)
{
    // Guard clauses
    if (!canRestart) return false;
    if (duration <= 0) return false;
    if (current > 0) return false; // Already running
    
    // Main logic - look how clean this is!
    current = duration;
    return true;
}
```

---

### Example 2: Inventory Add

#### ❌ Before

```csharp
public static bool TryAdd(ref int count, int max, int amount)
{
    bool success = false;
    if (amount > 0)
    {
        if (count < max)
        {
            int newCount = count + amount;
            if (newCount <= max)
            {
                count = newCount;
                success = true;
            }
            else
            {
                count = max;
                success = false; // Partial add
            }
        }
    }
    return success;
}
```

#### ✅ After

```csharp
[MethodImpl(MethodImplOptions.AggressiveInlining)]
public static bool TryAdd(ref int count, int max, int amount)
{
    if (amount <= 0) return false;
    if (count >= max) return false;
    
    int newCount = count + amount;
    if (newCount > max)
    {
        count = max;
        return false; // Partial add (hit cap)
    }
    
    count = newCount;
    return true;
}
```

---

## Guard Clauses in Action

A **Guard Clause** is just an `if` statement that returns early.

### Common Patterns

*   **Null Check:** `if (obj == null) return;`
*   **Range Check:** `if (index < 0 || index >= count) return;`
*   **State Check:** `if (!isActive) return;`

### Template

```csharp
[MethodImpl(MethodImplOptions.AggressiveInlining)]
public static bool TryProcess(ref int value, int min, int max)
{
    // === VALIDATION SECTION ===
    if (value < min) return false;
    if (value > max) return false;
    if (min >= max) return false;
    
    // === MAIN LOGIC ===
    value = (value - min) * 2 + min;
    
    return true;
}
```

---

## Performance Benefits

It's not just about readability. Early exits help the CPU.

1.  **Branch Prediction:** Modern CPUs are good at guessing which way code will go. Early exits make the "happy path" very linear and predictable.
2.  **Burst Compiler:** Burst loves linear code. It can optimize it much better than a tree of nested `if`s.
3.  **Cache:** Linear code fits better in the instruction cache.

---

## Common Mistakes

### 1. Hidden Else Blocks
Sometimes you write an `else` without writing `else`.

```csharp
// ❌ WRONG
if (condition)
{
    value = 1;
}
value = 2; // This overwrites value=1! It acts like an else/default but it's buggy.
```

**Fix:**
```csharp
// ✅ CORRECT
if (condition)
{
    value = 1;
    return; // Explicit return
}
value = 2;
```

### 2. Boolean Flags
Don't use a `success` flag. Just return.

```csharp
// ❌ WRONG
bool isValid = true;
if (a < 0) isValid = false;
if (isValid) { ... }
```

**Fix:**
```csharp
// ✅ CORRECT
if (a < 0) return false;
// Do work
```

---

**Next:** [Chapter 3: Return Values - The TryX Pattern →](./03-tryx-pattern.md)
