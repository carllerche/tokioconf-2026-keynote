
# Slide 01: Welcome

[Tokio logo]

Hey everyone! Welcome to the first ever TokioConf. Seriously, it is so exciting that we are actually doing this! I've been wanting a conference like this for years. Somewhere for practitioners building server applications with Rust to come together and go deep. Because, did you know, 19% of all crates on crates.io depend on Tokio? I think it's about time we, and by "we", I mean the async rust networking community as a whole, had our own conference... and now we do.

But, before I get into anything else — I have to thank Tiffanie, who did all the hard work to make this happen. All the logistics, and there was... a lot. So, thank you for taking all that on.

So, I was thinking about what to actually talk about up here and I started digging through old blog posts for inspiration... including the original blog post I wrote announcing Tokio, and I noticed something.

# Slide 02: Tokio turns 10

> 🎂 Tokio turns 10 🎉

Tokio is turning 10 years old this year... yeah. it was first announced in August of 2016!  That is crazy! When I wrote that blog post, in no way did I imagine it would end up being used by some of the biggest web services in the world. And I definitely didn't imagine that I would still be working on it 10 years later... that's a long time. Back then, I was just trying to work on a hobby project, I think I was playing around with a distributed database (they were all the rage back then), but there was no async networking library for Rust. So, I decided to shave that yak, and here we are.

# Slide 03: Let's reminisce

[old time ferris]

