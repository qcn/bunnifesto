---
layout: post
title: QC's Technical Design Bunnifesto 🐇☭
excerpt_separator: <!--more-->
---

This post is a slightly revised version of a document I wrote about a year ago
to share some gentle technical planning guidance with colleagues. It's not
intended to provide a template for writing technical design documents; every TD
is probably going to look a bit different, based on its scope and audience.
Instead, it's a narrative around **what you should think about** in order to
write an effective TD, **what you and others should be able to get out of it**,
and some guidelines on how to get started with your tech spike and written
document.

<!--more-->

---

### First, some context and definitions:

A **technical design document** (TD or TDD, not to be confused with test-driven
development), describes a solution to a technical problem. You might also see it
called a technical specification, engineering design doc, or similar, but they
all serve the same purpose: communicate _what_ the solution is, _how_ we're
going to build it, and _why_ we're going to do it that way.

A project will typically start with a **product requirements document** (PRD),
which describes what the _problem_ is and what the solution needs to do. From
the requirements, we build out a TD to guide us through the implementation of
the solution. TDs I've worked with typically cover both the general system
architecture and the implementation approach for the project, but your team
might split this into two separate documents.

While developing your TD, you'll typically engage in one or more **tech
spikes**: this is where you're building the simplest possible proofs of concept
to explore the technologies you're considering, so you can answer your own
questions about what they can do and how they compare to each other.

---

### And now to the bunnifesto:

**The ultimate goal of a TD** is to reduce technical risk. If you can't use it
to build a work plan, it's not finished. The TD helps with planning by giving a
bit more certainty around scoping -- it's the document that lets you say "I
think we can deliver this much in 6 weeks" and _be reasonably certain_ about it.

You're not trying to rigorously define the entire project -- at that point you
may as well have just written the code instead of the TD. Figure out the general
approach/technologies/interfaces needed, leave the implementation details to
whoever picks up the ticket.

