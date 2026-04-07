
# WELCOME

Hey everyone! Welcome to the first ever TokioConf. Yes! I am also so excited that we're actually doing this. I've been wanting a conference like this for years — a place for practitioners building server applications with Rust to come together and go deep. Did you know, 19% of all crates on crates.io depend on Tokio. I think it's about time we had our own conference. And by that, I really mean async Rust networking as a whole, not just Tokio.

But, before I get into anything else — I have to thank Tiffanie, who did all the hard work to make this happen. All the logistics, and there was... a lot. So, thank you for taking all that on.

So, I was thinking about what to actually talk about up here and went digging through old blog posts for inspiration, including the original blog post I wrote announcing Tokio, and I noticed something.

# Tokio turns 10

Tokio is turning 10 this year... it was first announced in August 2016!  That is crazy! When I wrote that blog post, in no way did I imagine it would end up being used by some of the biggest web services in the world. And I definitely didn't imagine that I would still be working on it 10 years later... that's a long time. Back then, I was just trying to work on a hobby project, I think I was playing around with a distributed database (they were all the rage back then), but there was no async networking library for Rust. So, I decided to shave that yak.

# Let's reminisce

Ok, so it is the first TokioConf. I hope you will indulge some reminiscing. It is 2015. Mio is just released. The very first RustCamp just happened (it wasn't even called RustConf yet). Javascript promises are hot, but it doesn't have async/await syntax yet. I start playing around with futures on top of Mio:

# Eventual

```rust
let (client, _) = tcp::connect(&"216.58.216.164:80".parse().unwrap()).unwrap();
let (dst_tx, dst_rx) = r.stream(client);

let a = dst_tx.send_all(src_rx).map_err(|_| ());
let b = src_tx.send_all(dst_rx).map_err(|_| ());

eventual::join((a, b)).and_then(|v| {
    println!(" + Socket done");
    Ok(v)
})
```

You will have to bear with me a bit... this was 11 years ago. I hardly remember what code I wrote 1 year ago does. But this is basically what my first attempt looked like. Note, I wasn't the only one exploring futures in Rust, and really all attempts more or less looked the same. The most important thing to note is, this implementation, and every other at the time, required an allocation and a callback at every single step. And backpressure was entirely your problem.

Needless to say, I'm glad this isn't what we ended up with. Because around the same time, some folks on the Rust team were also exploring futures — and they had a brilliant insight. They landed on a trait that was able to model futures while also maintaining the poll-based pattern that makes low-level I/O so efficient.

# Rust futures

```rust
pub trait Future: Send + 'static {
    type Item: Send + 'static;
    type Error: Send + 'static;
    fn poll(&mut self, task: &mut Task) -> Poll<Self::Item, Self::Error>;
    fn schedule(&mut self, task: &mut Task);
}
```

This is the earliest version of the trait I could find. Now, remember, up until now, nobody had yet designed a future that did not require a channel and callback at each step. The trait seems obvious now, but it was pretty ingenious. You can already start seeing the shape that is the Future trait in Rust today. The biggest difference is the `schedule` method, which handles what wakers do today. When `poll` returns that it is not ready, the runtime would call schedule with the task handle, and the future would then be responsible for notifying the task when the future should be called again. So, to implement Future, you would implement poll similarly to what you do today, but you would also have to implement schedule to pass that task handle call through the stack a second time. In practice, this meant you had to duplicate your code, once for poll and once for schedule.

# Tokio

And this gets us to the original Tokio announcement in August 2016. When I was working on Tokio, I tried using the future trait and noticed that pattern, that `poll` methods and `schedule` methods ended up being implemented to be identical. This is the solution I ended up coming up with, and I think is probably the one bit of invention I actually provided to Rust's async story, so I'm going to revel in it for a bit.

```rust
pub fn read(&self, buf: &mut [u8]) -> io::Result<usize> {
    if !self.source.is_readable() {
        return Err(would_block());
    }

    match (&self.mio).read(buf) {
        Ok(n) => {
            self.source.advance();
            Ok(n)
        }
        Err(e) => {
            if e.kind() == io::ErrorKind::WouldBlock {
                self.source.unset_readable();
            } else {
                self.source.advance();
            }

            Err(e)
        }
    }
}
```

This is the read function for the TCP socket type in the very first version of Tokio. That is it, I dug up the code and copy/pasted it. The signature was exactly the same as a blocking read. What I realized was that polling an I/O resource was an implicit intent of interest. What that means is, if you try to call read, the socket isn't ready, and it returns io-would-block, that you could implicitly register the task with the reactor to be scheduled as soon as the socket becomes ready again. There was no need for a second `schedule` method at all. I also ended up stashing the task variable from the original future trait in a thread-local so you didn't have to pass it through the call stack. Once I did that, I noticed you didn't even need a different signature and could repurpose the existing IO traits, so this was the very first Tokio socket read. Granted, nobody else thought that stashing the task handle in a thread-local and reusing blocking IO traits was a good idea, and yeah, they are probably right, so that part didn't stick, BUT, the pattern of using an async resource implies that you want the task to be scheduled when the IO resource becomes ready did stick and is the foundation of Rust async IO now.

So yeah, that was Tokio back then... very different. Again, well before async/await in Rust. To actually use Tokio at this point required writing all futures by hand, which meant maintaining many state machines by hand, writing many many enums. I considered including a real-world example of using Tokio, but it didn't fit on a slide.

# Pre-async/await

But, we were still building with it. In 2017, I joined Buoyant. Their main product is LinkerD, a service mesh, which you can think of as a fancy proxy. And at the time I joined, they were rewriting it from Scala to Rust. Now, this is a pretty complex piece of software and approaching a project of that scope with pre-async/await Tokio was... a challenge. So we tried to manage it with abstraction. I worked on Tower, which uses traits to layer a request/response abstraction on top of Tokio. The goal being to compartmentalize all that state machine code. It helped, but at the cost of trait insanity. Whether it was actually an improvement is debatable.

But honestly, this wasn't just a Tower problem. Rust and the ecosystem was still young. We were all figuring out how to design APIs. We were also drawn to the power of the type system. A lot of the buzz around Rust back then was zero-cost abstractions, and misuse-resistant APIs, which are both great things... when used the right way. I think a lot of us tried to work that in whenever we could, at the expense of being user friendly. 

Most developers using Rust back then were still power users and early adopters, able to figure out the more complicated compiler errors... well most of the time at least. And that was kind of the problem. Rust worked. It was fast, reliable. But it was a tough sell beyond the early adopters.

The productivity story wasn't there yet, and without fixing that, I wasn't sure how much further we'd go. Because, fundamentally, technology adoption is about productivity: how fast can I use a programming language, a library, a tool, to accomplish my goal.

# Async/await

And then async/await happened... a total game changer. It gives us a friendly programming interface for writing async code — and it does it without adding any runtime cost compared to writing state machines by hand. Some would say it's actually better than doing it by hand, because you can borrow data across await points, which would be very hard to do safely without the construct. This is the good kind of zero-cost abstraction — the kind that's actually nice to use whether it's zero-cost or not.

We take it for granted now. Obviously Rust has async/await. But I remember listening in on conversations the Rust team was having, trying to figure out how to do it, and it was not at all obvious, at the time, that it was possible. I know I didn't think it was — or at least I had no idea how. So, big props to the team that made it happen, including Aaron Turon, Taylor Cramer, and withoutboats.

And look, yes it isn't perfect. Yes Pin is hard to use... but you don't have to touch it most of the time. Yes being able to drop async blocks at any point of the execution can lead to confusing bugs... hindsight is 20/20 as they say. Overall, the async syntax we have today is so much better than what we were doing before. At least we didn't end up with prefix await syntax, which is what I was arguing for in those pages and pages of RFC comments. Glad nobody listened to me.

Show of hands: who here used Tokio before async/await? ... now keep your hands up if you want to go back to pre-async/await. ... Exactly. Async/await is the killer feature that made using Rust for writing server applications actually productive.

# Where we are today

And look at what's happened since. I keep an eye out whenever a new open source infrastructure project is announced, and the majority of them are built in Rust. We have people from some of the biggest companies in the world, working on the biggest services in the world. Now, I know that I am probably biased, but from where I am sitting, Rust has become the default for greenfield infrastructure where performance matters. And by all public metrics, Rust is still growing.

# Why stop here

But here's the thing — performance, reliability, fewer bugs — those aren't infrastructure-specific benefits. Those are just... good things. For any software. So, not to be a Rust maximalist, but why isn't more software being written with Rust?

At the end of the day, it comes down to productivity. Pragmatic developers pick whatever gets them to their goal fastest. When the requirement is efficient and reliable, Rust gets you there quickest. But, let's be honest. Not all software has efficient and reliable as a requirement. Sure, those are nice to have, and all things equal, of course you will pick efficient and reliable, but not at the cost of slower development.

# Rust can be productive

And the common belief is that Rust means slower development. But I don't think that's true. Here's what I've noticed. Teams that adopt Rust for infrastructure often start reaching for it for higher-level stuff too. Web apps, internal tools, things where performance isn't the main concern. Why? Because once you know Rust, you are past one of Rust's biggest productivity headwinds: getting started. You already know the language, the tooling, your team knows it, and you can share libraries across projects. One language across the stack is a real productivity win.

Yes, the compile times are real. And yes, Rust can be a complex language. But the complexity? Those are power tools — lifetimes, advanced trait bounds, unsafe. You don't need them to build a web app. The real gap isn't the language — it's the ecosystem.

Rust has a vibrant ecosystem, but it's historically focused on infrastructure. At the higher level, there just aren't as many libraries. That gap matters, because a full-featured, easy-to-use library ecosystem is a huge part, maybe the biggest part, of what makes a language productive for a given use case.

# AI

Ok, so now is the part where I need to put on my flame-retardant suit. I have to talk about AI. Look, whatever your feelings about AI, it is fundamentally changing how software gets written. That's just happening. Yes, there are ethical questions, but the cat is out of the bag, and whether we like it or not, it has implications for Rust.

I said earlier that Rust's biggest headwind is getting started. Well, AI makes learning new things so much easier. I've seen devs that have never used Rust before drop in and be productive right away. No more three month learning curve. And no, I don't mean vibe coding — I mean using AI as a learning partner while doing real work. Developers have been hearing for years that Rust is fast, and reliable, and safe — but the learning curve kept them out. That barrier is gone.

And here is the thing. It isn't just that AI helps you write Rust. Rust helps AI generate code. We've all heard the term 'AI slop' — the AI goes off track and generates junk. Well, what keeps it on track? Guardrails. And Rust is full of them. The type system, the borrow checker, the culture of misuse-resistant APIs — the same things that help us write better code also help AI write better code. I think that makes Rust the best target language for AI tools out there right now.

But the compiler can only do so much on its own. That's where libraries come in. The more functionality you can push into the library, the less code you or the AI has to write from scratch — and that's less surface area for things to go wrong. A well-designed library with strong conventions, leveraging Rust's type system to provide misuse-resistant APIs, shrinks the slop radius even further.

# Slide

So think about where that leaves us. The learning curve — the biggest thing holding Rust back — is disappearing. And Rust's own strengths make it the best language for AI. So if productivity is the same, possibly better, why would you not pick the language
that gives you the most efficient and reliable end product.

I don't think there is a good reason. In fact, I believe that Rust has the opportunity to become one of the top three languages for greenfield development in general over the next 10 years.

# What we need to build

But it won't just happen by itself. It will only happen if Rust becomes synonymous with productivity. That means a rich library ecosystem that handles the plumbing for you. And it means easy APIs that get out of your way — that remove the boilerplate and let you build features instead of getting bogged down in traits and lifetimes. That's what gets us there. And by the way, this isn't just about server applications. The same argument applies to data pipelines, device apps, game dev, DevOps tooling — every vertical. But server applications is what this room builds, so let's talk about what that looks like.

# Toasty

This is why I've been working on Toasty. Toasty is an ORM for Rust. Now, I know what some of you are thinking. "I don't like ORMs. I'd rather just write SQL. ORMs are too magical" And I get it. But think about that for a second, "magical"... is it really about "magic". SQL databases have full query engines — query planners, optimizers, tons of complexity under the hood. And we're all fine with that. The problem with ORMs isn't that they're magical. It's that the abstraction is leaky. They work great for the conventional cases, but the second you go across the grain, you hit a wall. And there's no good escape hatch. So you throw it all out and go back to raw SQL. But that's throwing the baby out with the bathwater.

# Application data is graph-based

When I started Toasty, I wanted to fix that. The fundamental challenge is that application data is graph-based — a user has many todos, a todo belongs to a category — but SQL is relational. ORMs have been trying to bridge that gap for 20 years, but it's bigger than people think. So I decided to take a different approach. Toasty is built to understand both sides — how your application thinks about data and how your database stores it — and let you move between them seamlessly, even within the same query. That's the escape hatch. You never hit the ORM wall because you can always drop down to SQL, right in the middle of a query, and Toasty handles it.

Now, to actually pull this off, Toasty needs something that, as far as I know, no ORM has tried before — a full application-level query engine. One that can actually understand, validate, plan, and translate queries across both levels. And this is where it comes full circle to Rust. Years ago, I actually worked on Ruby on Rails itself, including on ActiveRecord — and, back then, I wanted to do something similar for ActiveRecord. But, Ruby was way too slow, so it wasn't really an option. Rust really is the only language where putting a query engine into an ORM makes sense because it is both expressive enough for high-level abstractions and performant enough to do the work.

And this is what excites me most about Rust. It's a language where you can build a query engine and an ORM in the same codebase. Think about what that means more broadly. Rust is a language where you can do work that's traditionally considered low-level — font rendering, layout engines, game engines, GPU programming — and high-level work like ORMs, web frameworks, developer tooling — in the same language. The ability to blend both into one is a superpower.

So when we think about building that library ecosystem, we shouldn't just be recreating what exists in other languages. We should be asking: what new features and abstractions does Rust make possible? That's how we get there — not by catching up, but by building something better.

Now, I want to be honest. Toasty is still early. This is a massive project and there's a lot of work ahead. But I'm optimistic that the direction is sound. And it's the first piece — there's more coming by end of year.

# Why this matters to you

So why am I talking about higher-level libraries to a room full of infrastructure developers? Well, because Rust adoption starts with you. Rust gets into organizations through infrastructure, and it grows out from there. Maybe it's a control plane, or an admin dashboard, or an internal tool for managing your service — something that doesn't need to be fast, it just needs to get built. And if the ecosystem is there, you can say "just use Rust" with a straight face. And when you do, they get fast and reliable for free.

And that's what I hope we talk about over the next couple of days. Not just how to make async Rust faster, but how to make Rust easier. How to make it productive. How to make it the obvious choice for more than just infrastructure. Because if we do, I truly believe Rust becomes the default — not just for performance-sensitive infrastructure, but for greenfield development in general.

Thank you. Alice is going to come up and talk about what's next for Tokio.
