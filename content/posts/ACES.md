---
author: ["Julien Pourcel"]
date: '2025-05-14T14:48:49+02:00'
draft: false
title: 'Generating a Diversity of Challenging Programming Puzzles with Autotelic Generative Models'
description: "Article about ACES paper"
tags: ["autotelic", "Quality-Diversity", "code generation"]
params:
  math: true

cover:
    image: img/ACES/aces_fig.jpg
    alt: ACES architecture image
    caption: ACES architecture
---

<!---
TODO:
- need to recheck all stuff
- add colab demo
- add code link
-->

# Generating a Diversity of Challenging Programming Puzzles with Autotelic Generative Models (ACES)

## **Introduction**

Human intelligence is marked not just by the ability to solve problems, but by the creative act of inventing them. Automating the generation of novel, diverse, and challenging problems has wide-ranging applications-from personalized education to robust benchmarking of AI systems. The [`ACES`](https://neurips.cc/virtual/2024/poster/95626) (Autotelic CodE Search) framework, accepted as a **Spotlight Poster 💫 at NeurIPS 2024**, introduces a principled method for generating Python programming puzzles that are both difficult and semantically varied, pushing the boundaries of what current generative models can achieve alone.

---

## **What is ACES?**

ACES is an autotelic generative algorithm designed to automate the creation of Python programming puzzles. Inspired by intrinsic motivation in human learning, ACES seeks to generate puzzles that are:

- **Challenging:** Difficult for state-of-the-art language models to solve.
- **Diverse:** Spanning a wide range of programming skills and concepts.

The ACES pipeline leverages large language models (LLMs) both as puzzle generators and solvers, iteratively refining its archive of puzzles to maximize both difficulty and diversity.

---

## **How Does ACES Work?**

### **1. How to define a meaningful diversity?**

Measuring diversity in programming puzzles is challenging. Simple metrics like the number of lines of code do not capture meaningful conceptual differences and can be misleading, as code length often reflects style rather than substance. Using learned code embeddings can provide quantitative variety, but these representations are opaque and not easily interpretable by humans.

To address this, we defined diversity in a semantic, interpretable space: each puzzle is labeled with a combination of up to five out of 20 programming skill descriptors (such as recursion, dynamic programming, or array indexing), which are automatically assigned by a large language model. This approach ensures that diversity is both understandable and generalizable, as the descriptors can be adapted or extended by the LLM to cover new domains or topics. Furthermore, given this diversity space we can do goal-targeted generation of new problem given a set of semantic descriptor, this allow us to more efficiently explore this diversity space and so to explicitly maximize the diversity, whereas it is not possible given a diversity space based on embedding.

### **2. Semantic Skill Space**

Each puzzle is represented by a set of semantic skill descriptors (e.g., recursion, string manipulation, dynamic programming). These descriptors are assigned by an LLM labeler from a curated list of 20 programming skills, ensuring that diversity is measured in a way that aligns with human intuition.

### **3. Difficulty Assessment**

Puzzle difficulty is empirically measured using the success rate of a strong LLM solver (e.g., Llama-3-405B, Mistral-Large, Qwen-coder, ...), pass@1 estimated over $n$ attempts ($n=50$). A puzzle is considered more difficult if the solver rarely succeeds within n attempts and vice-versa.

### **4. Iterative Quality-Diversity Search**

ACES builds on the Map-Elites evolutionary algorithm, maintaining an archive of puzzles grouped by their skill combinations ("niches"). The process is as follows:

- **Sample a Target Niche:** Select a combination of skills for which to generate a new puzzle.
- **Select Examples:** Retrieve challenging puzzles from neighboring niches as few-shot examples.
- **LLM Generation:** Prompt the LLM to generate a new, harder puzzle targeting the sampled skills.
- **Evaluation:** Attempt to solve the puzzle with an LLM. Only puzzles that are solvable (but difficult) are retained.
- **Skill Labeling:** Use an LLM to assign skill descriptors to the new puzzle.
- **Archive Update:** Add the new puzzle to the appropriate niche in the archive.

This loop continues, progressively filling the archive with a greater diversity of more challenging puzzles.

---
## Benchmark generator

The ACES framework is not only a generator of diverse and challenging programming puzzles, but it is also a benchmark generator for evaluating the capabilities of code-generating language models. 

ACES-generated benchmarks are built by iteratively creating archive of Python programming puzzles for a given number of iteration, this let the user choose the number of problem in the benchmark and the targeted difficulty can also modified in many ways, for example:
- increasing the number of attempts to solve each problem during the generation will increase the difficulty
- using a better model 

You can also change the diversity space by adding or removing semantic descriptor, note that the total number of unique niches is 
$N_{\text{unique niche}}=\sum_{k=1}^{s} \binom{n}{k}$ where $n$ is the number of semantic descriptor and $s$ is the maximum number of combination of semantic descriptor allowed for each niche. For our experiment we use $n=20$ semantic descriptor with $s=5$ semantic descriptor for each niche which make up to a possible of 21699 unique niche. 

---

## **Why is ACES Important?**

### **1. Outperforming Baselines**

ACES generates puzzles that are significantly more diverse and difficult than those produced by standard generative approaches or even existing human-curated benchmarks like HumanEval and BigCodeBench. For example, models that perform well on HumanEval experience a substantial drop in accuracy on ACES-generated puzzles, indicating that these new puzzles are genuinely more challenging.

### **2. Transferability and Benchmarking**

The difficulty of ACES puzzles transfers across different LLMs, making them robust benchmarks for evaluating AI coding abilities. The generated benchmarks correlate more strongly with uncontaminated datasets like LiveCodeBench than with potentially saturated ones like HumanEval.

### **3. Improving Model Training**

Finetuning LLMs on ACES-generated data leads to better performance on challenging test sets compared to training on data from other synthetic generators. As test set difficulty increases, the advantage of ACES-trained models becomes even more pronounced.

---

## **Key Results**

- **Diversity:** ACES fills up to 700 unique skill niches, far surpassing other methods.
- **Difficulty:** Generated puzzles are, on average, three times more challenging than those in existing benchmarks.
- **Scalability:** Using larger LLMs as generators and solvers further increases both diversity and difficulty.
- **Quality-Diversity Score:** ACES achieves higher QD-scores, reflecting both the breadth and challenge of its puzzle archive.

---


## **Example: ACES-Generated Puzzle**

```python
# This puzzle requires skills in array indexing, mathematical operations, and game theory.

from typing import List

def f(moves: List[List[int]], initial_state=[3, 3, 2, 2, 3, 8]) -> bool:
    """
    Check if the sequence of moves solves a variant of the Nim game.
    Each move is a tuple (i, n) indicating removing n objects from heap i.
    The goal is to reach all heaps empty, with the bot playing optimally between moves.
    """
    def bot_move():
        vals = sorted(state, reverse=True)
        i_largest = state.index(vals[0])
        state[i_largest] -= max(vals[0] - vals[1], 1)
    state = initial_state[:]
    for (i, n) in moves:
        assert 0 < n <= state[i], 'Illegal move'
        state[i] -= n
        if set(state) == {0}:
            return True
        assert any(state), 'You lost!'
        bot_move()
    return set(state) == {0}

def g(initial_state=[3, 3, 2, 2, 3, 8]):
    # Generate a sequence of optimal moves to win the game
    state = initial_state[:]
    moves = []
    def losing(h):
        xor = 0
        for i in h:
            xor ^= i
        return xor == 0
    def optimal_move():
        assert not losing(state)
        for i in range(len(state)):
            for n in range(1, state[i] + 1):
                state[i] -= n
                if losing(state):
                    moves.append((i, n))
                    return
                state[i] += n
        assert False, "Shouldn't reach here"
    while any(state):
        optimal_move()
        if max(state) == 0:
            return moves
        vals = sorted(state, reverse=True)
        i_largest = state.index(vals[0])
        state[i_largest] -= max(vals[0] - vals[1], 1)
    return moves

assert f(g())
```


## Limitations and Future Directions
- Labeling Accuracy: The skill labeling process, while generally reliable, is not perfect and can sometimes misclassify the required skills.

- Computational Cost: Evaluating puzzle difficulty is expensive, requiring multiple LLM calls per puzzle.

- Potential for Hacking: There is a theoretical risk that the generator could exploit the difficulty metric, though this was not observed in experiments.

Future work could focus on improving the robustness of skill labeling, reducing computational costs, and extending the approach to other domains beyond programming puzzles.

# Conclusion
ACES represents a significant advance in the automatic generation of programming challenges. By explicitly optimizing for both diversity and difficulty in a semantic skill space, ACES produces a new generation of benchmarks that better reflect the open-ended nature of human creativity and present a more rigorous test for AI problem-solving capabilities.


---

**Reference:**  
[`[1] NeurIPS 2024: "Generating a Diversity of Challenging Programming Puzzles with Autotelic Generative Models (ACES)"`](https://neurips.cc/virtual/2024/poster/95626) 

---
