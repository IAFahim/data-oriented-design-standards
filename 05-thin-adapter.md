# Chapter 5: Unity Integration – The Thin Adapter Pattern

> **"MonoBehaviours are views, not models. They route input and display output. Nothing more."**

---

## Table of Contents

1. [The Problem with Fat MonoBehaviours](#the-problem-with-fat-monobehaviours)
2. [What is a Thin Adapter?](#what-is-a-thin-adapter)
3. [The Rules](#the-rules)
4. [Adapter Responsibilities](#adapter-responsibilities)
5. [Real-World Examples](#real-world-examples)
6. [Testing Without Unity](#testing-without-unity)

---

## The Problem with Fat MonoBehaviours

We've all written this class. It starts small—just moving a character. Then you add input handling. Then health logic. Then audio. Then UI updates.

Suddenly, you have a 2000-line `PlayerController.cs` that does *everything*.

```csharp
// ❌ FAT MONOBEHAVIOUR
public class PlayerController : MonoBehaviour
{
    public float speed = 5f;
    private float health = 100f;
    
    private void Update()
    {
        // Input logic...
        // Movement math...
        // Collision checks...
        // Audio triggers...
        // UI updates...
    }
}
```

**Why this kills your project:**
1.  **Untestable:** You can't write a unit test for `Update()`. You have to press "Play".
2.  **Locked to Unity:** You can't run this logic on a server without Unity.
3.  **Spaghetti:** Everything touches everything else.

---

## What is a Thin Adapter?

A **Thin Adapter** is a MonoBehaviour that does **zero thinking**. It's a dumb pipe.

**It connects two worlds:**
1.  **Unity World:** Input, Transform, Physics, Audio, Rendering.
2.  **Logic World:** Your pure C# static classes.

**The Flow:**
1.  Adapter gathers data from Unity (Input, Transform).
2.  Adapter calls a static Logic function.
3.  Adapter applies the result back to Unity (Transform, Audio).

---

## The Rules

### 1. No Logic in MonoBehaviour
If you are writing an `if` statement that calculates game rules, stop. Move it to a static class.

```csharp
// ❌ WRONG
void Update() {
    if (health < 0) Die(); // Logic!
}

// ✅ CORRECT
void Update() {
    if (HealthLogic.IsDead(health)) Die(); // Logic delegated
}
```

### 2. Only Unity APIs in MonoBehaviour
The only reason to have a MonoBehaviour is to access things like `transform`, `GetComponent`, `Input`, or `OnCollisionEnter`.

### 3. State Lives in Structs
Don't store loose fields in your MonoBehaviour. Store them in a data struct.

```csharp
// ❌ WRONG
public class Player : MonoBehaviour {
    public float health;
    public float maxHealth;
}

// ✅ CORRECT
public class Player : MonoBehaviour {
    public HealthData data; // Struct
}
```

---

## Adapter Responsibilities

Your adapter has three jobs:

1.  **Input:** "Hey Unity, is the space bar pressed?"
2.  **Delegate:** "Hey Logic, here's the input. What happens?"
3.  **Output:** "Hey Unity, move the transform to this new position."

---

## Real-World Examples

### Example 1: Movement

#### The Logic (Pure C#)
```csharp
public static class MovementLogic
{
    public static void Move(ref Vector3 currentPos, Vector3 direction, float speed, float dt)
    {
        currentPos += direction * speed * dt;
    }
}
```

#### The Adapter (MonoBehaviour)
```csharp
public class MovementAdapter : MonoBehaviour
{
    public float Speed = 5f;
    
    void Update()
    {
        // 1. Input
        float h = Input.GetAxis("Horizontal");
        float v = Input.GetAxis("Vertical");
        Vector3 dir = new Vector3(h, 0, v);
        Vector3 pos = transform.position;
        
        // 2. Delegate
        MovementLogic.Move(ref pos, dir, Speed, Time.deltaTime);
        
        // 3. Output
        transform.position = pos;
    }
}
```

---

### Example 2: Collision

#### The Logic
```csharp
public static class CombatLogic
{
    public static bool ShouldDie(int health, int damage)
    {
        return (health - damage) <= 0;
    }
}
```

#### The Adapter
```csharp
public class PlayerAdapter : MonoBehaviour
{
    public int Health = 100;
    
    void OnCollisionEnter(Collision other)
    {
        int damage = 10; // Simplified
        
        // Delegate logic
        if (CombatLogic.ShouldDie(Health, damage))
        {
            // Output (Unity API)
            Destroy(gameObject);
        }
        else
        {
            Health -= damage;
        }
    }
}
```

---

## Testing Without Unity

Because your logic is in static classes, you can test it instantly.

```csharp
[Test]
public void Movement_CalculatesCorrectly()
{
    Vector3 pos = Vector3.zero;
    MovementLogic.Move(ref pos, Vector3.forward, 10f, 1f);
    
    Assert.AreEqual(new Vector3(0, 0, 10), pos);
}
```

**No Unity Editor required. No "Play" button. Just instant results.**

---

**Next:** [Chapter 6: Optimization - The AV Pattern →](./06-av-pattern.md)