What's your end goal for this document? You're fearlessly (or fearfully) leading
your team into this project work. Your goal should be to anticipate their
questions and factor them into your proposal. You want your teammates to know
you've thought through everything -- the **scope**, the **risks**, reasonable
**alternatives** and their **tradeoffs** -- so they can **trust** your
recommendations about how to proceed. You want your manager to be able to read
the proposal and [just say yes](https://www.rubick.com/completed-staff-work/).

Your work plan might have **known unknowns** -- tasks that require some
exploration, you know what you need to do and you probably have some idea about
how to do it, but you might need to choose between a couple of approaches/scopes
at the time or read up on something unfamiliar. You're trying to avoid
[**unknown unknowns**](https://www.youtube.com/watch?v=REWeBzGuzCc), these are
what can really blow out a project. They're what happen when you say "she'll be
right" and launch into the project and then six weeks in someone says "OK, but I
need to be able to _read back_ that data too" and a look of horror slowly
appears on your face as you realise you designed and built an entire system
where you prevented data exfiltration by only ever storing cryptographic hashes
of user input and you forgot the next stage of the project involved displaying
that data. Oops.

On that note, **requirements**: you should be aware of the **scope** of your
design and how it fits into broader priorities. Read and re-read the PRD. Don't
spend a lot of time designing for the project you're not building right now.
Requirements change. _Do_ spend time _architecting_ for future requirements
though -- this feature might involve write-only data right now, but you know
down the line we're going to want to be able to read it back/scale it up/etc.
You don't want to have to completely rebuild the system in the near future. (For
this reason, we generally don't ask less experienced engineers to write TDs on
their own -- their main focus is still on learning to build solutions, as they
gain more experience they'll be able to see the bigger picture and think about
planning for the future.)

And make sure your TD covers _all_ your current requirements! Make a list. Tick
them off. If a requirement isn't clear enough or you don't understand it fully,
go and ask people. There's little point in building a beautiful system that
can't do what its users needed in the first place.

Don't forget the implicit non-functional requirements either -- the ones that
apply to all engineering work we do. Even if the PRD doesn't explicitly call out
security, performance, reliability, maintainability, usability, and other
"ilities", you should always keep these technical requirements in mind.

---

A lot of the work in writing a TD is considering technology choices and making
tradeoffs. Think about our current patterns and what the team already knows how
to use -- if we're already heavily integrated with AWS services everywhere and
know how to use them, and AWS and GCP have basically feature-identical offerings
that fit the bill, _use the AWS one_. That JS package may do _everything_, but
do we really need it to do everything -- is there a smaller package that does
_just what we need_ and is simpler to configure as a result? Is that new
hipsterware _actually_ prod-ready? Apply the principle of technological
minimalism, but be aware that it has its limits: as a team, we want to minimise
the number of technologies we need to maintain, but sometimes a well-informed
decision to introduce a new technology is the best choice.

**Write down what your tradeoffs are.** Don't just document what you do want to
use. Document what you _considered_ and rejected, and why. This gives you and
your reviewer confidence that you've deeply understood the requirements and
tradeoffs, and also means that when that ~~lagomorph~~ person who has an Opinion
on Everything but didn't actually read the TD pipes up with "why didn't you
choose BunnyRabbitWare?!" you can point to the TD without having to even argue.

**Broaden your options before narrowing.** Even if you're pretty sure you know
how we could do this, try to think of another way, even if it's silly and just
in your head. This makes you think about _why_ the silly thing is bad and thus
_some things that might be bad_ (that you want to avoid) and gives you some
baseline to compare against. If you can't come up with another option, you
probably don't understand the problem well enough.

Take the time to understand the **current state of the system** and **prior
art** (and write that down too). It might inform your options. Does this look
like something we're already doing -- did that work well, do we want to follow
the same example, or do we want to take a different approach after learning from
our mistakes there? If we have two different ways we're currently doing
something, which one is the modern/preferred pattern? If your feature hooks into
legacy code, do you need to rewrite a bunch of existing modules for
compatibility with the Modern One True Way (you know, as opposed to last year's
One True Way), and how much extra time is that going to take? If you don't have
that much time, can you limit the scope of what you need to rewrite, and at
least avoid introducing new legacy code even if you can't rip it all out?

---

You want to **break down the tasks** in such a way that you can ship **partial
value** even if you don't complete the whole project. Priorities might change,
or you might run out of time. If you can ship what you have and leave the system
in a good state to build on it in the future, this is a good outcome!

**Think about how you need to interact with existing systems too.** Draw on your
understanding of the system architecture. Your new feature _feels_ standalone...
but does it need to integrate with those constant background maintenance tasks?
Have you considered that strangely-implemented-but-important widget that no one
ever remembers? If you have questions about these things, ask now, don't wait
until the project has already started. If you _don't_ have questions, ask anyway
-- no matter how low your personal bus factor is, it's almost impossible for you to know
about every interaction in the system, so pick your colleagues' brains for
anything they can think of!

---

**Level of context:** as the author of the TD, you have all the relevant context
in your head. Don't assume your reviewer has all the same context, even if
they're the CTO (probably especially if they're the CTO; there's a lot of other
stuff floating around in their head). They shouldn't have to go digging through
the whole PRD to understand what needs to be done, so explain _why_ you're
making your choices -- don't rewrite the PRD, but don't be afraid to give some
background information.

Technical background information is _definitely_ a good idea. Explain the
current state of the system and what you're slotting into.

What's your narrative? Think about providing a smooth reading experience. It's
fine to write prose. Sometimes it's easier to grok than dot points. If you're
asking if there should be a diagram, the answer is almost certainly yes.

Ask around broadly for people to review your TD, but chase people specifically
too! Find the people who care the most about the design or are most likely to
disagree with you. Get them on board now. **Give your reviewer everything they
need to give you informed feedback.** You _want_ them to rip things apart at
this stage, before your team actually starts work.

---

On that "before your team actually starts work" point, let's talk about **tech spikes**: this is
an **exploration phase** and should be used to assess viability of an approach,
_not_ to pre-implement the feature.

