
# WELCOME

Hey everyone! Welcome to the first ever TokioConf. Yes! I am also so excited that we're actually doing this. I've been wanting a conference like this for years — a place for practitioners building network apps with Rust to come together and go deep. 19% of all crates on crates.io depend on Tokio. I think it's about time we had our own conference. And really, the scope isn't just Tokio — it's async Rust networking as a whole.

But, before I get into anything else — I have to thank Tiffanie, who did all the hard work to make this happen. All the logistics, and there was... a lot. So, thank you for taking all that on.

So, I was thinking about what to actually talk about up here and went digging through old blog posts for inspiration, including the original blog post I wrote announcing Tokio, and I noticed something.

# Tokio turns 10

Tokio is turning 10 this year... it was first announced in August 2016!  That is crazy! When I wrote that blog post, in no way did I imagine it would end up being used by some of the biggest web services in the world. And I definitely didn't imagine that I would still be working on it 10 years later... thats a long time. Back then, I was just trying to work on a hobby project, I think I was playing around with a distributed database (they were all the rage back then), but there was no async networking library for Rust. So, I decided to shave that yak.

And what Tokio looked like back then... very different. This is well before async/await in Rust is a thing. Who here used Tokio before async await? ... it was a tough time. Before async/await, we had to write all of our state machines by hand. I considered including an example of it looked like, but I don't think it would fit on a slide.

# Pre-async/await

But, we were still building with it. In 2017, I joined Buoyant. Their main product is LinkerD, a service mesh, which you can think of as a fancy proxy. And at the time I joined, they were rewriting it from Scala to Rust. Now, this is a pretty complex piece of software and approaching a project of that scope with pre-async/await Tokio was... a challenge. So we tried to manage it with abstraction. I worked on Tower, which uses traits to layer a request/response abstraction on top of Tokio. The goal being to compartmentalize all that state machine code. It helped, but at the cost of trait insanity. Whether it was actually an improvement is debatable.

But honestly, this wasn't just a Tower problem. Rust and the ecosystem was still young. We were all figuring out how to design APIs. We were also drawn to the power of the type system. A lot of the buzz around Rust back then was zer-cost abstrations, and misuse-resistant APIs, which are both great things... when used the right way. I think a lot of us tried to work that in whenever we could, at the expense of bing user friendly. 

Most developers using Rust back then were still power users and early adopters, able to figure out the more complicated compiler errors... well most of the time at least. And that was kind of the problem. Rust worked. It was fast, reliable. But it was a tough sell beyond the early adopters.

The productivity story wasn't there yet, and without fixing that, I wasn't sure how much further we'd go. Because, fundamentally, technology adoption is about productivity: how fast can I use a programming language, a library, a tool, to accomplish my goal.

# Async/await

And then async/await happened... a total game changer. It gives us a friendly programming interface for writing async code — and it does it without adding any runtime cost compared to writing state machines by hand. Some would say it's actually better than doing it by hand, because you can borrow data across await points, which would be very hard to do safely without the construct. This is the good kind of zero-cost abstraction — the kind that's actually nice to use whether it's zero-cost or not.

We take it for granted now. Obviously Rust has async/await. But I remember listening in on conversations the Rust team was having, trying to figure out how to do it, and it was not at all obvious, at the time, that it was possible. I know I didn't think it was — or at least I had no idea how. So, big props to the team that made it happen, including Aaron Turon, Taylor Cramer, and withoutboats.

And look, yes it isn't perfect. Yes Pin is hard to use... but you don't have to touch it most of the time. Yes being able to drop async blocks at any point of the execution an lead to confusing bugs... hindsight is 20/20 as they say. Overall, the async syntax we have today is so much better than what we were doing before. At least we didn't end up with prefix await syntax, which is what I was aruging for in those pages and pages of RFC omments. Glad nobody listened to me.

Show of hands: who here used Tokio before async / await? ... would you go back?

# Where we are today

