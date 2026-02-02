---
layout: post
title: "Decisions without an owner: when nobody decides, but someone pays"
date: 2026-02-22
categories: [AI, Technology, Culture]
tags: [AI, Responsibility, Governance, Complexity]
description: "How delegating to AI dilutes responsibility and reshapes who decides in an organization."
featureimage: "/images/posts/2026-02-22-decisiones-sin-responsable/feature.png"
---

# Decisions without an owner: when nobody decides, but someone pays

![Illustration about disconnected decisions](/images/posts/2026-02-22-decisiones-sin-responsable/feature.png)

There’s a moment, almost imperceptible, when an organization stops deciding the way it always did and starts doing it differently. There’s no official announcement or internal memo. There’s no clear “before and after.” It’s simply that one day someone proposes an idea and the response is:

> “Let’s ask the AI.”

At first it sounds reasonable. Even sensible.  
Why not do it, if the tool exists, responds fast, and seems to know what it’s talking about?

The black box with a smile enters the conversation like that—not as explicit authority, but as help. Support. A shortcut.

The problem is that shortcuts, once they become habit, end up redrawing the road.

![Illustration about shortcuts reshaping decisions](/images/posts/2026-02-22-decisiones-sin-responsable/scene-1.png)

In many organizations, the first experience with AI is dazzling.  
An executive uploads a hundreds-page document—a bid, a report, a complex proposal—and in seconds gets a clear summary, a structured analysis, or even an implementation plan. The feeling is immediate: *this works*.

And it does, truly.

AI is extraordinarily good at synthesis, drafting, reordering language, and framing ideas. That’s home turf. It’s not magic or a trick. It’s exactly what it was trained for.

The problem starts when that experience is extrapolated, almost without noticing, to a completely different kind of work.

Because not all problems are language problems.

When an engineer works on an architecture, infrastructure, or a system going to production, they’re not looking for a plausible answer. They’re looking for something much more uncomfortable: a solution that’s correct, coherent with their context, maintainable over time, and with known consequences.

There, it’s not enough for something to “sound good.”

This is where decisions come in that aren’t written anywhere:
- local conventions,
- tacit standards,
- accumulated lessons from past mistakes,
- limits you only learn after you’ve crossed them once.

That knowledge doesn’t usually live in official documentation. It lives in people.

![Illustration about tacit knowledge living in people](/images/posts/2026-02-22-decisiones-sin-responsable/scene-2.png)

To gain speed, many organizations have made a seemingly logical decision: tell engineers to use AI systematically. Have it write code, generate infrastructure, document systems. The argument is simple: if AI writes faster, the engineer only has to review.

In theory, it’s a clear improvement.

In practice, reviewing is not reading. Reviewing is **deeply understanding something you didn’t design**, detecting implicit assumptions, reconstructing decisions no one explained, and verifying everything fits into a system that existed before the box wrote a single line.

That’s not cheaper than creating from scratch.  
Often, it’s more expensive.

And still, it’s assumed the engineer should do it in less time, because “AI already did the hard part.”

Here we hit one of the most important tensions in this whole debate: **responsibility**.

Even if AI generates the code, proposes the architecture, writes the documentation, final responsibility remains human. If something fails in production, the model doesn’t fail. The box doesn’t fail. The person who reviewed, approved, and signed off does.

That person is the one who answers for incidents, explains what happened, and carries the consequences.

But many times, that same person:
- hasn’t had real authority to say “no,”
- hasn’t been able to set boundaries for AI use,
- hasn’t had enough time to review with judgment,
- and has worked under explicit pressure to go faster.

The equation is dangerous:
> **diffuse authorship + total responsibility**

![Illustration about diffuse authorship](/images/posts/2026-02-22-decisiones-sin-responsable/scene-3.png)

When that becomes normalized, the organization starts deciding without realizing it no longer really knows who is deciding.

This model has side effects you won’t see in the short term.

Senior engineers, with more experience, end up acting as system shock absorbers. They’re the ones who correct, refactor, put out fires, and carry the cognitive load of integrating what the box produces. Little by little, their work stops being about design and becomes about last-minute validation.

Juniors, on the other hand, learn to interact with AI before they’ve built their own judgment. They ask, get answers, and see that things work… without understanding why. And when something works without understanding, there’s no learning. Only dependency.

The organization, meanwhile, starts assuming that “AI does it better,” shrinking learning spaces right where generational handoff is formed.

The result isn’t immediate, but it is predictable:  
less distributed judgment, more dependence on opaque systems, and increasing fragility in the face of change.

None of this means AI is the problem.

AI is a powerful tool and, used well, it can greatly improve how we work. But like any powerful tool, **it has a domain where it adds value and another where it introduces risk**.

The conflict appears when it’s used outside that domain without redefining:
- how decisions are made,
- who has authority to stop them,
- how we train those who are starting,
- and who actually carries the consequences.

Because when a decision goes well, there’s always someone ready to celebrate it.  
But when it goes wrong, someone ends up paying the price.

And in too many cases, that someone:
- didn’t decide,
- didn’t have authority,
- didn’t have time,
- but still had to answer.

![Illustration about someone paying the price](/images/posts/2026-02-22-decisiones-sin-responsable/scene-4.png)

Maybe in a month the box will do everything perfectly.  
I hope so.

Maybe in a year we’ll look back and think these doubts were exaggerated.  
I hope that too.

I have no problem being wrong.  
In fact, it would be great news.

What worries me is that we’re making irreversible decisions—organizational, human, and strategic—**before we really understand what we’re delegating**.

Because organizations that decide faster than they understand  
don’t become more innovative.  
They become more fragile.

And fragility, at scale, rarely gives second chances.

---

{{< smallnote >}}
**Note**: This article is part of a series reflecting on the real use of AI in professional settings. It doesn’t aim to slow adoption, but to help use AI where it adds value and question it where it introduces risks we’re not measuring yet.
{{< /smallnote >}}
