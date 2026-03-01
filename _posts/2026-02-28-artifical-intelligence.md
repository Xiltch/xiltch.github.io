---
layout: post
type: post
author: Jonathan Tweedle
title: There's no Intelligence in AI
categories:
- General
categories:
- General
- Programming
tags:
- Thoughts
permalink: "/2022/02/28/ai/"
---

![banner][banner]

Not sure if anyone noticed, but there is this new buzz that’s gaining popularity. The concept is not new to us, [The Matrix](https://www.imdb.com/title/tt0133093/) came out in 1999 exploring a dystopian future where computers ruled over us. Fast forward to 2022, the first version of ChatGPT is released. Is this the beginning of the end?

In the last three years, AI has gained a lot of momentum. Evolving from the early days of `Machine Learning` to something more commercial. Don't get me wrong, to help advance the field requires funding. In a capitalistic world, the end goal is convincing the public to willingly spend their money. But how do you do that with something that does not yield something physical?

Welcome the Chat Bots, with every iteration made to "feel" more and more a real person. Combined with the modern cellphone, you have this all knowing, all talented, and multi-skilled person ready to do anything for you. Gone are the days you need an interior designer to remodel your kitchen, you have at your finger tips ... someone with all the knowledge of every professional interior designer just itching to help you. Need help with your taxes, no problem, it knows all the tax rules! Struggling to connect to your printer, it has seen every tech video and none of the awkward tech guy persona to deal with. 

We are truly on our way to a future like that of [WALL·E](https://www.imdb.com/title/tt0910970/), we can sit back and relax while everything is done for us... right?

These new tools are great, it did save me a bunch of time on the banner for this post. My normal process for a new post starts by spending hours searching for images that fit the narrative, but licensed as free to use. Using AI to generate the image based on a simple two line prompt, reduced that down to about 5 minutes. The results was good enough for my purpose, and that is where we start touching some of my concerns for the future of AI in our future.

I do not think we are heading to a Matrix style dystopian future or even that of WALL·E. The current immediate problem I see is the over-hyped promise of what it can do. Companies are making big bets centred around the adoption of different AI technologies. Everything now has to have AI in it, cars, laptops, cellphones, toothbrushes, even shoes! This does not end with the final products, a need to embed it directly into our daily work life is gaining a lot of steam. Companies like Anthropic, Google and others are building tools fit for that purpose. Tools designed to super-charge your productivity and accelerate the time it takes to produce value output. This is where things start looking grim for the future.

Before computers became a core foundation in the office, the solution was hordes of people sitting at desks manually processing paper documents. Computes changed everything, what took say 10 people was now managed by a single person. Sure it was not great for the other 9 people but it helped save costs and increased productivity. Does our future spell doom for the last person ... maybe just have one person running the whole company, controlling an army of AI bots?

I don't think so.

How can I be so confident? Well the problem with AI is simple, it has no Intelligence. Sure it can be designed to feel like it does but it has no more intelligence than the index at the back of a book. It is just a little more advanced version designed to help wade through the growing mass of knowledge collated from the world. 

But you can have a conversation with it!

What you have is something "trained" on countless conversations that allows it to emulate a decent conversation through probabilistic recursive guessing of the next word. In essence the AI Models are glorified [Pachinko Machines](https://en.wikipedia.org/wiki/Pachinko), taking your [tokens](https://decagon.ai/glossary/what-are-ai-tokens) through a complex maze until it lands on a spot that provides a short list of probable words. It then randomly picks one of those answers and repeats the whole process again to figure out the next word.

This is not [intelligence](https://en.wikipedia.org/wiki/Intelligence), because using the model does not directly change the model. To train a new model, requires moving the pachinko pins around the board. This process requires massive data centres, consuming enormous amounts of resources as well as months to complete. This is evident when you consider the latest [model from Anthropic](https://en.wikipedia.org/wiki/Claude_(language_model)#Models), Claude Sonnet 4.6 only contains knowledge up to August 2025. If you consider a more general model like [Chat GPT](https://en.wikipedia.org/wiki/ChatGPT#Model_versions), even the time between minor release versions takes a few months. By the time the model is released, it is already out of date. This means a constant cycle of retraining is needed to keep them up to date.

So how then can AI chat bots know about current events, or the fact we can feed it all our company data to customise it results...

True, but that comes at a cost and this is how the AI companies plan to cash in. While training a model does take a lot of time, energy, and processing power. The resulting model requires less powerful hardware to run it. The same way assembling a calculator takes a lot of time, using one produces almost instant results. AI models are no different. Some models can be slimmed down with minimal functionality, such that it can run even on a cellphone. While the cloud versions of the more popular chat bots, run on dedicated [NPU](https://en.wikipedia.org/wiki/Neural_processing_unit) to handle more complicated models. They have more than enough memory available fit the whole model with space to spare. This is where the [context window](https://en.wikipedia.org/wiki/Large_language_model#Architecture) comes into play.

Running an AI model requires very fast memory, and there is a physical limit to how much memory can be physically attached directly to the NPU. The trained model uses some of that memory but leaves a lot free for the context window. When chatting to a AI bot, your conversation is broken up into tokens and each token takes up space in that context window. This is kind of like pre-loading the field of the pachinko machine. Every time your conversation continues, you are basically taking the whole conversation and feeding it into the AI over and over. This gives the illusion that AI is "remembering", but in reality your just repeating everything to it. So pre-loading all the business knowledge into the system, reduces the overall context window limiting the effectiveness. Eventually as the window fills up, strange things start to happen as it starts to [hallucinate](https://en.wikipedia.org/wiki/Hallucination_(artificial_intelligence)).

Technical advancements in hardware might help but [Moore's Law](https://en.wikipedia.org/wiki/Moore%27s_law) has a limit as we approach the physical limitations of the universe. Light cannot travel faster than light and we cannot manufacture things smaller than a single atom. Eventually we will hit a ceiling and while some clever engineering mitigates the problem, there can only be so much AI is capable of.

I am glossing over a lot of amazing engineering going into the field of "AI" and the tools they are producing. New models with improved reasoning will emerge designed to solve some real world problems. My concern is how some businesses may be tempted to down-size, placing an in-human amount of faith this technology will propel them towards massive financial gains. 

There is a limit to what the tools can do, and failing to fully understand the scope of them before forcing a square peg into a triangle hole cannot end well.

With growing pressure to deliver value faster than it can be curated, risks a sort of garbage spilling out. Will be able to clean up after it? We have seen the rise of "AI Slop", destined to feed back into the training data of future models. One can only imagine what that might look like. Then the risk of malicious or poison data from bad actors seeking to take advantage and compromise future systems. Hopefully we still have some skilled engineers left to protect us... 

In the professional setting, is there a risk of skill rot (skill decay) as our dependence on AI limits us. How will be people continue to work when the internet connection drops?

[banner]: /assets/2026/02/Gemini_Generated_Image_ou3cbcou3cbcou3c.png
[prev_post]: {% post_url 2026-02-22-catching-up %}