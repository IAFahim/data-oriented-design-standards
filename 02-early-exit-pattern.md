# Chapter 2: Control Flow – The Early Exit Pattern

> **"Flatten your code. Reduce cyclomatic complexity. Validate early, exit fast."**  
> — *The Path to Maintainable Logic*

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
9. [When to Break the Rules](#when-to-break-the-rules)

---

## The Problem with Nested Logic

Traditional programming encourages nested `if-else` blocks that quickly become unmaintainable:

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

### Why This Fails

```mermaid
graph TD
    A[Start] --> B{isInvincible?}
    B -->|No| C{damage > 0?}
    B -->|Yes| Z[Return true]
    C -->|Yes| D{health > 0?}
    C -->|No| Z
    D -->|Yes| E[health -= damage]
    D -->|No| Y[Return false]
    E --> F{health < 0?}
    F -->|Yes| G[health = 0]
    F -->|No| H[Return health > 0]
    G --> H
    
    style A fill:#95afc0
    style B fill:#ff6b6b
    style C fill:#ff6b6b
    style D fill:#ff6b6b
    style F fill:#ff6b6b
    style Z fill:#feca57
    style Y fill:#feca57
    style H fill:#feca57
```

**Problems:**
- **Cyclomatic Complexity: 7** (max recommended: 3)
- **Indentation: 5 levels** (max recommended: 2)
- **Else blocks everywhere** (banned in DOD)
- **Hard to test** (2^7 = 128 possible paths)
- **Impossible to read** after 6 months

---

## What is the Early Exit Pattern?

The **Early Exit Pattern** validates inputs at the top of a function and returns immediately on failure.

### The Mental Shift

```mermaid
graph LR
    subgraph Traditional_Nested
        direction TB
        T1[Check condition 1] --> T2{Valid?}
        T2 -->|Yes| T3[Check condition 2]
        T2 -->|No| TE1[Handle error]
        T3 --> T4{Valid?}
        T4 -->|Yes| T5[Main logic]
        T4 -->|No| TE2[Handle error]
    end
    
    subgraph Early_Exit_Flat
        direction TB
        E1[Check condition 1] --> E2{Valid?}
        E2 -->|No| EE1[return early]
        E2 -->|Yes| E3[Check condition 2]
        E3 --> E4{Valid?}
        E4 -->|No| EE2[return early]
        E4 -->|Yes| E5[Main logic]
        E5 --> ER[return result]
    end
    
    style T1 fill:#ff6b6b
    style T3 fill:#ff6b6b
    style T5 fill:#ff6b6b
    style TE1 fill:#ee5a6f
    style TE2 fill:#ee5a6f
    
    style E1 fill:#26de81
    style E3 fill:#26de81
    style E5 fill:#26de81
    style EE1 fill:#48dbfb
    style EE2 fill:#48dbfb
    style ER fill:#1dd1a1
```

---

## The Rules

### Rule 1: No `else` Blocks (MANDATORY)

```csharp
// ❌ WRONG - else block
if (condition)
{
    return true;
}
else
{
    return false;
}

// ✅ CORRECT - early return
if (condition)
{
    return true;
}
return false;
```

### Rule 2: Validate at the Top

```csharp
// ❌ WRONG - validation scattered
public static void Process(ref int value)
{
    value *= 2;
    if (value < 0) return; // Too late!
}

// ✅ CORRECT - validate first
public static void Process(ref int value)
{
    if (value < 0) return; // Guard clause
    
    value *= 2; // Main logic
}
```

### Rule 3: Maximum 2 Levels of Indentation

```csharp
// ❌ WRONG - 4 levels
public static bool Process(int a, int b)
{
    if (a > 0)
    {
        if (b > 0)
        {
            if (a > b)
            {
                if (a % 2 == 0)
                {
                    return true; // 4 levels deep!
                }
            }
        }
    }
    return false;
}

// ✅ CORRECT - flat structure
public static bool Process(int a, int b)
{
    if (a <= 0) return false;
    if (b <= 0) return false;
    if (a <= b) return false;
    if (a % 2 != 0) return false;
    
    return true; // Success path
}
```

### Rule 4: One Return Type Per Function

```csharp
// ❌ WRONG - unclear return meaning
public static int GetValue()
{
    if (invalid) return -1; // Error sentinel?
    return 42; // Actual value?
}

// ✅ CORRECT - use bool + out
public static bool TryGetValue(out int value)
{
    if (invalid)
    {
        value = default;
        return false; // Clear failure
    }
    
    value = 42;
    return true; // Clear success
}
```

---

## Cyclomatic Complexity Explained

**Cyclomatic Complexity** measures the number of linearly independent paths through code.

### Formula
```
CC = E - N + 2P
Where:
  E = edges in control flow graph
  N = nodes
  P = connected components (usually 1)
```

**Simpler formula:**
```
CC = 1 + (number of decision points)
Decision points = if, for, while, case, &&, ||, ?:
```

### Complexity Ratings

```mermaid
graph LR
    subgraph Complexity_Scale
        C1["1-3<br/>Simple<br/>Low Risk"]
        C2["4-7<br/>Moderate<br/>Medium Risk"]
        C3["8-10<br/>Complex<br/>High Risk"]
        C4["11+<br/>Untestable<br/>Refactor"]
    end
    
    style C1 fill:#26de81
    style C2 fill:#feca57
    style C3 fill:#ff6b6b
    style C4 fill:#ee5a6f
```

### Example Analysis

```csharp
// ❌ BAD: Cyclomatic Complexity = 7
public static bool Validate(int a, int b, int c)
{
    if (a > 0)              // +1
    {
        if (b > 0)          // +1
        {
            if (c > 0)      // +1
            {
                if (a > b && b > c)  // +2 (&& is a decision)
                {
                    return true;
                }
            }
        }
    }
    return false;
}
// Total: 1 (base) + 6 = 7

// ✅ GOOD: Cyclomatic Complexity = 5
public static bool Validate(int a, int b, int c)
{
    if (a <= 0) return false;      // +1
    if (b <= 0) return false;      // +1
    if (c <= 0) return false;      // +1
    if (a <= b) return false;      // +1
    if (b <= c) return false;      // +1
    
    return true;
}
// Total: 1 (base) + 5 = 6
// Still better readability despite same CC
```

### Visual Comparison

```mermaid
graph TB
    subgraph Nested_CC7
        direction TB
        N1[Start] --> N2{a greater 0?}
        N2 -->|No| NF[Return false]
        N2 -->|Yes| N3{b greater 0?}
        N3 -->|No| NF
        N3 -->|Yes| N4{c greater 0?}
        N4 -->|No| NF
        N4 -->|Yes| N5{a greater b?}
        N5 -->|No| NF
        N5 -->|Yes| N6{b greater c?}
        N6 -->|No| NF
        N6 -->|Yes| NT[Return true]
        
        style N2 fill:#ff6b6b
        style N3 fill:#ff6b6b
        style N4 fill:#ff6b6b
        style N5 fill:#ff6b6b
        style N6 fill:#ff6b6b
        style NF fill:#ee5a6f
    end
    
    subgraph Early_Exit_CC6
        direction TB
        E1[Start] --> E2{a less equal 0?}
        E2 -->|Yes| EF1[Return false]
        E2 -->|No| E3{b less equal 0?}
        E3 -->|Yes| EF2[Return false]
        E3 -->|No| E4{c less equal 0?}
        E4 -->|Yes| EF3[Return false]
        E4 -->|No| E5{a less equal b?}
        E5 -->|Yes| EF4[Return false]
        E5 -->|No| E6{b less equal c?}
        E6 -->|Yes| EF5[Return false]
        E6 -->|No| ET[Return true]
        
        style E2 fill:#26de81
        style E3 fill:#26de81
        style E4 fill:#26de81
        style E5 fill:#26de81
        style E6 fill:#26de81
        style ET fill:#1dd1a1
        style EF1 fill:#48dbfb
        style EF2 fill:#48dbfb
        style EF3 fill:#48dbfb
        style EF4 fill:#48dbfb
        style EF5 fill:#48dbfb
    end
```

**Notice:** Early exit creates a **linear flow** (top to bottom), while nesting creates a **tree** (exponential paths).

---

## Real-World Examples

### Example 1: Damage Processing

#### ❌ Before (Nested)

```csharp
public static bool TryApplyDamage(ref int current, int damage, bool isInvincible)
{
    if (!isInvincible)
    {
        if (damage > 0)
        {
            if (current > 0)
            {
                current -= damage;
                if (current < 0)
                {
                    current = 0;
                }
                return current > 0;
            }
        }
    }
    return true;
}
```

**Cyclomatic Complexity: 5**  
**Indentation: 4 levels**

---

#### ✅ After (Early Exit)

```csharp
[MethodImpl(MethodImplOptions.AggressiveInlining)]
public static bool TryApplyDamage(ref int current, int damage, bool isInvincible)
{
    // Guard clauses
    if (isInvincible) return true;
    if (damage <= 0) return true;
    if (current <= 0) return false;
    
    // Main logic
    current -= damage;
    if (current < 0) current = 0;
    
    return current > 0;
}
```

**Cyclomatic Complexity: 4**  
**Indentation: 1 level**  
**Lines saved: 6**

---

### Example 2: Cooldown Reset

#### ❌ Before

```csharp
public static void Reset(ref float current, float max, bool canReset)
{
    if (canReset)
    {
        if (max > 0)
        {
            if (current != max)
            {
                current = max;
            }
        }
    }
}
```

---

#### ✅ After

```csharp
[MethodImpl(MethodImplOptions.AggressiveInlining)]
public static void Reset(ref float current, float max, bool canReset)
{
    if (!canReset) return;
    if (max <= 0) return;
    if (current == max) return;
    
    current = max;
}
```

---

### Example 3: Inventory Add

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

---

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

**Guard Clauses** are the building blocks of the Early Exit Pattern.

### Anatomy of a Guard Clause

```csharp
if (invalidCondition) return defaultValue;
// ^^^                 ^^^^^^
// Check               Exit immediately
```

### Common Patterns

```mermaid
graph TD
    subgraph Guard_Clause_Types
        G1["Null Check<br/>if value equals null return"]
        G2["Range Check<br/>if value less 0 or greater max return"]
        G3["State Check<br/>if not isValid return"]
        G4["Divide by Zero<br/>if divisor equals 0 return"]
        G5["Empty Collection<br/>if list Length equals 0 return"]
    end
    
    style G1 fill:#48dbfb
    style G2 fill:#48dbfb
    style G3 fill:#48dbfb
    style G4 fill:#48dbfb
    style G5 fill:#48dbfb
```

### Template

```csharp
[MethodImpl(MethodImplOptions.AggressiveInlining)]
public static bool TryProcess(ref int value, int min, int max)
{
    // === VALIDATION SECTION ===
    // Guard clauses go here (top of function)
    if (value < min) return false;
    if (value > max) return false;
    if (min >= max) return false;
    
    // === MAIN LOGIC ===
    // Happy path logic (no nesting)
    value = (value - min) * 2 + min;
    
    return true;
}
```

---

## Performance Benefits

### Branch Prediction

Modern CPUs predict branch outcomes. Early exits help the predictor.

```mermaid
graph LR
    subgraph Nested_Hard_to_Predict
        N1[if A] --> N2{branch}
        N2 --> N3[if B]
        N3 --> N4{branch}
        N4 --> N5[if C]
        N5 --> N6{branch}
        
        style N2 fill:#ff6b6b
        style N4 fill:#ff6b6b
        style N6 fill:#ff6b6b
    end
    
    subgraph Early_Exit_Predictable
        E1[if not A return] --> E2[if not B return]
        E2 --> E3[if not C return]
        E3 --> E4[main logic]
        
        style E1 fill:#26de81
        style E2 fill:#26de81
        style E3 fill:#26de81
        style E4 fill:#1dd1a1
    end
```

**Why?**
- Early exits fail fast → CPU learns "usually continues"
- Nested logic has unpredictable interleaved branches

### Burst Compiler Optimization

Burst can better optimize linear code:

```csharp
// ✅ BURST-FRIENDLY
[MethodImpl(MethodImplOptions.AggressiveInlining)]
public static void Tick(ref float current, float delta)
{
    if (current <= 0) return; // Burst: branch hint
    
    current -= delta; // Burst: vectorize this
    if (current < 0) current = 0; // Burst: cmov instruction
}
```

**Generated Assembly (simplified):**
```asm
; Check if current <= 0
vcomiss xmm0, xmm1
jbe     exit        ; Jump if below/equal (early exit)

; Main logic (vectorized)
vsubss  xmm0, xmm0, xmm2  ; current -= delta
vxorps  xmm1, xmm1, xmm1  ; zero register
vmaxss  xmm0, xmm0, xmm1  ; clamp to zero (no branch!)

exit:
ret
```

### Cache Performance

```mermaid
graph TB
    subgraph Instruction_Cache
        IC1["Guard 1: 2 bytes"]
        IC2["Guard 2: 2 bytes"]
        IC3["Guard 3: 2 bytes"]
        IC4["Main Logic: 10 bytes"]
        IC5["Return: 1 byte"]
    end
    
    subgraph Cache_Line_64bytes
        CL["Entire function fits in 1 cache line"]
    end
    
    IC1 --> CL
    IC2 --> CL
    IC3 --> CL
    IC4 --> CL
    IC5 --> CL
    
    style CL fill:#26de81
    style IC1 fill:#48dbfb
    style IC2 fill:#48dbfb
    style IC3 fill:#48dbfb
    style IC4 fill:#1dd1a1
    style IC5 fill:#48dbfb
```

**Benefit:** Linear code = fewer instruction cache misses.

---

## Common Mistakes

### Mistake 1: Hidden Else Blocks

```csharp
// ❌ WRONG - implicit else
if (condition)
{
    value = 1;
}
value = 2; // This is an "else" in disguise!
```

**Fix:**
```csharp
// ✅ CORRECT - explicit early return
if (condition)
{
    value = 1;
    return;
}
value = 2;
```

---

### Mistake 2: Boolean Flags

```csharp
// ❌ WRONG - flag-driven logic
bool isValid = true;
if (a < 0) isValid = false;
if (b < 0) isValid = false;
if (isValid)
{
    // Main logic
}
```

**Fix:**
```csharp
// ✅ CORRECT - immediate return
if (a < 0) return false;
if (b < 0) return false;

// Main logic
return true;
```

---

### Mistake 3: Unnecessary Nesting for Positive Checks

```csharp
// ❌ WRONG - nesting for positive case
if (isEnabled)
{
    // 50 lines of logic
}

// ✅ CORRECT - invert and exit
if (!isEnabled) return;

// 50 lines of logic (no indentation!)
```

---

### Mistake 4: Multiple Returns in Complex Logic

```csharp
// ⚠️ ACCEPTABLE BUT FRAGILE
public static int Calculate(int a, int b)
{
    if (a < 0) return -1;
    if (b < 0) return -2;
    if (a == 0) return 0;
    if (b == 0) return 0;
    return a / b;
}

// ✅ BETTER - consistent return pattern
public static bool TryCalculate(int a, int b, out int result)
{
    result = default;
    
    if (a < 0) return false;
    if (b < 0) return false;
    if (b == 0) return false; // Prevent div by zero
    
    result = a / b;
    return true;
}
```

---

## When to Break the Rules

### Exception 1: Performance-Critical Positive Checks

```csharp
// ✅ ACCEPTABLE - hot path optimization
public static void ProcessHotPath(ref int value, bool fastMode)
{
    if (fastMode) // 95% of calls
    {
        value *= 2; // Inlined fast path
        return;
    }
    
    // Rare slow path
    value = ComplexCalculation(value);
}
```

**Reason:** CPU branch predictor will strongly prefer the fast path.

---

### Exception 2: Single-Statement Ternary

```csharp
// ✅ ACCEPTABLE - cleaner than if-else
return value > 0 ? value : 0;

// vs verbose alternative
if (value <= 0) return 0;
return value;
```

---

### Exception 3: Switch Statements (State Machines)

```csharp
// ✅ ACCEPTABLE - state machines need switches
switch (state)
{
    case State.Idle:
        ProcessIdle();
        return;
    case State.Running:
        ProcessRunning();
        return;
    default:
        return;
}
```

---

## Summary

### The Early Exit Checklist

```mermaid
graph TD
    START[Start Writing Function] --> Q1{Guard clauses at top?}
    Q1 -->|No| FIX1[Move validation to top]
    Q1 -->|Yes| Q2{Any else blocks?}
    Q2 -->|Yes| FIX2[Remove else use early return]
    Q2 -->|No| Q3{Indentation greater 2 levels?}
    Q3 -->|Yes| FIX3[Flatten with guard clauses]
    Q3 -->|No| Q4{Cyclomatic Complexity greater 3?}
    Q4 -->|Yes| FIX4[Split into smaller functions]
    Q4 -->|No| DONE[Good to go]
    
    FIX1 --> Q1
    FIX2 --> Q2
    FIX3 --> Q3
    FIX4 --> Q4
    
    style START fill:#95afc0
    style DONE fill:#26de81
    style FIX1 fill:#feca57
    style FIX2 fill:#feca57
    style FIX3 fill:#feca57
    style FIX4 fill:#feca57
    style Q1 fill:#48dbfb
    style Q2 fill:#48dbfb
    style Q3 fill:#48dbfb
    style Q4 fill:#48dbfb
```
    style Q3 fill:#48dbfb
    style Q4 fill:#48dbfb
```

### Key Principles

| Principle | Rule |
|-----------|------|
| **No else blocks** | Use early returns instead |
| **Guard clauses first** | Validate at the top |
| **Max 2 levels** | Flatten deeply nested code |
| **Linear flow** | Top to bottom, no jumping |
| **One concern per check** | Each guard clause tests one thing |
| **Fail fast** | Return immediately on invalid input |

### Before/After Template

```csharp
// ❌ BEFORE
public static bool Process(int a, int b, int c)
{
    if (a > 0)
    {
        if (b > 0)
        {
            if (c > 0)
            {
                return Calculate(a, b, c);
            }
            else
            {
                return false;
            }
        }
        else
        {
            return false;
        }
    }
    else
    {
        return false;
    }
}

// ✅ AFTER
[MethodImpl(MethodImplOptions.AggressiveInlining)]
public static bool TryProcess(int a, int b, int c, out int result)
{
    result = default;
    
    // Guard clauses
    if (a <= 0) return false;
    if (b <= 0) return false;
    if (c <= 0) return false;
    
    // Main logic
    result = Calculate(a, b, c);
    return true;
}
```

---

**Previous:** [← Chapter 1: The Golden Rule](./01-the-golden-rule.md)  
**Next:** [Chapter 3: Return Values - The TryX Pattern →](./03-tryx-pattern.md)

---

## Quick Reference Card

```csharp
// THE EARLY EXIT PATTERN

// 1. Guard clauses at the top
if (invalid1) return defaultValue;
if (invalid2) return defaultValue;
if (invalid3) return defaultValue;

// 2. Main logic (no nesting)
ProcessData();

// 3. Return success
return successValue;

// ===========================
// BANNED PATTERNS
// ===========================

// ❌ NO else blocks
if (condition) { } else { }

// ❌ NO nested if statements (>2 levels)
if (a) { if (b) { if (c) { } } }

// ❌ NO boolean flags for control flow
bool flag = true;
if (x) flag = false;
if (flag) { }
```

---

*Master the Early Exit Pattern, and your code becomes self-documenting, testable, and Burst-friendly.*