No, and I think that is true across the industry now. I keep an eye out whenever a new open source infrastructure level project is announced, and the majority of them are build in Rust. I mean, we have people from some of the biggest companies in the world, working on the biggest services in the world. Now, I know that I am probably biased, but from where I am sitting, Rust has become the default for greenfield infrastructure where performance matters. And by all public metrics, Rust is still growing.

# Why stop here

But here's the thing — performance, reliability, fewer bugs — those aren't infrastructure-specific benefits. Those are just... good things. For any software. So, not to be a Rust maximalist, but why isn't more software being written with Rust?

At the end of the day, it comes down to productivity. Pragmatic developers pick whatever gets them to their goal fastest. When the requirement is efficient and reliable, Rust gets you there quickest. But, lets be honest. Not all software has efficient and reliable as a requirement. Sure, those are nice to have, and all things equal, of course you will pick efficient and reliable, but not at the cost slower development.

# Rust can be productive

But here's what I've noticed. Teams that adopt Rust for infrastructure often start reaching for it for higher-level stuff too. Web apps, internal tools, things where performance isn't the main concern. Why? Because once you know Rust, you are past one of Rust's biggest productivity headwinds: getting started. You already know the language, the tooling, your team knows it, and you can share libraries across projects. One language across the stack is a real productivity win.

Yes, the compile times are real. And yes, Rust can be a complex language. But the complexity? Those are power tools — lifetimes, advanced trait bounds, unsafe. You don't need them to build a web app. The real gap isn't the language — it's the ecosystem.

Rust has a vibrant ecosystem, but it's historically focused on infrastructure. At the higher level, there just aren't as many libraries. That gap matters, because a full-featured, easy-to-use library ecosystem is a huge part, maybe the biggest part, of what makes a language productive for a given use case.

# AI

Ok, so now is the part where I need to put on my flame-retardant suit. I have to talk about AI. Look, whatever your feelings about AI, it is fundamentally changing how software gets written. That's just happening. Yes, there are ethical questions, but the cat is out of the bag, and whether we like it or not, it has implications for Rust.

I said earlier that Rust's biggest headwind is getting started. Well, AI makes learning new things so much easier. I've seen devs that have never used Rust before drop in and be productive right away. No more three month learning curve. And no, I don't mean vibe coding — I mean using AI as a learning partner while doing real work. Developers have been hearing for years that Rust is fast, and reliable, and safe — but the learning curve kept them out. That barrier is gone.

And here is the thing. It isn't just that AI helps you write Rust. Rust helps AI generate code. We've all heard the term 'AI slop' — the AI goes off track and generates junk. Well, what keeps it on track? Guardrails. And Rust is full of them. The type system, the borrow checker, the culture of misuse-resistant APIs — the same things that help us write better code also help AI write better code. I think that makes Rust the best target language for AI tools out there right now.

---

And here is the thing. It isn't just that AI helps you write Rust. Rust is the best target language for these AI tools. I've been calling this the 'slop radius' — how far off the rails can AI-generated code go before something catches it? With Rust, the compiler keeps it tight. Type errors, memory bugs — caught. But libraries can shrink it even further. A well-designed library with strong conventions and guardrails? The AI can't do the wrong thing if the API doesn't let it.

---

And here is the thing. It isn't just that AI helps you write Rust. Rust is the best target language for these AI tools. Rust's strong type system and the culture of misuse resistant APIs, the same things that helps us, as humans, write better code, also helps AI generate better code.

But the compiler only gets you so far. It catches type errors, memory bugs. It doesn't catch bad design or logic bugs. That's where libraries come in.


A well-designed, Rust library leverages the typesystem to make the pit of success as big as possible. This is generally good practice for humans, but it is also good for AI. Libraries with strong conventions, guard rails, and misuse resistant APIs limits how far off the rails AI can go, or what I started calling the slop radius.



  "But the compiler only gets you so far. It catches type errors, memory bugs. It doesn't
  catch bad design or logic bugs. That's where libraries come in. I've been calling this
  the 'slop radius' — how far off the rails can AI-generated code go before something
  catches it? The compiler shrinks it. But a well-designed library with strong conventions
   and guardrails? That shrinks it way further. The AI can't do the wrong thing if the API
   doesn't let it."
