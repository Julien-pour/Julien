---
author: ["Julien Pourcel"]
date: '2025-05-14T14:49:04+02:00'
draft: false
title: 'Self-Improving Language Models for Evolutionary Program Synthesis: A Case Study on ARC-AGI'
description: "Article about SOAR paper"
tags: ["ARC-AGI", "self-improvement", "code generation"]
params:
  math: true

cover:
    image: img/SOAR/soar_fig.jpg
    alt: SOAR architecture image
    caption: SOAR architecture
---


# Self-Improving Language Models: A New Frontier in Program Synthesis

Program synthesis is like teaching a computer to write its own code. Instead of giving it step-by-step instructions, you provide examples of what you want, and it figures out how to make it happen. It’s a powerful concept with huge potential, but it’s also incredibly challenging—especially for complex tasks like those in the Abstraction and Reasoning Corpus (ARC) benchmark. ARC puzzles test an AI’s ability to reason by asking it to transform colored grids based on just a few examples, and even the best language models struggle to crack them in one shot.

That’s where SOAR comes in—a groundbreaking framework called **Self-improving Operators for Automated program Refinements**. Introduced in the paper *"Self-Improving Language Models for Evolutionary Program Synthesis: A Case Study on ARC-AGI"*, SOAR helps language models get better at generating and refining programs by learning from their own attempts. This blog post dives into what SOAR is, how it works, and why it’s a game-changer for program synthesis.

## What’s the Big Deal with Program Synthesis?

Imagine you want to automate a task but don’t know how to code it yourself. With program synthesis, you can show the computer a few examples—like “input this, output that”—and it will generate a program to do the job. In ARC, these examples are visual: input grids of colored cells and their corresponding output grids. The AI has to deduce the transformation rule and write a program to apply it.

This is no small feat. ARC tasks often involve abstract reasoning—think pattern recognition, spatial relationships, or even basic physics—making them easy for humans but tough for machines. Traditional methods and even cutting-edge language models hit a wall here, solving only a tiny fraction of these puzzles without extra help.

## Why Current Models Fall Short

Even top-tier models like GPT-4o (4.75% success), Gemini-1.5-Pro (2.75%), or Claude-3.5-Sonnet (11.25%) falter on ARC tasks when asked to solve them in a single attempt. Smaller open-source models, like Qwen-2.5-Coder-7B, fare even worse at 1%. Why? These tasks demand reasoning beyond what a one-shot guess can handle. But give these models a chance to explore multiple solutions, and things start to improve—hinting at the potential for a smarter approach.

## SOAR: A Self-Improving Framework



![alt](/img/SOAR/soar_fig.jpg) 

<!-- <img src="/img/SOAR/soar_fig.jpg" alt="alt" style="width:100%;" /> -->


<!-- <img style="position: absolute; top:2000px; right: 32px" src="/img/SOAR/soar_fig.jpg" alt="peace dove" /> -->


SOAR flips the script by turning language models into self-learners. It’s built on a two-phase loop that keeps getting better with each cycle:

1. **Search Phase**: The model generates candidate programs and refines them using feedback from running those programs against the training examples.
2. **Learning Phase**: It takes the data from those search attempts—both successes and failures—and fine-tunes itself to improve for the next round.

This creates a virtuous cycle: a sharper model leads to smarter searches, which produce richer data for further improvement. Let’s break it down.

### The Search Phase: Exploring the Solution Space

In the search phase, SOAR starts by having the language model generate a pool of possible programs—say, 3,000 candidates. These are written in Python, not some restrictive custom language, giving the model freedom to experiment. Then, it refines these programs iteratively, using execution feedback to tweak them. Think of it like navigating a maze: the model tries different paths, learns from dead ends, and hones in on the exit.

To balance exploring new ideas and refining promising ones, SOAR uses a technique called REX (a bandit algorithm) and caps it off with majority voting to pick the best solution from the final pool of 6,000 candidates (half from generation, half from refinement).

### The Learning Phase: Turning Mistakes into Lessons

Here’s where SOAR shines. Instead of discarding failed attempts, it learns from them. For **generation**, it uses *hindsight relabeling*: if a program didn’t solve the original task, it’s still a valid solution for the output it *did* produce. This trick turns every attempt into a training example, creating a massive dataset—up to 2.4 million problem-solution pairs from ARC’s 400 training tasks.

For **refinement**, SOAR collects examples where an incorrect program was successfully fixed and uses those to teach the model how to polish its work. After fine-tuning, the model returns to the search phase smarter than before.

### Iterative Improvement: A Virtuous Cycle

SOAR doesn’t stop at one pass. Each iteration builds on the last, compounding gains. After four rounds, performance jumps by 13-16% across different model sizes. And it’s not just for training—it adapts to new tasks at test time, improving even without seeing the answers, thanks to a modified self-improvement loop.

## Results That Speak for Themselves

The numbers tell the story. On ARC’s test set, base Qwen-2.5-Coder models (7B, 14B, 32B) solve 8-21% of tasks with search alone. After four iterations of SOAR, this leaps to 24% (7B), 31% (14B), and 34% (32B). Combine outputs from all three sizes, and SOAR hits **41.25%**—a new state-of-the-art for program synthesis with open-source models on ARC.

Compare that to Claude-3.5-Sonnet’s 11.25% in one shot, and it’s clear: SOAR lets smaller models punch above their weight through self-improvement.

| Model               | 1-Shot (%) | Search-6k (%) | SOAR After 4 Iterations (%) |
|---------------------|------------|---------------|-----------------------------|
| Claude-3.5-Sonnet   | 11.25      | -             | -                           |
| Qwen-2.5-Coder-7B   | 1.00       | 8.25          | 24.00                       |
| Qwen-2.5-Coder-14B  | 1.50       | 18.00         | 31.00                       |
| Qwen-2.5-Coder-32B  | 2.25       | 21.13         | 34.00                       |
| Combined (SOAR)     | -          | -             | 41.25                       |

## Why This Matters

SOAR proves AI can bootstrap its own growth without mountains of human-labeled data. By learning from its own search experiences, it turns failures into stepping stones, paving the way for systems that adapt and improve on the fly. Imagine AI assistants mastering new skills as they go, or solvers tackling novel problems by building on past efforts—SOAR makes that future feel closer.

## What’s Next?

The paper hints at exciting possibilities: scaling SOAR to bigger models or more iterations, testing it on other benchmarks, or blending it with techniques like reinforcement learning. Could it crack even tougher reasoning tasks? Time will tell.

## Conclusion

SOAR is a leap forward for program synthesis. By weaving language models into a self-improving loop, it transforms static tools into dynamic learners, achieving impressive results on ARC and setting a new benchmark for open-source AI. More than that, it’s a blueprint for building systems that grow smarter over time, pushing us closer to AI that truly reasons and adapts like we do.

