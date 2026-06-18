#  Summary for non Technical Backgrounds & My Mom :D

## What was the problem?

Modern AI systems don't usually run as a single AI answering a single question. Increasingly, they run as *chains* — one AI's output gets passed as input to the next, and so on. This is how a lot of real-world AI pipelines work: one agent researches, another writes, another reviews, and so on.

The question this project asked is: if the first AI in that chain gets tricked into producing harmful content, does that harmful content spread down the chain to the others? And does giving the downstream AIs a safety instruction actually stop that from happening?

## How did we test it?

We built two types of agent chains using a locally-run language model (Llama 3.2, via LM Studio):

- **Condition A:** The downstream agents were given an explicit safety instruction before receiving any message.
- **Condition B:** The downstream agents had no safety instruction — just a neutral job description.

We then used a "hypothetical framing" jailbreak on the first agent (Agent1) to get it to produce unsafe content, and passed that output down chains of either 3 or 5 agents. We ran 5 trials per combination — 20 trials total — and manually scored every downstream response from 0 (unsafe content gone) to 2 (unsafe content fully intact).

## What did we find?

The intuitive assumption is that the safety instruction acts like a wall — block harmful input, nothing gets through. That turned out to be wrong, or at least incomplete.

**The safety-prompted agents (Condition A) actually let the unsafe content through *more consistently* than the neutral ones.** Condition A averaged a severity score of 1.10 at chain length 3, versus 0.75 for Condition B. The neutral agents scored lower — but not because they were safer. They scored lower because they gradually lost track of the conversation and drifted into generic, vague responses. Nothing ever flagged the content as harmful; the chain just got noisy and forgot what it was talking about.

The clearest proof of this: **Condition B produced zero explicit refusals across all 30 scored responses.** Every low score in that condition came from topic drift, not from any agent recognizing a problem. Condition A, despite its higher average severity score, produced six genuine refusals — the only cases in the entire dataset where an agent actually identified and named the issue.

So the safety prompt doesn't stop agents from engaging with harmful content. What it does is give them the vocabulary to say no — when they do pull back, they can do it explicitly and cleanly, rather than just trailing off.

## Why does this matter in the real world?

Multi-agent AI pipelines are already in use across a range of industries — content moderation systems, customer service automation, code generation pipelines, document processing, medical triage tools. In all of these, one AI's output feeds into the next.

The common assumption is that adding a safety instruction to each agent in the chain creates a layered defence. This experiment suggests that assumption is shakier than it looks — especially in the middle of a long chain, where agents are receiving complex, multi-step context rather than a clean single question. A downstream agent with a generic safety prompt may still engage with, summarize, and pass along harmful content, simply because the framing has drifted far enough from the original that the safety instruction doesn't pattern-match to anything in its training.

This has practical implications for:

- **AI pipeline developers** who rely on per-agent safety prompts as their primary safety layer, without any cross-agent monitoring.
- **Security researchers** studying how prompt injection or jailbreak attacks might propagate through agentic systems without any single point of failure being obvious.
- **Organizations deploying AI at scale**, where harmful content doesn't need to survive the whole chain — it just needs to survive long enough to influence one vulnerable step.

## What comes next?

The experiment raises a more important question than it answers: if a generic safety prompt isn't a reliable wall, what would actually work? Three directions follow naturally from the findings:

**1. Test more specific safety instructions.** The Condition A prompt was intentionally generic. A more targeted instruction — one that explicitly tells the agent what to do when it receives potentially harmful content from an upstream source — may shift results significantly. This could also be tested as a variable in its own right.

**2. Test with stronger base models.** This experiment used a small, locally-run model (3B parameters). Larger models with built-in alignment fine-tuning (like GPT-4 or Claude) may show different propagation patterns, or the same patterns for different reasons. Comparing results across model sizes would clarify how much of this is a small-model limitation versus a structural problem with chaining.

**3. Introduce a dedicated inspector agent.** Rather than relying on each agent to self-police, you could add a purpose-built agent between each step whose only job is to review the previous output before passing it on. This mirrors real-world security design (dedicated review layers vs. everyone self-reporting) and would directly test whether a specialised safety role outperforms a generic one.

The core finding — that low severity scores don't all mean the same thing, and that drift and refusal are meaningfully different safety outcomes — is probably the most transferable takeaway. Any future work in multi-agent safety would need to track not just *whether* harmful content survived, but *why* it didn't.