- **Avoid confirmation bias.** Remember to consider different options and make
  sure you know what question you're trying to answer. The question is probably
  more like "Which bunny should I adopt?"[^1] than "should I adopt this bunny?".
  Don't get tunnel vision and only try to validate the single solution we've
  been talking about for months.
- The most you should implement is a proof of concept, not a prod-ready
  prototype. **Throw away the code!** Say it with me: _it's okay to throw away
  work_. The product of that period of work was the journey and what you learnt
  from it. **Do not build the feature. Do not collect $200 (yet).**

  <figure>
    <img src="/assets/images/td-bunnifesto/the-climb.jpg" alt="A frame showing
    Miley Cyrus in the music video for The Climb" />
    <figcaption>In the words of the great philosopher Miley Cyrus, as I once
    explained to an unsuspecting CEO</figcaption>
  </figure>

- The idea is to work out whether this technology choice would meet the
  requirements, not to actually build the feature. For example, if the
  requirement is for a text-to-speech feature, the tech spike might involve
  playing around with AWS Polly in the AWS console to make sure we can feed in
  text and get it back in a voice that's acceptable, as opposed to building the
  actual integration between AWS Polly and our codebase.
- Look for prior art. You want to figure out if this looks like something we do
  already and deep-dive into understanding how that works if so. If this needs
  to integrate with existing systems, you need to understand how those work and
  how your new thing will tie into it.
- Consult experts! No engineer is an island, and just like when you're writing
  code, you should seek input and clarification early and often. You're trying
  to become the technical expert on Your New Thing, and that means you need to
  understand the context around it.
  - Experts aren't just limited to your colleagues -- if there's someone in your
    network who's solved this kind of problem before, reach out to them! If
    you're considering third-party systems (eg. AWS) as part of your solution,
    you might have access to their support/experts too.
- Definitely don't _build the feature_ as your "tech spike" and then go back and
  retcon the TD to fit it!

[^1]:
    The answer to this is "all the bunnies", so it was probably the wrong
    example to use.

---

Don't retcon the TD after actually implementing the solution, either. But if you
can, _document_ your divergence -- the best-laid plans go oft awry, after all,
and codebases evolve over time.

In the words of [a friend](https://github.com/AllyMarthaJ) who has seen (and
perhaps even been responsible for) many, many programming sins:

> Tell me why your plan failed!!!! If you don't you're as bad as that one
> stackoverflow user who asked a question and then replied with "I solved it!!!"
>
> HOW????? HOW DID YOU SOLVE IT USER OF 9 YEARS AGO?????

Knowing why the initial attempt _didn't_ go as planned can be core to
understanding why the eventual solution works. Future archaeologists will thank
you.

---

### "_QCCCC_," you whine, "that's _too much text_. Just tell me how to get started."

Sorry, friend, I'm not here to give you a template. I want you to _read_ all
those words! But because I'm feeling generous, a basic structure to start with
might be:

1. **Problem statement.** What's the problem we're trying to solve? _Why_ is it
   a problem (that needs to be solved with technology)?
1. **Background information.** What does the system currently look like? Which
   parts of it support what we're trying to do, what do we need to build on to
   solve our problem?
1. **The broadening: solution space.** How might we solve the problem? What are
   our options? What are the tradeoffs between them?
1. **The narrowing: recommendation.** We've talked about what we _could_ do.
   Now, what _should_ we do?

---

### "Is there a checklist for writing a TD?"

Yes, it's this bunnifesto.

More seriously, though, I don't believe a generic checklist provides enough
utility; as I mentioned in the preamble, every TD is going to be different,
because so much of what goes into it is contextual. Instead, I'd encourage you
to re-read this bunnifesto with the context of _your project_ in mind, and
construct your _own_ list of things to ask yourself (and other people).

Make sure your TD answers the right questions. If you're still not sure what
those questions are, ask people who should know, until you find ~~the answer to
the question of what questions you should answer~~ out.

🐰🐰🐰

---
