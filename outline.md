# TokioConf 2026 Keynote — Outline Draft

**Speaker:** Carl Lerche
**Slot:** ~25-30 minutes (first half of 60-min shared keynote with Alice)
**Date:** August 2026

---

## 1. Welcome & Why TokioConf (~2 min)

- Welcome everyone, express genuine excitement — this is the first TokioConf ever
- TokioConf is a space for **practitioners** of async Rust networking to come together
  - Not just a Tokio conference — it's the async Rust networking community
  - Topics here go deeper than a general Rust conference allows
- Why does this matter? We're still figuring things out. The hallway conversations,
  the shared war stories — that's where the real progress happens
  - _[FILL IN: personal anecdote about a conference conversation that changed your
    thinking? or keep it general?]_

## 2. Tokio Turns 10 / The Origin Story (~3 min)

- "It so happens that Tokio turns 10 this year. I announced it in August 2016."
- The yak shave story:
  - Got hooked on Rust, started a hobby distributed database project
  - Tokio was the yak shave — "one I'm still digging myself out of"
  - _[This should get a laugh — lean into it]_
- Already knew Rust was the best language for infrastructure code
- Had written mio, but mio was just a binding to epoll — not an abstraction
- Had used Java/Netty, Node.js, Ruby, Clojure, Go — wanted something at that
  level of usability for Rust
- Good timing: the rust-lang team was exploring Futures at the same time
  - _[FILL IN: any specific detail about that early collaboration worth mentioning?]_

## 3. The Dark Middle (~2 min)

- First version of Tokio: all futures written by hand, manually
  - Very tedious. Honestly not that much better than using mio directly
- Tower: an attempt to recreate Finagle-like abstractions in Rust
  - Ended up as trait-hell — a cautionary tale (come back to this later)
- The broader context: Rust as a community was still young
  - We were all still learning how to design Rust libraries
  - The patterns and conventions hadn't crystallized yet
  - _[FILL IN: any specific memory from this era that captures the frustration?
    A GitHub issue, a conversation, a moment where you thought "this might not work"?]_

## 4. The Turning Point: async/await (~3 min)

- This is the moment everything changed
- Credit the designers — _[FILL IN: name names here, they deserve it]_
  - No prior art for ergonomic async/await without a runtime
  - It wasn't even clear this was possible
- **Audience moment:** "Raise your hand if you wrote async Rust before async/await"
  - Acknowledge them — they know the pain
- The before/after:
  - Before: writing futures by hand, combinators everywhere, unreadable code
  - After: code that looks like normal Rust, just with `.await`
  - This is when adoption started accelerating
  - _[FILL IN: is there a specific "wow" moment you remember when you first used
    async/await? The moment it clicked?]_

## 5. The Payoff (~2 min)

- Where we are today — _[FILL IN: numbers, adoption stats, notable users]_
- Make the claim: async Rust has **won** infrastructure
  - When performance matters and it's a greenfield project, there is no other
    reasonable option
