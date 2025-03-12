---
title: "typescript-go"
description: "TypeScript-Go: A 10x Faster Compiler with Go"
publishDate: "2025-03-11"
tags: ["typescript", "go", "Anders Hejlsberg", "c#", "csharp", "compiler"]
---

## What is typescript-go

Today Anders Hejlsberg published a new blog post titled [A 10x Faster TypeScript](https://devblogs.microsoft.com/typescript/typescript-native-port/), introducing a new Go-based TypeScript compiler that achieves 10x faster compilation speeds with significantly reduced memory usage.

### Why Rewrite the TypeScript Compiler

The current TypeScript compiler suffers from slow compilation speeds in large codebases due to its complex type system, a longstanding pain point in the TypeScript community. To improve development experience for large TypeScript projects, Anders decided to rewrite the compiler in Go.

### Performance Benchmarks

| Codebase                    | Size (LOC) | Current | Native | Speedup |
|-----------------------------|------------|---------|--------|---------|
| VS Code                     | 1,505,000  | 77.8s   | 7.5s   | 10.4x   |
| Playwright                  | 356,000    | 11.1s   | 1.1s   | 10.1x   |
| TypeORM                     | 270,000    | 17.5s   | 1.3s   | 13.5x   |
| date-fns                    | 104,000    | 6.5s    | 0.7s   | 9.5x    |
| tRPC (server + client)      | 18,000     | 5.5s    | 0.6s   | 9.1x    |
| rxjs (observable)           | 2,100      | 1.1s    | 0.1s   | 11.0x   |

### Community Reactions
As expected, in the [typescript-go](https://github.com/microsoft/typescript-go/discussions/411) discussions, the most pressing question was: "Why Go instead of C#?" Given Anders Hejlsberg's role as the creator of C#, many expected him to choose C# for the rewrite. C# offers strong performance and supports Native AOT, and being a Microsoft project itself, the choice of Go seemed puzzling.

Anders responded:
:::note
I believe Go is the lowest-level language we can adopt while retaining automatic garbage collection (GC), and the closest to native development we can get while maintaining GC capabilities.

C# was fundamentally designed with a bytecode-first philosophy. While it now offers AOT compilation support for some platforms, this coverage isn't universal. More importantly, C#'s AOT capabilities haven't been battle-tested for over a decade, and its underlying architecture wasn't originally designed for such scenarios.

Go demonstrates stronger capabilities in data structure layout and inline structs. Our current codebase follows a functional programming style—notably, the core TypeScript compiler doesn't even use classes. This aligns perfectly with Go's design philosophy.

Go is built around functions and data structures, while C# is deeply rooted in object-oriented programming (OOP). Adopting C# would force a shift to OOP paradigms, creating significantly more friction than migrating to Go. Ultimately, Go represents the path of least resistance for this evolution.
:::

However, much of the community expressed disappointment. Many believe these challenges could have been addressed through collaboration with the .NET team, creating a win-win scenario that would advance both C#'s Native AOT capabilities and TypeScript's performance.

### My Perspective
This project is an excellent initiative that will significantly enhance development experience for large TypeScript projects. However, the decision to avoid C# remains regrettable. Choosing C# could have demonstrated Microsoft's commitment to its own language while accelerating progress on C#'s Native AOT capabilities—a missed opportunity for mutual advancement.