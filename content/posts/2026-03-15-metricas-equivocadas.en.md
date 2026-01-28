---
layout: post
title: "Wrong metrics: why measuring speed is taking us to the wrong place"
date: 2026-03-15
categories: [AI, Technology, Culture]
tags: [AI, Metrics, Productivity, Complexity]
description: "Why measuring speed with AI pulls us away from value and increases complexity."
featureimage: "/images/posts/2026-03-15-metricas-equivocadas/feature.svg"
---

# Wrong metrics: why measuring speed is taking us to the wrong place

![Illustration of conflicting charts](/images/posts/2026-03-15-metricas-equivocadas/feature.svg)

If there’s anything we like in tech almost as much as a new tool, it’s measuring it.  
And if there’s anything more calming than measuring, it’s seeing a chart go up and to the right.

It doesn’t matter what we’re measuring.  
It doesn’t matter if we fully understand what it means.  
If it goes up, we’re doing well.

With AI, that reflex has sped up even more.

Suddenly, everything seems to improve according to the metrics:
more lines of code per day,  
more tasks closed,  
more documents generated,  
less “apparent” time to delivery.

Dashboards smile.  
The black box with a smile does too.

And someone says the magic phrase:  
> “See? It works.”

The problem is we’re measuring what’s easy, not what’s important.

We measure speed because it’s visible.  
We measure output because it’s countable.  
We measure activity because it fits nicely on a chart.

But we almost never measure:
understanding,  
maintainability,  
future debt,  
cognitive cost,  
or how many times someone thought *“this shouldn’t be like this”* and moved on.

That doesn’t fit neatly in a KPI.

This mistake isn’t new.  
We’ve been making it for decades—we’re just automating it now.

For years we measured support engineers by the number of incidents resolved. Not by complexity, not by impact, not by whether the problem came back. Just by how many were closed. The result was predictable: fast tickets, structural problems intact, and customers calling again.

We’ve also measured developers by the number of lines pushed to the repo. As if writing more code meant delivering more value. The result, again, was expected: bloated code, unnecessary abstractions, and systems that were harder to understand and maintain.

Measuring speed with AI is exactly the same error, just with nicer charts.

There’s something worth remembering—and we forget it too easily: **more code isn’t better code**.  
In fact, it’s often the opposite.

In security we understand this perfectly.  
The more software you install, the larger the attack surface.  
More binaries, more dependencies, more configurations, more points of failure.

Code is the same.

Every additional line:
- is a possible source of bugs,
- increases system complexity,
- raises review effort,
- and increases future maintenance cost.

AI is excellent at producing code quickly.  
But producing code quickly doesn’t mean producing better systems.

Here’s a dangerous confusion: activity isn’t progress.

AI multiplies activity.  
It generates more code, more variants, more plausible solutions.

But in complex systems, more is rarely better.  
And fast almost never means efficient.

One of the most perverse effects of measuring only speed is that it penalizes thinking.

Thinking doesn’t produce immediate output.  
Questioning a solution doesn’t close tickets.  
Stopping to understand something doesn’t move the sprint needle.

So the system learns fast:
the one who doubts delays;  
the one who reviews slows down;  
the one who asks “why” gets in the way.

Meanwhile, the one who accepts the box’s answer and commits quickly looks like a hero.  
At least for a while.

There are metrics that should matter much more, even if they don’t look as good on a slide.

For example:
- number of iterations before an engineer trusts the generated code,
- real time from first commit to a stable production version,
- number of reworks after delivery,
- frequency of changes in the same modules,
- post-deploy bug rate,
- or how many components nobody wants to touch.

These metrics aren’t theoretical. They can be captured.

Iterations can be measured in the workflow itself: discarded commits, reopened PRs, review cycles. Time to stability can be pulled from CI/CD pipelines. Rework shows up in issue history. “Untouchable” modules identify themselves: they always end up in the same hands.

It’s not that we don’t know how to measure these things.  
It’s that they’re uncomfortable.

Another common mistake is measuring reduced human effort without measuring displaced effort.

If writing code used to take eight hours and now writing, reviewing, debugging, reinterpreting, and re-reviewing takes ten, but we only measure the first two, AI looks like a huge success.

The effort didn’t disappear.  
It became invisible.

And invisible effort always comes due.

The black box with a smile contributes to this illusion.  
It responds quickly, confidently, without showing doubt. It feels like the hard work is already done. That all that’s left is “polishing.”

And then we start optimizing for the wrong metric.

Not for better systems.  
For prettier charts.

A mature organization doesn’t only ask how much faster it’s going.  
It asks what it’s sacrificing to go faster.

Because in engineering you almost always sacrifice something:
understanding,  
flexibility,  
learning,  
margin for error.

If you don’t know what you’re sacrificing, it’s not efficiency.  
It’s luck.

Maybe AI will let us go much faster.  
I hope so.

But if we only measure speed, we’ll never know when we’ve crossed the point where hidden cost exceeds visible benefit.  
We’ll never know when technical debt starts growing faster than delivered value.  
We’ll never know when we’ve trained the organization not to think.

And no new model fixes that.

Maybe the problem isn’t that AI writes too fast.  
Maybe the problem is that we keep measuring as if speed were the final goal.

And in systems that have to last for years,  
speed is rarely the most important thing.

Even if it looks great on a chart.

{{< smallnote >}}
**Note**: This article is part of a series that questions how we’re adopting AI in professional environments. Measuring is necessary. Measuring well is essential. Choosing the wrong metrics doesn’t just give you the wrong information—it changes the behavior of the entire organization.
{{< /smallnote >}}