- But be honest: not everything is perfect
  - Async cancellation — we made it almost too easy. Just drop. But your code
    might not run to completion, and that leads to unexpected behavior
  - A historically hard problem; we're still working on it
  - _[One or two sentences, don't dwell — name it, own it, move on]_

## 6. Why Stop at Infrastructure? (~5-6 min)

**This is the core of the talk. Take your time here.**

- Rust's benefits — performance, reliability, productivity — apply to **all** code,
  not just infrastructure
- But Rust hasn't won the higher-level app space. Why?
  - At the app level, speed is "nice to have," not required
  - Developer productivity is the most important thing
  - There's a chicken-and-egg problem: no ecosystem because no users, no users
    because no ecosystem

- **The "why should you care" argument** — speaking directly to the room:
  - You're infrastructure developers. Rust is already in your org.
  - Once an org adopts Rust, there's a natural pull to use it more broadly
  - Fewer languages = more productivity. Less context switching, shared tooling,
    shared internal libraries across projects
  - When you need to build the higher-level app on top of your infrastructure,
    consider reaching for Rust

- **The reframe:**
  - "If the barrier to entry is low, and productivity is the same — why would you
    choose the slower, less reliable option?"
  - Let that land.

- **The bold prediction:**
  - "I see a path where, in 10 years, most new software written from scratch is
    in Rust"
  - _[FILL IN: how much do you want to qualify this? "I know that sounds crazy,
    but..." or just let it stand?]_

- What's missing to get there:
  - Ecosystem at the app level — libraries that prioritize productivity
  - Build times — the code-compile-run cycle is a real problem. JS is killer here.
    This needs to get better
    - _[FILL IN: do you want to name specific efforts? Cranelift? Or leave it as
      an open challenge?]_
  - Simple, easy-to-use APIs (next section)

## 7. How We Get There (~4-5 min)

### Simple APIs — a design philosophy

- Bring back the Tower cautionary tale:
  - Traits have significant cognitive cost, especially as bounds get complicated
  - Before reaching for generics, consider if an enum would work
  - Don't reach for lifetimes too early — cloning and moving aren't sins
  - "Premature optimization is the root of all evil" — and that applies to API
    design too
  - _[FILL IN: can you give a one-sentence concrete example? "Maybe you can avoid
    that generic in favor of an enum" — is there a real case you can point to?]_

- This isn't just about app-level code. Even at the infrastructure level, we
  don't need every power feature all the time

### Toasty and what's next

- Toasty: a high-level ORM for Rust, focused on productivity
  - Just released — _[FILL IN: one line on what makes it different / what it
    optimizes for]_
  - "This is the first piece. There's more coming by end of year."
  - _[Let people connect the dots — don't say "Rails for Rust"]_

### Macros

- Rust's superpower for productivity — macros can hide complexity and provide
  ergonomic APIs
- But sharp edges remain: compiler errors with macros are rough
  - Better macro error messages would be a big win for the ecosystem

### AI as tailwind

- Rust is at a pivotal moment, and AI is one of the tailwinds
- Three beats:
  1. **Barrier to entry is collapsing** — the hardest thing about Rust was learning
     it. AI is solving that. The barrier to starting with Rust is effectively gone
  2. **Rust is the ideal target language for AI code generation** — the type system,
     the borrow checker, the compiler guarantees all serve as guardrails that keep
     AI-generated code on track and reduce bugs
  3. **The "slop radius"** — strong library abstractions constrain how far off the
     rails AI-generated code can go. Good API design isn't just for humans anymore —
     it's the guardrails for AI too. This makes the case for simple, well-typed
     APIs even stronger
- _[NOTE: AI also partially mitigates the build time problem — the compile-wait
  cycle hurts less when AI tools are also taking a few seconds to think. Consider
  whether to include this or if it undercuts the "build times need to get better"
  point]_

## 8. Close / Handoff to Alice (~1-2 min)

- _[FILL IN: How do you want to end?]_
- Options to consider:
  - **Optimism + energy:** "The next 10 years are going to be bigger than the
    first 10"
  - **Challenge to the room:** "We have the language. We have the runtime. Now
    we need the ecosystem. Let's build it."
  - **Gratitude + handoff:** Thank the community, then "Alice is going to tell
    you what we're building to make this real"
  - Some combination of the above
- Hand off to Alice

---

## Open items to resolve

- [ ] Specific numbers/adoption stats for section 5
- [ ] Names to credit for async/await design (section 4)
- [ ] Whether to name specific build-time efforts or leave as open challenge
- [ ] The close — what feeling to leave the room with
- [ ] Whether the AI/build-time mitigation point helps or undercuts
- [ ] Any personal anecdotes to fill in marked spots
- [ ] How much to qualify the bold prediction
- [ ] Coordinate with Alice on handoff and avoiding overlap
