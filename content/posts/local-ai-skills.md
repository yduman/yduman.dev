+++
authors = ["Yadullah Duman"]
title = "The Case for Learning Local AI"
date = "2026-04-07"
description = "The Case for Learning Local AI"
categories = ["Enterprise AI"]
tags = ["MLOps", "Local AI", "AI"]
+++

{{< notice note >}}
This post is fully written by me and was checked for typos and grammar errors with Claude.
{{< /notice >}}

We talk a lot about agentic coding, but there is a skillset emerging that I think is far more valuable and far less discussed.

I have been [deep in agentic coding workflows](/posts/six-months-of-agentic-coding/) and [debugging with agents](/posts/debugging-workflow-with-agents/). Through that work, I started noticing that the entire conversation around AI assumes that cloud APIs are a given. That assumption is breaking down fast.

I want to talk about local AI infrastructure skills: inference engineering, agent sandboxing, model routing, GPU optimization, and hybrid architectures. These are slowly becoming one of the most sought-after skills in our industry right now. On-premise infrastructure held [57.46% of AI infrastructure spending in 2025](https://www.mordorintelligence.com/industry-reports/ai-infrastructure-market), and that number is growing.

# The cloud-only narrative is breaking down

There is a compliance wall that most developers never think about. Hospitals cannot send patient records to AI providers. Banks cannot route financial data through third-party APIs. Defense contractors operate airgapped, with zero external connectivity. These are massive industries with massive budgets.

[Google deployed an airgapped AI appliance](https://cloud.google.com/blog/topics/public-sector/google-distributed-cloud-at-the-edge-powers-us-air-force-mobility-guardian-2025/) for the military in 2025. [Healthcare AI spending hit $1.4 billion in 2025](https://menlovc.com/perspective/2025-the-state-of-ai-in-healthcare/), nearly three times the previous year. These organizations need AI and they need it on their own terms, on their own hardware, behind their own firewalls.

Then there is the cost reality. Running inference through cloud APIs at scale is expensive. According to [LangChain's benchmarks](https://blog.langchain.com/open-models-have-crossed-a-threshold), open models like GLM-5 and MiniMax M2.7 are up to 10 times cheaper than proprietary alternatives. Running 10 million tokens/day costs roughly $12 with an open model versus $250 with Claude Opus. That is $87,000 in annual savings. At scale, the math is impossible to ignore.

In my work at [MaibornWolff](https://www.maibornwolff.de/en/), I see this question come up more and more. Customers want to use AI, but they cannot just send their data to the cloud. The first question is always: _can we run this locally?_ And the honest answer today is: yes, you can. But you need the right people to make it work.

# Open models are good by now

For a long time, the argument against local models was simple: they are not good enough. That is no longer true.

The results of [LangChain's benchmarks on open models for agentic tasks](https://blog.langchain.com/open-models-have-crossed-a-threshold) are compelling. GLM-5 scored 64% correctness on their benchmark. Claude scored 68%. A four-point gap, which for many use cases, doesn't really matter. Furthermore, GLM-5 achieves 0.65-second latency at 70 tokens/second. Claude sits at 2.56 seconds at 34 tokens/second. Open models are cheaper and faster.

Google's recent release of [Gemma 4](https://blog.google/innovation-and-ai/technology/developers-tools/gemma-4/) reinforces this. A 31B open-weight model built for agentic workflows, scoring 89% on AIME 2026 and 80% on LiveCodeBench, designed to run on consumer hardware. When even Google is shipping frontier-capable open models, the direction is clear.

For high-volume, privacy-sensitive workloads, local inference makes more sense now. Local models just need to be good enough for the job at hand, instead of trying to win every benchmark against their frontier counterparts.

# The emergence of inference engineering

Inference engineering is neither training models nor prompt engineering. It is the discipline of making models run efficiently at serving time. Quantization, batching strategies, KV cache management, hardware optimization, model routing. The [Pragmatic Engineer](https://newsletter.pragmaticengineer.com/p/what-is-inference-engineering) recently covered this emerging field and identified three essential domains: runtime optimization, infrastructure scaling, and developer tooling.

Quantization alone can yield up to 50% performance gains. That is the difference between a model that fits on a single GPU and one that requires an entire cluster. The difference between sub-second latency and multi-second responses. The difference between a viable product and a cost center.

The supply-demand imbalance is what makes this a career opportunity. There are still relatively few professionals working on inference, and newcomers can become experts quickly. The field is young enough that six months of focused learning puts you ahead of most. Meanwhile, the market is signaling loud and clear that it wants these skills. [AI engineer salaries jumped to an average of $206,000 in 2025](https://www.acceler8talent.com/resources/blog/ai-engineer--salary---market-rates-2025-2026/), a $50,000 increase from the previous year. Specialists in inference optimization and MLOps command significant salary premiums on top of that.

And the GPU economics reinforce the urgency. [H100 rental prices jumped 40%](https://newsletter.semianalysis.com/p/the-great-gpu-shortage-rental-capacity) between October 2025 and March 2026, from $1.70 to $2.35 per hour per GPU. All capacity is booked through August. Memory manufacturers are [prioritizing AI chips over consumer GDDR modules](https://www.clarifai.com/blog/gpu-shortages-2026), slowing gaming GPU production as data center demand eats into the global memory supply. When GPUs are this scarce and expensive, you need engineers who know how to squeeze every last drop of performance out of them.

# Security is not optional

When I wrote about my [debugging workflow with agents](/posts/debugging-workflow-with-agents/), I described agents instrumenting code, reading files, and running commands. That is powerful. It is also dangerous if the agent is compromised or hallucinating in the wrong direction. Now imagine that agent running on your company's infrastructure with access to proprietary code, customer data, or internal systems.

[The lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/) by Simon Willison plays a role here: an agent that has access to private data, processes untrusted content, and can communicate externally. When all three are present, prompt injection can exfiltrate sensitive data, and no guardrail can fully prevent it. 

Vitalik Buterin laid out [six critical threats](https://vitalik.eth.limo/general/2026/04/02/secure_llms.html) in his secure LLM post: data leakage to remote model providers, data leakage through model jailbreaks, accidental exposure, hidden backdoors, and software supply chain vulnerabilities. That last one is especially relevant right now. Poisoned MCP servers and malicious agent skills are the new attack vector, and since skills can ship assets, they are effectively binaries you are installing into your agent's runtime. 

This is where agent sandboxing becomes a discipline of its own. At MaibornWolff, we recognized this issue very early and therefore developed a sandboxing tool internally which works to our surprise very similarly to the recently released [OpenShell](https://github.com/NVIDIA/OpenShell) by NVIDIA. It implements defense in depth across filesystem restrictions, network policies, process controls, and inference routing. You give the agent autonomy to operate, but you constrain what it can touch. Simple but critical.

Luis Cardoso wrote a solid [overview of sandboxing approaches for AI agents](https://www.luiscardoso.dev/blog/sandboxes-for-ai) that is worth reading if this topic interests you.

Sandboxing is the only responsible way to let agents operate on infrastructure that matters. And the engineers who understand how to design these boundaries are exactly the ones enterprises are looking for.

# The skill gap is real

[84% of developers use AI tools](https://survey.stackoverflow.co/2025/ai), but only 18% are actually involved in building AI integrations. 76% said they do not even plan to use AI for deployment and monitoring. Almost everyone consumes AI through cloud APIs. Almost nobody knows how to deploy a model, tune it for specific hardware, or run inference locally.

Meanwhile, the market is moving in the opposite direction. Companies spent [$37 billion on generative AI in 2025](https://menlovc.com/perspective/2025-the-state-of-generative-ai-in-the-enterprise/), a 3.2x increase year over year. The edge AI market sits at [$25 billion in 2025 and is projected to hit $143 billion by 2034](https://www.precedenceresearch.com/edge-ai-market) at a 21% compound annual growth rate. [Nearly 90% of CIOs and CTOs](https://intuitionlabs.ai/articles/ai-engineer-job-market-2025) report creating new AI-related positions, while simultaneously worrying about workforce shortages.

The disconnect is staggering. Enterprises are pouring money into local and hybrid AI infrastructure, and the talent pool for engineers who can actually build and operate it is tiny. When I look at the engineers who are most in demand right now, it's the ones who can actually deploy and operate these systems. And there are shockingly few of them.

# Skills to look at

Let me get practical. If you want to build skills in this space, here is what the landscape looks like.

### Inference optimization

You need to understand quantization (reducing model precision to fit on smaller hardware), batching strategies (processing multiple requests efficiently), and KV cache management (how context windows eat memory). Runtimes like vLLM, llama.cpp, and TensorRT-LLM each have different tradeoffs. Knowing when to use which runtime for which workload is the kind of practical knowledge that separates someone who has read about local AI from someone who can actually run it. Everyone starts somewhere. Build it, understand it. Here is my PoC when I was learning: [ai-platform-poc](https://github.com/yduman/ai-platform-poc).

### Securing agents

Understanding the attack surface of AI agents is becoming essential. What filesystem access does the agent need? What network calls should it be allowed to make? How do you isolate the inference process from the rest of your system? How do you handle credentials? NVIDIA's OpenShell is a good starting point, but the real skill is thinking about security architecture for autonomous systems.

### Model routing and selection

Not every request needs the biggest model. A simple classification task doesn't need a 70 billion parameter model. A complex reasoning task might need to hit a cloud frontier model. The skill is designing a routing layer that sends each request to the right model based on complexity, latency requirements, cost, and privacy constraints. This is where hybrid architectures shine.

### GPU infrastructure

Understanding GPU memory, VRAM requirements, multi-GPU setups, and the economics of buying versus renting is essential. With GPU rental prices volatile and supply constrained, making smart infrastructure decisions directly impacts whether a project is viable or not. You should understand what fits on a single GPU versus what needs a cluster, and how to plan capacity accordingly.

### Hybrid architecture design

The majority of enterprises have adopted hybrid architectures. The question is never "local or cloud?" but "which traffic goes where?" Designing the routing layer, the fallback strategies, and the observability for a system that spans local and cloud inference is a real engineering challenge. And it is exactly what most enterprises need.

# Where I think we're heading

Some local AI use cases are already solved. STT with Whisper is as good locally as anything in the cloud. Image generation runs well on consumer hardware. Code autocomplete works with models like Qwen.

The frontier is local agents that can operate autonomously with proper sandboxing. NVIDIA investing in OpenShell signals that the industry sees this coming. Open models crossing the performance threshold, as LangChain documented, means the economics will only get better from here.

I would not be surprised if, a year from now, a significant portion of my agentic coding workflow runs locally. The models are getting there and the tooling is getting there. Now, engineers need to know how to put it all together.

The engineers who figure out how to run AI infrastructure locally, securely, and efficiently will have one of the most valuable skillsets in our industry. And right now, only a few are learning it.
