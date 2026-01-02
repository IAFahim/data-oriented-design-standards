# Data-Oriented Design Standards

> **Battle-tested standards for writing scalable, testable, and high-performance Unity code. Write it once, build it to last.**

---

## 📚 The Guide

This isn't just a list of rules—it's a complete architectural philosophy. We're moving away from the "Unity way" of messy MonoBehaviours and towards a clean, data-oriented approach that scales.

### **The Core Patterns**

#### [**Chapter 1: The Golden Rule – The Data-Logic-Extension Triad**](./01-the-golden-rule.md)
The foundation of everything we do. We separate data (structs), logic (static classes), and syntax (extensions). Once you get this, everything else falls into place.

#### [**Chapter 2: Control Flow – The Early Exit Pattern**](./02-early-exit-pattern.md)
Stop writing arrow code. We flatten our logic, ban `else` blocks, and validate inputs immediately. It makes code readable and debuggable.

#### [**Chapter 3: Return Values – The TryX Pattern**](./03-tryx-pattern.md)
Exceptions are for exceptional circumstances, not game logic. We use `bool` returns with `out` parameters to handle success and failure explicitly and efficiently.

#### [**Chapter 4: Performance – Burst Compatibility**](./04-burst-compatibility.md)
How to write C# that runs like C++. We stick to primitives and inline aggressively so the Burst compiler can do its magic.

### **Unity Integration**

#### [**Chapter 5: The Thin Adapter Pattern**](./05-thin-adapter.md)
MonoBehaviours should be dumb. They're just adapters that route Unity events to your pure logic. Keep them thin, and your game becomes testable.

#### [**Chapter 6: The AV Pattern (Cached Components)**](./06-av-pattern.md)
`GetComponent` is slow. We fix that by caching everything in a static dictionary. It's a simple pattern that gives us massive performance gains for free.

### **Architecture & Data**

#### [**Chapter 7: Blittable Types & Memory Layout**](./07-blittable-types.md)
If you can't `memcpy` it, it doesn't belong in our hot path. We learn how to layout memory sequentially for maximum cache efficiency.

#### [**Chapter 8: Assembly Separation**](./08-assembly-separation.md)
Your core game logic shouldn't even know Unity exists. We split our code into Core (pure C#) and Runtime (Unity adapters) to enable lightning-fast testing and portability.

---

## 🚀 Quick Start

1.  **Read Chapter 1.** Seriously, don't skip it. It explains the "Triad" architecture that everything else is built on.
2.  **Refactor a small system.** Take a simple MonoBehaviour and split it into Data, Logic, and Adapter.
3.  **Adopt the patterns.** Start using Early Exit and TryX in your daily coding.
4.  **Optimize.** Once you're comfortable, look at Burst and the AV Pattern to speed things up.

---

## 📊 Why We Do This

| Pattern | The Old Way | The DOD Way | The Gain |
|---------|-------------|-------------|----------|
| **SIMD** | Scalar processing | Vectorized (4-8x) | **8x** |
| **Caching** | `GetComponent` spam | Static Dictionary | **50x** |
| **Memory** | Cache misses everywhere | Sequential layout | **8x** |
| **Burst** | Managed code | Native machine code | **25x** |

**Bottom line:** When you combine these, you get 10-100x performance improvements on your hot paths.

---

## 🎯 The Philosophy

1.  **Separate Concerns:** Data, Logic, and Extensions never mix.
2.  **Fail Fast:** Validate inputs immediately. No nested `if/else` spaghetti.
3.  **Be Explicit:** Success and failure should be obvious in the API (`TryX`).
4.  **Keep it Simple:** Primitives are faster than objects. Static is faster than instance.
5.  **Cache Everything:** If you need it more than once, cache it.
6.  **Testable Core:** Your game logic should run without Unity.

---

## 🤝 Contributing

Got a better way to do something? Found a typo? Open an issue or PR. We want this to be the best resource for high-performance Unity development.

---

## 📜 License

MIT License. Go build something awesome.

---

**Ready to dive in?**

👉 **[Start with Chapter 1: The Golden Rule](./01-the-golden-rule.md)**

---

*Write once. Optimize forever. Ship everywhere.*
