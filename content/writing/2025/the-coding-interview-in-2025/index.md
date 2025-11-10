---
title: "The coding interview in 2025"
date: 2025-09-05T8:31:38+02:00
tags: ["management", "hiring", "ai"]
url: "the-coding-interview-in-2025"
---

Over my career, I've conducted several hundreds of interviews, written coding exercises, and even designed entire hiring
pipelines. I've also screwed up, many many times.

On the other side of the fence, I use AI coding tools frequently but not daily. I'm very aware of when they're a
productivity boost and when they're a waste of time, money, and resources. Coding assistants are evolving incredibly
fast, and I know first-hand how staying current with them takes time and commitment. I'm also experiencing how using
them to write more and more of my code, I tend to forget the finer details of languages and libraries. While I think
it's unacceptable for a software engineer not being able to use generative AI tools on the job in 2025,
including this aspect into the hiring process is tricky.

From what I see and am told, companies today use variations of these techniques for technical screening interviews:
some prohibit AI completely, doing their best to prevent candidates from using it by asking them to share their full
screen and narrate their thought process. Others explicitly permit AI, sometimes using interview tools with integrated
assistants so the interviewer can watch the interaction unfold, or simply asking candidates to be transparent if they
use these tools.

I believe that prohibiting AI in interviews is a flawed approach. First, getting used to AI tools can actually
[make you slower](https://arxiv.org/abs/2506.08872) when they are taken away, which means you might lose strong
candidates who simply lack practice coding without an assistant. Second, you want to hire someone who either knows how
to use coding assistants, or at least has an opinion about why they shouldn't be used. And finally, let's be honest,
cheaters gonna cheat. But even when AI is permitted, during the interview it is often seen as a tool, and there's
little to no focus on understanding how the candidate incorporated this tool into their engineering work.

All things considered, I think the best approach is to **go all in on generative AI and see how the candidate navigates its quirks.**

## The interview blueprint

The best problems for this kind of exercise are those that have an easy but sub-optimal solution that can be
refined into a less obvious ideal solution.

To make this post more concrete, I'm going to show an exercise we used during phone screenings when I worked at
[Datadog](https://www.datadoghq.com/). But if you decide to use this blueprint for your own interviews, I highly recommend
you create an original problem. For the record, we stopped using this particular problem around 2016 after it leaked on
Glassdoor, so I'm comfortable sharing it.

This blueprint should take no more than 30 minutes.

### Step 1: illustrate the problem

Datadog has its own query language and grammar, and supports templated variables in monitors descriptions for instance.
We'd like to write a generic `is_balanced` function that would tell if a given string is balanced. We only want to
support `(` and `)` for now. Parenthesis could be nested.

```python
def is_balanced(word):
    pass

# Test cases:

print is_balanced('Warning: load is high on (host.ip)'), True
print is_balanced('((hello)(world))'), True
print is_balanced('my (monitor))(message'), False
```

Start by asking if the candidate needs any clarification before moving to the next step.

### Step 2: get to a working solution

Any good coding assistant should be able to provide a working solution very quickly. What you're looking for here is
how the candidate interacts with the AI tools. The problem is simple enough that some candidates may opt to code it
manually, which is perfectly fine and a good sign of a strong performer. If they do, you can shift the AI-coding
evaluation later, during one of the subsequent steps.

In this case, chances are that the keywords "balanced parentheses" will skew both the human and the coding agent
towards a stack-based solution:

```python
def is_balanced(word: str) -> bool:
    stack = []
    for char in word:
        if char == '(':
            stack.append(char)
        elif char == ')':
            if not stack:
                return False
            stack.pop()
    return not stack
```

What to look for:
- **Evaluate the prompt they're using**. Many will just paste the entire problem into the assistant's prompt, but pay attention to any original or creative approaches.
- **Is the candidate correcting the code provided?** The provided problem description intentionally lacks type hints. Assistants usually understand typed code better, so fixing this upfront is a good practice. Note if the candidate thinks to do this.
- **How does the candidate handle the test cases?** The tests are in pseudo-code and not ready to be run with a test runner (the `print` statement is there only to celebrate the good old times :). Check if the candidate uses them and how. Translating them into working Python might be a more effective strategy.

### Step 3: discuss the first solution

This step is all about seeing if the candidate can understand the code produced by the assistant and if they are able
to question its decisions. Start with these follow-up questions:
- Ask them about the time and space complexity of the solution. This will force them to walk through the generated code to understand it.
- If the code produced is stack-based, ask them if there's a better solution with lower space complexity.
- If the code doesn't use a stack, ask them if a stack would be better or worse.

What to look for:
- **Can the candidate refine the assistant's output?** Look for examples like adding or improving typing, or making variable names clearer.
- **Is the candidate able to iterate on the output?** See if the candidate is able to optimize the code and how.

### Step 4: produce the optimal solution

The final step is to arrive at the ideal, optimal solution. Since we want to have a more responsible use of memory, we
can replace the stack with a simple counter. A coding agent would generate this code if explicitly asked to refine the
stack-based solution in order to consume less memory. Obviouly, the same happens if the candidate comes up with this
idea on their own and they prompt the agent with a simple _"rewrite the function replacing the stack with a counter"_:

```python
def is_balanced(word: str) -> bool:
    count = 0
    for char in word:
        if char == '(':
            count += 1
        elif char == ')':
            if count == 0:
                return False
            count -= 1
    return count == 0
```

What to look for:
- **Does the candidate know what to optimize next?** Or are they just going to rely on a zero-shot prompt to the assistant?
- **Does the candidate own the generated output?** See if the candidate can discuss trade-offs between performance and future-proofing.

## Conclusion

Embracing generative coding in the hiring process isn't just about passively allowing the use a new technology; it's
about finding out how a candidate adapted to these new tools and getting a clearer picture of their real-world skills.
You’ll see if they can ask the right questions, critically evaluate a generated solution and refine it into an elegant,
optimal product, which is something coding assistants have yet to prove they can do.
