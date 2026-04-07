# TokioConf 2026 Opening Keynote — Summary

**Speaker:** Carl Lerche (Tokio creator)
**Duration:** ~25-30 minutes (first half of 60-min shared slot with Alice)

---

## Part 1: Retrospective — Tokio at 10

### Welcome
- First ever TokioConf — a conference for async Rust networking practitioners
- 19% of all crates on crates.io depend on Tokio
- Thank organizer (Tiffanie)

### Tokio turns 10
- Announced August 2016 — started as a yak shave from a hobby distributed database project
- No async networking library existed for Rust at the time

### Technical history (with code examples)
- **Eventual (2015):** First attempt at futures — copied callback pattern from other languages. Required allocation + callback at every step. Didn't work well in Rust.
- **Rust Futures trait:** Rust team's insight — poll-based futures that maintain low-level I/O efficiency. Ingenious design, but required implementing both `poll` and `schedule` (code duplication).
- **Tokio's contribution:** Realized polling an I/O resource is an implicit intent of interest — eliminated the need for a separate `schedule` method. This pattern became the foundation of Rust async I/O.
- Pre-async/await, using Tokio meant writing all state machines by hand.

### The dark middle
- 2017: Joined Buoyant, worked on Linkerd (service mesh rewrite from Scala to Rust)
- Built Tower — traits layered on top of Tokio to manage complexity. Helped, but at the cost of "trait insanity."
- Broader issue: Rust community was young, over-indexing on zero-cost abstractions and misuse-resistant APIs at the expense of usability.
- Rust worked, but the productivity story wasn't there. Technology adoption is fundamentally about productivity.

### Async/await — the turning point
- Game changer: friendly async interface with zero runtime cost
- Credit to Aaron Turon, Taylor Cramer, withoutboats — no prior art, wasn't clear it was even possible
- Honest: Pin is hard, async cancellation is a footgun, but overall massively better
- Audience interaction: "Who used Tokio before async/await? ... want to go back?"
- Async/await is the killer feature that made Rust productive for server applications

## Part 2: Vision — Where Rust goes next

### Rust won infrastructure
- Majority of new open-source infrastructure projects are in Rust
- Rust is the default for greenfield infrastructure where performance matters
- Still growing by all public metrics

### Why isn't Rust used more broadly?
- Performance, reliability, fewer bugs are good for ALL software, not just infrastructure
- But at the app level, productivity > performance. Developers pick the fastest path to their goal.
- Common belief that Rust = slower development. **This isn't true** — teams that adopt Rust for infra start reaching for it more broadly.
- Biggest headwind is getting started. Once past that, Rust is productive — one language across the stack is a real win.
- Compile times are real. Complexity features (lifetimes, trait bounds, unsafe) are power tools you don't need for app-level code.
- The real issue is the ecosystem — not enough higher-level libraries, and existing ones often prioritize performance over usability.

### AI changes the equation
- AI is fundamentally changing how software gets written — implications for Rust whether we like it or not
- **Barrier to entry collapsing:** AI makes learning Rust dramatically easier. Not vibe coding — AI as a learning partner.
- **Rust is the best target language for AI:** Type system, borrow checker, misuse-resistant APIs act as guardrails. AI writing Rust gets checked by the compiler.
- **Slop radius:** Libraries shrink how far off track AI-generated code can go. Good API design is now guardrails for AI, not just humans.

### The bold prediction
- If productivity is the same (possibly better), why choose the slower, less reliable option?
- **Rust has the opportunity to become a top-three language for greenfield development in the next 10 years.**

## Part 3: What we need to build

### The path forward
- Rust must become synonymous with productivity
- Need: rich library ecosystem + easy APIs
- Applies to all verticals (data pipelines, device apps, game dev, DevOps) — focusing on server applications here

### API design philosophy
- "Premature optimization is the root of all evil" — applies to API design too
- Concrete examples: traits when an enum would do, lifetime parameters to avoid a clone, flexibility nobody asked for
- Result: APIs that are powerful but painful. Need to prioritize productivity — easy to use first, optimize later.

### Toasty — an ORM for Rust
- Biggest missing piece for higher-level server apps is the database layer — most server apps are read/logic/write against a database
- Toasty: ORM focused on productivity. Code example shows simple model definition, no lifetimes, minimal boilerplate.
- Anecdote: initially added a lifetime to avoid a copy, caught myself, removed it. Staying disciplined about simplicity.

### Don't just recreate — go further
- Lesson learned across Eventual → Tokio: copying patterns from other languages doesn't work in Rust. Leaning into Rust's strengths produces something better.
- Same applies to Toasty: rethinking ORMs from the ground up.

### Why ORMs need rethinking
- ORMs have a real problem: the abstraction is leaky. Works for simple cases, hits a wall on anything complex.
- People say ORMs are "too magical" — but SQL engines are far more complex and nobody complains. The difference is SQL engines are smart enough. ORMs aren't yet.
- The answer isn't less magic — it's smarter magic.
- Application data is graph-based; databases are relational. Toasty understands both sides and lets you move between them seamlessly.
- Requires a full application-level query engine — unprecedented in an ORM.
- Tried this in Ruby (too slow) and Java (GC pressure). Rust is the only language where this is viable.
- **This is what "Rust enables more" means:** not doing the same things faster, but building things that aren't possible in other languages.
- Rust can blend low-level work (query engines, layout engines, GPU) and high-level work (ORMs, web frameworks) in one language. That's a superpower.

### Toasty status + what's next
- Honest: Toasty is still early, massive project, lots of work ahead
- First piece — more coming by end of year

### Why this matters to infrastructure developers
- Rust adoption starts at the infrastructure level and grows outward
- Control planes, admin dashboards, internal tools — things that don't need to be fast, just need to get built
- If the ecosystem is there, you can say "just use Rust" — and they get fast and reliable for free

## Close
- Hope for TokioConf: not just how to make async Rust faster, but how to make Rust easier, productive, the obvious choice beyond infrastructure
- If we do, Rust becomes the default for greenfield development in general
- Handoff to Alice for what's next for Tokio
