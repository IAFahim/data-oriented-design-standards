# Data-Oriented Design Standards

> **Standards for writing scalable, testable, Unity-independent, zero-allocation, Burst-compatible code—written once and built to last.**

---

## 📚 The Complete Guide

### **Foundation: The Core Patterns**

#### [**Chapter 1: The Golden Rule – The Data-Logic-Extension Triad**](./01-the-golden-rule.md)
The absolute foundation. Separate data (structs), logic (static classes), and syntax (extensions). Never mix them.

**You'll Learn:**
- Why OOP fails in high-performance games
- The three-layer architecture
- Memory layout optimization
- SIMD vectorization benefits
- Cache-friendly design

**Key Takeaway:** *Separate what changes from what doesn't.*

---

#### [**Chapter 2: Control Flow – The Early Exit Pattern**](./02-early-exit-pattern.md)
Flatten your code. Ban `else` blocks. Validate early, exit fast.

**You'll Learn:**
- Guard clauses mastery
- Cyclomatic complexity reduction
- Branch prediction optimization
- How Burst compiles flat code

**Key Takeaway:** *If complexity > 3, you're doing it wrong.*

---

#### [**Chapter 3: Return Values – The TryX Pattern**](./03-tryx-pattern.md)
Make success/failure explicit. Use `bool` + `out` parameters. Avoid exceptions for control flow.

**You'll Learn:**
- TryX pattern rules
- vs Exceptions comparison
- Multiple return values
- Zero-allocation returns

**Key Takeaway:** *Exceptions are for exceptions, not logic.*

---

#### [**Chapter 4: Performance – Burst Compatibility & Primitive Logic**](./04-burst-compatibility.md)
Write C# that compiles to C++ performance. Pass primitives, inline everything.

**You'll Learn:**
- Burst compilation rules
- Why primitives only
- Method inlining deep dive
- Burst Inspector usage
- Common Burst errors

**Key Takeaway:** *If it's not primitives, Burst can't vectorize it.*

---

### **Unity Integration: Bridging to Unity**

#### [**Chapter 5: Unity Integration – The Thin Adapter Pattern**](./05-thin-adapter.md)
MonoBehaviours are views, not models. They route input, apply output. Zero logic.

**You'll Learn:**
- MonoBehaviour as adapter layer
- Zero logic in Unity classes
- Testing without Unity
- Reusing logic in Burst jobs

**Key Takeaway:** *If your MonoBehaviour has math, you're doing it wrong.*

---

#### [**Chapter 6: Optimization – The AV Pattern (Cached Components)**](./06-av-pattern.md)
Never call `GetComponent` twice. Static dictionary. 5-50x faster.

**You'll Learn:**
- Static component caching
- Performance benchmarks
- When to cache vs not
- Scene load cache clearing

**Key Takeaway:** *First call: 500ns. Cached: 10ns. Cache everything.*

---

### **Data & Architecture: The Complete System**

#### [**Chapter 7: Data Structure – Blittable Types & Memory Layout**](./07-blittable-types.md)
If it can't memcpy, it's not DOD. Sequential layout. No references.

**You'll Learn:**
- What makes types blittable
- Memory alignment & padding
- Field ordering optimization
- Validation techniques

**Key Takeaway:** *Blittable = Burst-compatible = Fast.*

---

#### [**Chapter 8: Architecture – Assembly Separation (Core vs. Runtime)**](./08-assembly-separation.md)
Game logic should compile without UnityEngine.dll. Test without Unity. Deploy everywhere.

**You'll Learn:**
- Two-assembly architecture
- Assembly definition files
- Testing strategy
- Platform independence

**Key Takeaway:** *Core = logic. Runtime = Unity. One-way dependency.*

---

## 🚀 Quick Start

### 1. **Start with Chapter 1**
Understand the Data-Logic-Extension Triad before anything else.

### 2. **Apply Chapter 2**
Refactor your methods to use Early Exit pattern.

### 3. **Master Chapter 3**
Convert all your return patterns to TryX.

### 4. **Optimize with Chapters 4-6**
Make it Burst-compatible and fast.

### 5. **Structure with Chapters 7-8**
Organize your project architecture.

---

## 📊 Performance Impact

| Pattern | Before | After | Speedup |
|---------|--------|-------|---------|
| **SIMD Vectorization** | 1000 cycles | 125 cycles | **8x** |
| **Component Caching** | 500 ns/call | 10 ns/call | **50x** |
| **Memory Layout** | 1000 cache misses | 125 cache hits | **8x** |
| **Burst Compilation** | 200 ns | 8 ns | **25x** |

**Combined Effect:** 10-100x performance improvement on hot paths.

---

## 🎯 The Philosophy

### **The Golden Rules**

1. **Data-Logic-Extension Triad** - Never mix the three layers
2. **Early Exit Pattern** - Ban `else`, validate early
3. **TryX Pattern** - `bool` + `out`, not exceptions
4. **Primitives Only** - Logic methods take `ref int`, not structs
5. **Thin Adapters** - MonoBehaviours route, never compute
6. **Cache Everything** - GetComponent once, cache forever
7. **Blittable Types** - If it can't memcpy, refactor it
8. **Assembly Separation** - Core has zero Unity dependencies

