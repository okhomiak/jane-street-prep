# 18 · OCaml literacy (two evenings, no more)

[← Trading domain](../17-trading-domain/README.md) · [Index](../README.md) · [Next: Mock interviews →](../19-mock-interviews/README.md)

You will not code in OCaml in this loop. Jane Street says it directly: interview in the language you know best, there are no bonus points for OCaml, and "please don't use OCaml just because you think it will make us happy" ([SWE prep](https://www.janestreet.com/preparing-for-a-software-engineering-interview/), [FAQ](https://www.janestreet.com/join-jane-street/interviewing/)). Their own cybersecurity engineer, Minjie, had no OCaml before joining and every developer does an OCaml bootcamp after ([profile](https://www.janestreet.com/join-jane-street/get-to-know-us/minjie)). So the goal of this block is narrow: talk about OCaml credibly in the "why Jane Street" and hiring-manager conversations, read a snippet without flinching, and be honest that you have not shipped it. That is it. Tags: [JS] primary.

One caveat worth knowing: the security team does build tooling in OCaml. The NYC Senior Cybersecurity Operations Engineer posting is explicitly "(OCaml Experience)" ([posting](https://www.janestreet.com/join-jane-street/position/8687509002/)). You are not expected to arrive fluent, but do not say OCaml is irrelevant to the role.

---

## The "why OCaml" answer, in one breath

Have this ready, because it is a natural hiring-manager question and it shows you did the reading:

> A small, expressive type system with inference catches a whole class of bugs at compile time and keeps a large codebase legible, which matters when a few hundred programmers share one monorepo. It is garbage-collected, so programs are memory-safe by default. And OxCaml adds modes (locality, then ownership) so the hot paths can stack-allocate and avoid the garbage collector without giving up the type inference. You get safety and performance from the same language.

That is three claims: **types catch bugs early**, **GC gives memory safety**, **modes buy performance back**. Each is on Jane Street's own pages ([ocaml.org success story](https://ocaml.org/success-stories/large-scale-trading-system), [OxCaml locality post](https://blog.janestreet.com/oxidizing-ocaml-locality/)).

---

## The concepts to recognise

You should be able to explain these words if they come up. You do not need to write them fluently.

- **`let`** binds a value or function; everything is an expression.
- **Immutability by default.** Values do not change unless you opt into a `ref` or a mutable record field. This is why the type system can reason so much.
- **Type inference.** You rarely write types; the compiler infers them and rejects mismatches. "If it compiles, a large class of bugs is already gone" is the culture line.
- **Variants (algebraic data types)** are "one of these" types, and **pattern matching** forces you to handle every case, which is where a lot of the safety comes from.
- **`option`** is OCaml's answer to null: a value is `Some x` or `None`, and the compiler makes you handle `None`, so there is no null-pointer surprise.
- **Records** are named-field structs; **modules** group types and functions and are how large code is organised (with functors as modules parameterised by modules, which you can just name, not explain deeply).
- **Async / monadic style.** Concurrency uses a library (Async) with a bind operator rather than threads everywhere; you only need to know it exists and that it is cooperative.

<details>
<summary><b>A snippet you should be able to read out loud.</b></summary>

```ocaml
(* a variant: a log event is exactly one of these *)
type event =
  | Login   of { user : string; ok : bool }
  | Logout  of { user : string }
  | Unknown of string

(* pattern matching must cover every case; option handles "not found" *)
let describe (e : event) : string =
  match e with
  | Login { user; ok = true }  -> user ^ " logged in"
  | Login { user; ok = false } -> user ^ " failed to log in"
  | Logout { user }            -> user ^ " logged out"
  | Unknown line               -> "unparsed: " ^ line
```

What to say about it: it is a variant with four cases, the `match` is exhaustive (the compiler errors if you forget one), the records have named fields, and there is not a null anywhere. That is the whole pitch of the language in ten lines: make the illegal states impossible to represent, and make the compiler check you handled every real one.
</details>

---

## The ecosystem names

Recognise these so a dropped name does not throw you. You do not need to have used any.

| Name | What it is |
| --- | --- |
| Base, Core | Jane Street's alternative and extended standard libraries (they found the default stdlib too small) |
| Async | Their cooperative concurrency library |
| Dune | The build system |
| OxCaml | Jane Street's OCaml extensions, including the modes that buy performance |
| Bonsai | Their web-UI framework (OCaml compiled to JavaScript via js_of_ocaml) |
| Hardcaml | Designing FPGA hardware in OCaml |
| Iron | Their code-review system (reviews ordered by dependency) |
| magic-trace, Memtrace | Performance and memory tracing tools they open-sourced |

*Real World OCaml* is the standard book, co-authored by a Jane Street engineer, and it uses Base/Core/Async; naming it is a fine way to say how you would ramp up.

---

## The security framing (your angle)

OCaml is not just a trading-code choice; it has a security story you can speak to, which connects to your memory-safety points elsewhere:

- Memory-unsafe C turns small bugs into exploitable undefined behaviour; a garbage-collected, type-safe language removes that whole class. Say you treat it as removing the bug class, not the instance ([block 06](../06-linux/README.md) buffer-overflow answer).
- OCaml has been used to build security-sensitive infrastructure: DNS and SSH implementations and a post-Heartbleed TLS stack (the MirageOS work), and a unikernel "Bitcoin piñata" that stood up to a very large number of attacks as a public demonstration ([Signals and Threads ep. 12](https://signalsandthreads.com/what-is-an-operating-system/)).
- The AI tie-in: Jane Street trains its own models partly because public models are weaker in OCaml than in mainstream languages, and because the compiler surfaces design mistakes that would be runtime errors elsewhere ([ep. 31](https://signalsandthreads.com/the-changing-craft-of-programming/)). You can use this in the AI conversation ([block 16](../16-ai-and-llm-security/README.md)): verifying AI-written OCaml is easier precisely because the type system rejects a lot of wrong code before a human ever reviews it.

---

## The honest close

Say it plainly, because "know what you don't know" is a scored principle. If you are interviewing soon: you have never shipped OCaml, you have read enough to respect why they use it, and their own pages say most engineers arrived the same way. If you have done month three of the [six-month roadmap](../24-six-month-roadmap/README.md): you have written a few hundred lines, here is the repo, here is what surprised you, and you know how far that is from fluency. Either way, their pages say most engineers learned it in the first eighteen months ([Departments](https://www.janestreet.com/join-jane-street/departments/), [Programming](https://janestreet.com/programming)). That answer is stronger than a shaky attempt to sound fluent. Do not fake depth here; faking it breaks the one principle they test hardest.

[← Trading domain](../17-trading-domain/README.md) · [Index](../README.md) · [Next: Mock interviews →](../19-mock-interviews/README.md)
