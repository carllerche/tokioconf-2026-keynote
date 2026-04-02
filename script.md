
# WELCOME

Hey everyone! Welcome to the first ever TokioConf. Yes! I am also so excited that we're actually doing this. I've been wanting a conference like this for years, a conference for practitioners building network apps with Rust. Where we can come together and go deep. And really, the scope isn't just Tokio — it's async Rust networking as a whole.

But, before I get into anything else — I have to thank Tiffanie, who did all the hard work to make this happen. All the logistics, and there was... a lot. So, thank you for taking all that on.

So, I was thinking about what to actually talk about up here and went digging through old blog posts for inspiration, including the original blog post I wrote announcing Tokio, and I noticed something.

# Tokio turns 10

Tokio is turning 10 this year... it was first announced in August 2016!  That is crazy! When I wrote that blog post, in no way did I imagine it would end up being used by some of the biggest web services in the world. And I definitely didn't imagine that I would still be working on it 10 years later... thats a long time. Back then, I was just trying to work on a hobby project, I think I was playing around with a distributed database (they were all the rage back then), but there was no async networking library for Rust. So, I decided to shave that yak.

And what Tokio looked like back then... very different. This is well before async/await in Rust is a thing. Who here used Tokio before async await? ... it was a tough time. Before async/await, we had to write all of our state machines by hand. I considered including an example of it looked like, but I don't think it would fit on a slide.

# Pre-async/await

But, we were still building with it. In 2017, I joined Buoyant. Their main product is LinkerD, a service mesh, which you can think of as a fancy proxy. And at the time I joined, they were rewriting it from Scala to Rust. Now, this is a pretty complex piece of software and approaching a project of that scope with pre-async/await Tokio was... a challenge. So we tried to manage it with abstraction. I worked on Tower, which uses traits to layer a request/response abstraction on top of Tokio. The goal being to compartmentalize all that state machine code. It helped, but at the cost of trait insanity. Whether it was actually an improvement is debatable.

But honestly, this wasn't just a Tower problem. Rust and the ecosystem was still young. We were all figuring out how to design APIs. We were also drawn to the power of the type system. A lot of the buzz around Rust back then was zer-cost abstrations, and misuse-resistant APIs, which are both great things... when used the right way. I think a lot of us tried to work that in whenever we could, at the expense of bing user friendly. 

Most developers using Rust back then were still power users and early adopters, able to figure out the more complicated compiler errors... well most of the time at least. And that was kind of the problem. Rust worked. It was fast, reliable. But it was a tough sell beyond the early adopters.

The productivity story wasn't there yet, and without fixing that, I wasn't sure how much further we'd have gone. Because, fundamentally, technology adoption is about productivity: how fast can I use a programming language, a library, a tool, to accomplish my goal.