Ok, so it is the first TokioConf. I hope you will indulge some reminiscing. The year is 2015. Mio is just released. The very first RustCamp just happened (it wasn't even called RustConf yet). Javascript has promises, but no async/await syntax yet. I decided to start playing around with futures on top of Mio:

# Slide 04: Eventual

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

My first iteration was called Eventual... which actually is a pretty good name, I don't remember why I didn't stick with it to be honest. Anyway, this was an example of Eventual I dug up. You will have to bear with me a bit... this was 11 years ago. I hardly remember what code I wrote 1 year ago does. I would say it is relatively self evident. The most important thing to note is, this implementation, required an allocation and a callback at every single step. And backpressure was entirely your problem.

Needless to say, I'm glad this isn't what we ended up with. Because around the same time, some folks on the Rust team were also exploring futures — and they had a brilliant insight. They landed on a trait that was able to model futures while also maintaining the poll-based pattern that makes low-level I/O so efficient.

# Slide 05: Rust futures

```rust
pub trait Future: Send + 'static {
    type Item: Send + 'static;
    type Error: Send + 'static;
    fn poll(&mut self, task: &mut Task) -> Poll<Self::Item, Self::Error>;
    fn schedule(&mut self, task: &mut Task);
}
```

And this is the earliest version of the Future trait I could find, and you can already see the shape of today's future trait. Now, remember, up until now, every future or promise implementation, not just in rust, but all languages required an allocated callback and channel at each step. This trait was a breakthrough.

Instead of callbacks, the runtime polls the future, asking "are you done yet?" If it is, great. But what if it isn't? This is async — you can't just block the thread and wait. Something has to tell the runtime when to come back and poll again. That's what the second method, `schedule`, is for. The runtime calls it with a task handle, and the future implementation is responsible for using that handle to notify the runtime when it's ready. Conceptually, this is a very similar idea to what wakers are today, if you are familiar with that. Anyway, it worked, but in practice, when you implemented Future, the `poll` and `schedule` methods ended up being almost identical code. You were writing everything twice.

# Slide 06: Tokio intro

[Tokio logo]

And this gets us to the very first version of Tokio in August 2016. As I was building Tokio, that duplication between the `poll` and `schedule` methods kept bugging me. I think the solution that I came up with to avoid the duplication is probably the one bit of invention I actually brought to Rust's async story, so I'm going to revel in it for a bit.

# Slide 07: Tokio code

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

This is the read function for the TCP socket type in the very first version of Tokio — I dug up the code and copy/pasted it. Look at the signature — it's identical to a regular blocking read, and there is no separate `schedule` method. What I realized was: if the runtime is polling a socket for data, or any future really, and it isn't ready, we should just assume the runtime wants to be notified when the socket becomes ready. That's it. The poll itself already tells you everything you need to know. So `schedule` just... went away. And again, this seems obvious in hindsight, but it really wasn't at the time... which I guess is the best kind of insight. Anyway, I also thought it was clever to match the blocking read method signature exactly — the idea was that any existing implementation of the Read and Write traits could then just work with Tokio. In practice, that didn't work out so well, so it didn't stick. But that core idea — that polling implies wanting readiness notification — that did stick. It's the foundation of Rust async IO today...

So yeah, that was Tokio back then... very different. This was well before async/await. And to be honest, using it was not great. But people were still building with it.

# Slide 08: Pre-async/await

> It worked, but...

In 2017, I joined Buoyant to work on Linkerd, a service mesh, which you can think of as a fancy proxy. Anyway, they had originally built it in Scala, but a service mesh runs as a side process alongside every service on every server, — so the memory footprint has to be tiny. They needed to get the memory footprint under 10 megabytes. And, They tried every trick in the book to reduce the JVM's memory footprint, but it just wasn't happening. A rewrite was their last resort, and their options there were Rust or C, and Rust won because of the safety guarantees. But if they could have stayed on the JVM, they probably would have. And honestly, I don't blame them. We did successfully rewrite Linkerd in Rust, but without async/await, every single future was written by hand. Enums for every state, match statements for every transition. In a codebase that size... yeah. It worked, but nobody was choosing Rust for server applications because they wanted to — they were choosing it because they had to.

# Slide 09: Productivity is what matters

> Technology adoption is about productivity

Yes, rust is fast and reliable, but the productivity story wasn't there yet. And without fixing that, I wasn't sure how much further Rust would have gone in the server application space. Because, fundamentally, technology adoption is about productivity: how fast can I use a programming language, a library, a tool, to accomplish my goal.

# Slide 09: Async/await

> async/await

But then, something great happened... async/await for Rust happened... and it was a total game changer. It gave us a friendly programming interface for writing async code — and it did it without adding any runtime cost compared to writing those same state machines by hand. Arguably, it's actually better than doing it by hand, because you can borrow data across await points, which would be very hard to do safely without the construct. That's the good kind of zero-cost abstraction — the kind that's actually nice to use whether it's zero-cost or not.

We take it for granted now. Obviously Rust has async/await. But I remember listening in on conversations the Rust team was having, trying to figure out how to do it, and, at the time, it was not at all obvious that async/await in Rust was possible. I know I didn't think it was — or at least I had no idea how to make it happen. So, big props to the team that made it happen, including Aaron Turon, Taylor Cramer, and withoutboats.

And look, yes it isn't perfect. Yes Pin is hard to use... but you don't have to touch it most of the time, and when you do need to for some reason the pin-project macro is great. Yes being able to drop async blocks at any point of the execution can lead to confusing bugs... if we had known then what we know now, I'm sure the design would be different, but hindsight is 20/20 as they say. Overall, the async syntax we have today is so much better than what we were doing before.

# Slide 10: Show of hands

> ✋

Show of hands: who here used Tokio before async/await? ... now keep your hands up if you want to go back to pre-async/await. ... Exactly. Async/await is the killer feature that made using Rust for writing server applications actually productive.

# Slide 11: Where we are today

> Rust is the default for infrastructure

And look at what's happened since. I keep an eye out whenever a new open source infrastructure project is announced, and the majority of them are built in Rust. We have people from some of the biggest companies in the world, working on the biggest services in the world. Now, I know that I am probably biased, but from where I am sitting, Rust has become the default for greenfield server-applications where performance matters. And by all public metrics, Rust is still growing. And I don't think that would have happened without async/await syntax.

# Slide 12: Why stop here

> Why stop at infrastructure?

But here's the thing — performance, reliability, fewer bugs — those aren't just useful for infrastructure-level applications. Those are just... good things. For any software. So, not to be a Rust maximalist, but why isn't more software being written with Rust?

At the end of the day, it comes down to productivity. Pragmatic developers pick whatever gets them to their goal fastest. When the requirement is efficient and reliable, Rust gets you there quickest. But, let's be honest. Not all software has efficient and reliable as a requirement... Sure, those are nice to have, and all things equal, of course you will pick efficient and reliable, but not at the cost of slower development.

# Slide 13: Rust can be productive

> Rust = slow development?

And today, a common belief is still that Rust means slower development. But I don't think that's true. Here's what I've noticed. Teams that adopt Rust for infrastructure often start reaching for it for higher-level stuff too. Web apps, internal tools, things where performance isn't the main concern. Would they be doing that if using Rust actually slowed them down? No... So why are they? Because once you know Rust, you're past its biggest productivity headwind: getting started. You already know the language, the tooling, your team knows it, and you can share libraries across projects. At that point, Rust doesn't slow you down — and one language across the stack is a real productivity win.

Yes, the compile times are real. And yes, Rust can be a complex language. But the complexity? Those are power tools — lifetimes, advanced trait bounds, unsafe. You don't need them to build a web app. You can be highly productive with Rust while only using the "easy parts". The real gap isn't the language — it's the ecosystem. Rust has a vibrant ecosystem, but it's historically focused on infrastructure. At the higher level, there just aren't as many libraries. That gap matters, because a full-featured, easy-to-use library ecosystem is a huge part, maybe the biggest part, of what makes a language productive for a given use case.

# Slide 14: AI

> 🤖

Ok, so now is the part where I need to put on my flame-retardant suit. I have to talk about AI. Look, whatever your feelings are about AI, the fact is, it is fundamentally changing how software gets written. That's just happening. Yes, there are ethical questions, but the cat is out of the bag, and whether we like it or not, it has implications for Rust.

# Slide 15: AI lowers Rust's barrier

> AI lowers Rust's barrier

I said earlier that Rust's biggest headwind is getting started. Well, AI makes learning new things so much easier. I've seen devs that have never used Rust before drop in and be productive right away. No more three month learning curve. And no, I don't mean vibe coding — I mean using AI as a learning partner while doing real, high quality, work. Developers have been hearing for years that Rust is fast, and reliable, and safe — but the learning curve kept them out... not because they couldn't manage it, but simply because they didn't have time, or the motivation to take it on. That barrier is now essentially gone.

# Slide 16: Rust helps AI

> Rust 🤝 AI

And here is the thing. It isn't just that AI helps you write Rust. Rust helps AI generate code. We've all heard the term 'AI slop' — the AI goes off track and generates junk. Well, what keeps it on track? Guardrails. And Rust is full of them. The type system, the borrow checker, the culture of misuse-resistant APIs — the same things that help us write better code also help AI write better code. I think **that** makes Rust the best target language for AI tools out there right now.

# Slide here

> ??

But the compiler can only do so much on its own... And that's where libraries come in. The more functionality you can push into the library, the less code you or the AI has to write from scratch — and that's less surface area for things to go wrong. A well-designed library with strong conventions, leveraging Rust's type system to provide misuse-resistant APIs, shrinks that slop radius even further. I think that now, more than ever, in this new era we are entering of AI assisted or AI written code, these high quality libraries with well defined abstraction boundaries are going to become even more important.

So think about where that leaves us. The learning curve — the biggest thing holding Rust back — is disappearing. And Rust's own strengths make it the best language for AI. So if productivity is the same, possibly better, why would you not pick the language that gives you the most efficient and reliable end product.

I don't think there is a good reason.

# Slide 17: The opportunity

> Top 3 language for greenfield development

In fact, I believe that now, more than ever, Rust has the opportunity to become one of the top three languages for greenfield development in general.

But it won't just happen by itself. It will only happen if Rust becomes synonymous with productivity. That means a rich library ecosystem that handles the plumbing for you. [pause] It means easy APIs, for all high-level use cases, that get out of your way. That's what gets us there. And by the way, this isn't just about server applications. The same argument can apply to desktop apps, mobile apps, game development, whatever. But server applications is what what I know, and what this room builds, so that is what I'm going to talk about.

# Slide 19: Toasty

And I think the biggest missing piece for higher-level server applications in Rust, like web apps or backends for mobile apps, is at the database layer. If you think about what most of these kinds of apps actually do — they read from a database, apply some business logic, write back to the database. That's the core of it. The database layer touches everything. Get that right, and you unlock a huge amount of productivity. That's what I've spent the last two years working on. Toasty is an ORM for Rust. Let me show you what it looks like.

```rust
#[derive(toasty::Model)]
struct User {
    #[key]
    #[auto]
    id: u64,
    #[unique]
    email: String,
    name: String,
}

// Batch insert
toasty::create!(User::[
    { name: "Alice", email: "alice@example.com" },
    { name: "Bob",   email: "bob@example.com" },
]).exec(&mut db).await?;

let user = User::get_by_email(&mut db, "alice@example.com").await?;
```

That's it. Define your data, and start building. No lifetimes, minimal traits, no boilerplate. That's the kind of API I think we need more of in Rust. We all know the Knuth quote — 'premature optimization is the root of all evil.' I think that applies to API design just as much as it applies to code. We reach for traits when an enum would do. We add lifetime parameters to avoid a clone. We design for flexibility nobody asked for. And we end up with APIs that are powerful but painful to use. And I'm guilty of this too, maybe more than most (*cough* tower). In fact, in an earlier version of Toasty, I fell to the temptation. That query there, "get_by_email", it takes a string-like argument. I had initially added a single lifetime to the query to avoid having to copy the argument. I thought, one lifetime, how bad could it be. Then, I realized I was going against my goals for Toasty and removed the lifetime. I think it will end up being the right decision, the query API is very simple. Time will tell.

Now, you might have noticed — Toasty uses macros. A lot. And I know not everyone loves macros. The big objection is that they're a black box — you can't see the generated code, IDE support suffers, and when something goes wrong, the compiler errors can be cryptic. Those are real problems, and I don't want to dismiss them. But that means we should fix them and lean into macros, not avoid using them. In fact, I've already started a conversation with some the compiler team with ideas for improving error messages for macro-generated code. Because, one of the top things people cite that they love about Rust is how friendly the compiler error messages can be. I think there is a path to get compiler error messages for macros at the same level, where they can catch errors early and guide the user to fix it instead of being cryptic.

And that's exactly why these issues are worth solving — macros are already one of Rust's biggest developer experience superpowers, and could become more so. Think about serde. Serde is consistently cited as one of the best things about Rust — and not because of performance... because of the developer experience. You slap `#[derive(Serialize, Deserialize)]` on a struct and it just works. That's a macro doing all the heavy lifting, and nobody complains because the productivity boost is massive and it works the way you would expect. Macros give Rust the expressivity of dynamic languages like Ruby or JavaScript — but with full type safety.

But I think we can go further than just matching what other languages have. We can build libraries that take advantage of Rust's unique strengths, and be better because of it. This is a lesson I had to learn myself over the years. You saw Eventual earlier — that was me trying to copy the callback pattern from other languages, and it didn't work out well in Rust. But when we leaned into designing for Rust, we got Tokio. And, at least for me, that is a tough lesson. Every time I start a new library, I always go in hopeful I can just copy my favorite equivalent library from another language because... well that would just be easier. But, it never quite workers.  What works is leaning into what Rust is actually good at. So that's why, when I started working on Toasty, I ended up rethinking ORMs from the ground up... because, why not.

Now, I know what some of you are thinking. ORMs can trigger a strong response. Many of you have been burned before. I've heard things "I don't like ORMs. I'd rather just write SQL. ORMs are too complex." And you aren't wrong to be skeptical. But think about what SQL databases do under the hood — query planners, optimizers, tons of complexity. And we're all fine with that. The difference isn't the complexity — it's whether the abstraction actually holds up. SQL engines are smart enough to handle the complexity they take on. ORMs, historically,... just aren't. So you hit a wall, there's no escape hatch, and you throw it all out and go back to raw SQL. But that's throwing the baby out with the bathwater.

# Slide 20: Application data is graph-based

So when I started Toasty, I took the lesson I keep learning and asked — instead of copying ORMs from other languages, can I lean into Rust to do something better? [pause] So, the fundamental challenge with ORMs is that application data is graph-based — a user has many todos, a todo belongs to a category — but SQL is relational. And what existing ORMs do is take a query written against that graph model and try to directly translate it to SQL. That works for simple cases, but the two models don't map cleanly onto each other, so the translation breaks down fast. That's the wall everyone hits.

What if the ORM could understand your application's data model the same way a SQL database understands its schema? What if it could plan and optimize queries at the application level, the same way Postgres does at the database level? Then the translation doesn't have to be direct — it can be smart. The ORM can figure out the right way to get your data, even when the graph-to-relational mapping isn't obvious.

Now, to actually pull this off, Toasty needs something that, as far as I know, no ORM has tried before — a full application-level query engine. One that can actually understand, validate, plan, and translate queries across both levels. And this is where it comes full circle to Rust. Years ago, I was actually on the Ruby on Rails core team. In case you don't already know, Rails is a web framework, and ActiveRecord is its ORM — and, back then, we talked about giving ActiveRecord a deeper understanding of the application-level data models But Ruby was way too slow, so it wasn't really an option. Rust really is the only language where putting a query engine into an ORM makes sense because it is both expressive enough for high-level abstractions and performant enough to do the work.

And this is what excites me most about Rust. It's a language where you can build a query engine directly into an ORM. Think about what that means more broadly. Rust is a language where you can do work that's traditionally considered low-level — font rendering, layout engines, game engines, GPU programming — and high-level work like ORMs, web frameworks, developer tooling — in the same language. The ability to blend both into one is powerful. And honestly, I don't think we've even scratched the surface of what that makes possible.

And that's really what I want to leave you with. When we think about building that library ecosystem, we shouldn't just be recreating what exists in other languages. We should be asking: what does Rust make possible that wasn't possible before? That's how we get there — not by catching up, but by building something better.

[meta: maybe a joke about wanting to do more w/ power of the query engine. Something about "crazy ideas"]

That, and also you should totally use Toasty for all your ORM needs. Though, full disclosure, Toasty is still very early and there is a lot of work ahead. So, please try it and give me your feedback.

# Slide 21: Why this matters to you

So why am I talking about higher-level libraries to a room full of infrastructure developers? Because as Rust grows in your organization, you're going to be the ones building the shared libraries — the internal frameworks, hopefully some open source ones too. When you do, keep easy APIs as a priority. Don't reach for the power tools when you don't need them.

And beyond that — you're the ones who got Rust through the door. Now, your teammates are going to start asking if it's ready for more. Is Rust a good fit for the next control plane, or an admin dashboard, or an internal tool — something that doesn't need to be fast, it just needs to get built. And if the ecosystem is there, you can say "just use Rust" with a straight face.

That's what I hope we talk about — not just over the next couple of days, but going forward as a community. Not just how to make async Rust faster, but how to make Rust easier. How to make it productive. How to make it the obvious choice for more than just infrastructure. Because if we do, I truly believe Rust becomes the default — not just for performance-sensitive infrastructure, but for greenfield development in general.

Thank you. Alice is going to come up and talk about what's next for Tokio.
