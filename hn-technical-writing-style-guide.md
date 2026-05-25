# Hacker News Technical Writing Style Guide

Lifecycle: active
Role: guide
Project: agent-playbook-suite
Updated: 2026-05-25

This guide is based on a reading pass over a small sample of popular Hacker News technical posts from roughly the last year. The sample is not a scientific ranking. It is a practical corpus of articles that clearly resonated with HN readers and are strong style references for blog posts and technical articles.

## Sample used

- [Show HN: I made an open-source laptop from scratch](https://www.byran.ee/posts/creation/) by Byran Huang. HN: 3237 points, 323 comments.
- [How I ship projects at big tech companies](https://www.seangoedecke.com/how-to-ship/) by Sean Goedecke. HN: 1425 points, 381 comments.
- [Things we learned about LLMs in 2024](https://simonwillison.net/2024/Dec/31/llms-in-2024/) by Simon Willison. HN: 984 points, 582 comments.
- [How I program with LLMs](https://crawshaw.io/blog/programming-with-llms) by David Crawshaw. HN: 919 points, 332 comments.
- [Upgrading an M4 Pro Mac mini's storage for half the price](https://www.jeffgeerling.com/blog/2025/upgrading-m4-pro-mac-minis-storage-half-price/) by Jeff Geerling. HN: 434 points, 275 comments.
- [How I keep up with AI progress](https://blog.nilenso.com/blog/2025/06/23/how-i-keep-up-with-ai-progress/) by Atharva Raykar. HN: 284 points, 118 comments.
- [How I got promoted to staff engineer twice](https://www.seangoedecke.com/staff-engineer-promotions/) by Sean Goedecke. HN: 92 points, 83 comments.
- [My approach to running a link blog](https://simonwillison.net/2024/Dec/22/link-blog/) by Simon Willison. HN: 53 points, 19 comments.

## What these posts have in common

They are concrete, opinionated, and useful.

They do not write like company marketing, academic papers, or generic “content.” They read like a smart engineer explaining something they have actually done, learned, or observed closely.

The common pattern is:

1. Start with a real problem, project, or claim.
2. State the point early.
3. Back it up with specifics.
4. Show tradeoffs, not just conclusions.
5. End with a compact summary or reusable lesson.

## Core style rules

### 1. Lead with the point

Hacker News readers are impatient in a good way. They want the article to declare itself quickly.

Good openings do one of these immediately:

- name the problem
- state the argument
- describe the build
- explain why the thing matters

Do not spend four paragraphs warming up.

Bad:

"Technology is changing rapidly, and over the years I have had many thoughts about..."

Better:

"I have shipped a lot of projects at big tech companies, and most engineers misunderstand what shipping actually is."

### 2. Be specific enough to be debatable

The strongest HN posts do not hide behind safe generalities. They make claims that a technical reader can test, argue with, or apply.

Examples:

- a concrete process
- a strong interpretation of how companies work
- a precise build choice
- a detailed failure mode

The article should make the reader think either:

- "yes, that matches what I have seen"
- "no, that is wrong, and I know why"

Both reactions are good. Vagueness gets neither.

### 3. Write from direct experience

Most of the strongest posts are grounded in first-hand work:

- I built this
- I tried this
- I shipped this
- I use this
- I learned this the hard way

That does not mean every post has to be autobiographical. It does mean the article should feel earned.

If the piece is a survey, make the survey obviously curated and informed, not scraped together.

### 4. Use clean sectioning

Popular HN technical posts are easy to skim:

- short sections
- descriptive headings
- one main idea per section
- compact paragraphs

Good headings are functional, not clever. They help the reader build a mental map of the article.

Examples:

- `Why use chat at all?`
- `Communication`
- `DFU Restore`
- `The median productive engineer`

### 5. Favor concrete detail over inflated prose

When the article gets technical, it should become more concrete, not more abstract.

Use:

- commands
- file names
- hardware parts
- error modes
- numbers
- constraints
- timelines
- examples

This is one of the biggest differences between HN-friendly writing and generic tech blogging. The strong posts give the reader something to inspect.

### 6. Explain tradeoffs honestly

Strong HN posts do not pretend the author found the perfect answer.

They often include lines like:

- this worked, but here is the cost
- this is useful for my situation, but maybe not yours
- this is better than the alternative, not universally correct
- here is where this breaks down

That honesty builds trust fast.

### 7. Use first person without over-centering yourself

The voice is usually personal, but not self-dramatizing.

Good:

- "This is how I use LLMs."
- "Here is what I think engineers get wrong."
- "I tried this because I wanted to know if it would work."

Less good:

- long emotional framing before the technical content
- performative vulnerability used instead of substance

The post is about the idea or the build. The author is the guide, not the subject.

### 8. Respect the reader’s time

This shows up everywhere:

- short intros
- minimal repetition
- summaries when helpful
- links to deeper material instead of re-explaining everything

Even long posts usually feel efficient. They earn their length by adding new information section by section.

## Common article shapes that do well

### Shape 1: Strong-opinion practitioner essay

Used well by Sean Goedecke.

Structure:

1. State a strong claim.
2. Explain why most people misunderstand it.
3. Break the claim into sub-points.
4. Use examples from real work.
5. End with a short summary list.

Best for:

- engineering career topics
- team dynamics
- architecture tradeoffs
- process and delivery

### Shape 2: Hands-on build log

Used well by Jeff Geerling and Byran Huang.

Structure:

1. Show what was built or modified.
2. Explain constraints and parts.
3. Walk through the critical steps.
4. Include measurements, photos, benchmarks, or diagrams.
5. Conclude with cost, outcome, and caveats.

Best for:

- hardware
- tooling
- infrastructure
- performance work
- reverse engineering

### Shape 3: Technical survey or annual review

Used well by Simon Willison.

Structure:

1. Define the scope.
2. Provide a clear map near the top.
3. Break the topic into themes.
4. Mix synthesis with examples and links.
5. Keep returning to a few central conclusions.

Best for:

- year-in-review pieces
- ecosystem analysis
- state-of-the-art explainers
- curated reading guides

### Shape 4: Short link post with value-add

Used well by Simon Willison’s link blog.

Structure:

1. Link to something interesting.
2. Quote the most important bit.
3. Add one useful thing:
   context, interpretation, criticism, comparison, or a related link.

Best for:

- commentary
- curation
- fast publishing
- keeping a blog active without padding

The key rule is simple: do more than summarize.

### Shape 5: Applied workflow write-up

Used well by David Crawshaw and Atharva Raykar.

Structure:

1. State the working question.
2. Describe the actual method you use.
3. Show where it works and where it breaks.
4. Give the reader a few rules of thumb.
5. End with scope limits, not grand claims.

Best for:

- AI or tooling workflows
- engineering habits
- debugging processes
- research or reading workflows

## Tone guide

Aim for:

- calm
- direct
- technically literate
- slightly opinionated
- non-salesy

Avoid:

- hype language
- TED-talk inspiration tone
- fake neutrality
- SEO listicle voice
- internal-corporate jargon unless it is explained

Good HN posts sound like someone useful explaining something to another useful person.

## What to copy

- Plain, descriptive titles
- A clear thesis in the first screenful
- Useful headings
- Specific examples
- Honest uncertainty
- Strong summaries
- Real constraints

## What not to copy

- long throat-clearing intros
- dramatic claims without evidence
- bloated history sections
- generic advice that applies to everything
- product marketing disguised as an article
- excessive abstraction with no examples

## Practical checklist

Before publishing, ask:

1. Does the title say exactly what the piece is about?
2. Does the opening tell the reader why this matters?
3. Is there at least one concrete example in every major section?
4. Did I include the constraints or tradeoffs?
5. Can a reader skim only the headings and still understand the shape?
6. Did I cut filler that does not add information?
7. Would a technical reader have something real to agree or disagree with?

## Recommended default template

Use this when drafting a technical post:

```markdown
# Clear, plain-English title

One-paragraph opening:
- what this is
- why it matters
- what the article will argue or show

## The problem

Describe the actual situation, constraint, or question.

## The key idea

State the thesis clearly.

## How it works

Walk through the implementation, process, or reasoning.

## What was harder than expected

Include failure modes, surprises, and tradeoffs.

## What I would do again or differently

Turn the experience into reusable advice.

## Summary

- short bullet recap
- one-sentence closing takeaway
```

## One final rule

Write like you are trying to help a skeptical, smart engineer decide whether your claim is true.

That is the center of gravity for good Hacker News technical writing.