---

## 🛠️ Project Template

```
MyGame/
├── Assets/
│   ├── Scripts/
│   │   ├── Core/                          [Game.Core Assembly]
│   │   │   ├── Game.Core.asmdef          (noEngineReferences: true)
│   │   │   ├── Data/
│   │   │   │   ├── PlayerData.cs         [Serializable, Sequential]
│   │   │   │   └── WeaponData.cs
│   │   │   ├── Logic/
│   │   │   │   ├── PlayerLogic.cs        [Static class, primitives only]
│   │   │   │   └── CombatLogic.cs
│   │   │   └── Extensions/
│   │   │       └── PlayerExtensions.cs   [Adapter layer]
│   │   │
│   │   └── Runtime/                       [Game.Runtime Assembly]
│   │       ├── Game.Runtime.asmdef       (references: Game.Core)
│   │       └── Adapters/
│   │           └── PlayerAdapter.cs      [MonoBehaviour, thin]
│   │
│   └── Tests/                             [Game.Tests Assembly]
│       ├── Game.Tests.asmdef
│       └── Core/
│           └── PlayerLogicTests.cs       [Fast, no Unity]
```

---

## 📖 Reading Order

### **For Beginners:**
1. Chapter 1 (Architecture)
2. Chapter 5 (Unity Integration)
3. Chapter 2 (Control Flow)
4. Rest in order

### **For Experienced Developers:**
1. Chapter 1 (Triad)
2. Chapter 8 (Assembly Separation)
3. Chapters 2-7 as needed

### **For Performance Optimization:**
1. Chapter 4 (Burst)
2. Chapter 6 (Caching)
3. Chapter 7 (Memory Layout)

---

## 🧪 Testing Philosophy

```mermaid
graph TB
    subgraph Test_Pyramid
        T1["Integration Tests (10%)<br/>Unity required<br/>~10 seconds"]
        T2["Unit Tests (90%)<br/>No Unity required<br/>~1 second"]
    end
    
    T2 --> T1
    
    style T1 fill:#feca57
    style T2 fill:#26de81
```

**Goal:** 90% of your code should be testable without starting Unity.

---

## 💡 Key Concepts

### **The Triad**
```csharp
// Layer A: Data (struct)
[Serializable]
[StructLayout(LayoutKind.Sequential)]
public struct PlayerData
{
    public float Health;
    public float Speed;
}

// Layer B: Logic (static class)
public static class PlayerLogic
{
    [MethodImpl(MethodImplOptions.AggressiveInlining)]
    public static void Move(ref float x, ref float y, float speed, float deltaTime)
    {
        if (speed <= 0) return; // Early exit
        x += speed * deltaTime;
    }
}

// Layer C: Extensions (adapter)
public static class PlayerExtensions
{
    public static void Move(ref this PlayerData player, float deltaTime)
    {
        PlayerLogic.Move(ref player.X, ref player.Y, player.Speed, deltaTime);
    }
}
```

---

## 🎓 Learning Path

```mermaid
graph LR
    START[Start Here] --> CH1[Ch1: Triad]
    CH1 --> CH2[Ch2: Early Exit]
    CH2 --> CH3[Ch3: TryX]
    CH3 --> CH4[Ch4: Burst]
    CH4 --> CH5[Ch5: Adapters]
    CH5 --> CH6[Ch6: Caching]
    CH6 --> CH7[Ch7: Blittable]
    CH7 --> CH8[Ch8: Assembly]
    CH8 --> MASTER[Master DOD]
    
    style START fill:#95afc0
    style MASTER fill:#26de81
```

---

## 🔥 Common Pitfalls

| ❌ Don't Do | ✅ Do Instead |
|------------|--------------|
| Logic in structs | Static logic classes |
| Logic in MonoBehaviours | Thin adapters |
| Passing structs to logic | Pass primitives (`ref int`, `ref float`) |
| `GetComponent` every frame | Cache with AV Pattern |
| `else` blocks | Early exit pattern |
| Exceptions for control flow | TryX pattern |
| References in structs | Blittable types only |
| Unity types in Core | Platform-independent types |

---

## 🤝 Contributing

Found an issue? Have a suggestion? Open an issue or PR!

**Guidelines:**
- Keep each chapter focused on ONE topic
- No overlaps between chapters
- Real-world examples only
- Include benchmarks for performance claims

---

## 📜 License

MIT License - Use freely in your projects.

---

## 🙏 Acknowledgments

Built on the shoulders of giants:
- Unity DOTS Team
- Burst Compiler Team

---

## 🚀 Start Your Journey

**Ready to write production-grade, high-performance Unity code?**

👉 **[Start with Chapter 1: The Golden Rule](./01-the-golden-rule.md)**

---

*Write once. Optimize forever. Ship everywhere.*
