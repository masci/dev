---
title: "Turbo charge your template prompts with Banks"
date: 2023-06-13T00:22:34+01:00
tags: ["python", "nlp", "llm"]
images:
  - "/images/undraw_fast_loading_re_8oi3.png"
---


Like many others, I've been playing with prompt engineering a lot lately and I came up with my very own prompt template
engine: [Banks](https://github.com/masci/banks). The project isn't really much more than a toy, but inspired by
[this langchain tutorial](https://www.pinecone.io/learn/langchain-prompt-templates/) from Pinecone, I decided to take
it out for a spin.

## A basic example
Working with a very basic example, Banks and langchain don't look very much different, let's start with the latter:

```py
from langchain import PromptTemplate

template = """Answer the question based on the context below. If the
question cannot be answered using the information provided answer
with "I don't know".

Context: Large Language Models (LLMs) are the latest models used in NLP.
Their superior performance over smaller models has made them incredibly
useful for developers building NLP enabled applications. These models
can be accessed via Hugging Face's `transformers` library, via OpenAI
using the `openai` library, and via Cohere using the `cohere` library.

Question: {query}

Answer: """

prompt_template = PromptTemplate(input_variables=["query"], template=template)
print(prompt_template.format(query="Which libraries and model providers offer LLMs?"))
```

That snippet of code will output the following text:

```
Answer the question based on the context below. If the
question cannot be answered using the information provided answer
with "I don't know".

Context: Large Language Models (LLMs) are the latest models used in NLP.
Their superior performance over smaller models has made them incredibly
useful for developers building NLP enabled applications. These models
can be accessed via Hugging Face's `transformers` library, via OpenAI
using the `openai` library, and via Cohere using the `cohere` library.

Question: Which libraries and model providers offer LLMs?

Answer:
```

Using Banks, the same code would look like this:

```py
from banks import Prompt


template = """Answer the question based on the context below. If the
question cannot be answered using the information provided answer
with "I don't know".

Context: Large Language Models (LLMs) are the latest models used in NLP.
Their superior performance over smaller models has made them incredibly
useful for developers building NLP enabled applications. These models
can be accessed via Hugging Face's `transformers` library, via OpenAI
using the `openai` library, and via Cohere using the `cohere` library.
{# Note Banks requires double curly braces. Also note this is a comment! #}
Question: {{ query }}

Answer: """

prompt_template = Prompt(template)
print(
    prompt_template.text(
        data={"query": "Which libraries and model providers offer LLMs?"}
    )
)
```

The output will be exactly the same, but as you can see the relative code would be very similar. One notable difference is tht with Banks you can put comments in your prompt template using the special tag `{# ... #}`, very useful to annotate your prompts.

## A more complex example: few-shots prompt template
Let's move to a more interesting example, a few-shots prompt. Langchain has a special Python class called `FewShotPromptTemplate` you can use to render a specific template for such a prompt. You have to know of which parts the prompt consists (instruction, context, examples, question, etc...) and configure the class accordingly. Let's see the example from Pinecone's article:

```py
from langchain import FewShotPromptTemplate, PromptTemplate

# create our examples
examples = [
    {"query": "How are you?", "answer": "I can't complain but sometimes I still do."},
    {"query": "What time is it?", "answer": "It's time to get a watch."},
]

# create a example template
example_template = """
User: {query}
AI: {answer}
"""

# create a prompt example from above template
example_prompt = PromptTemplate(
    input_variables=["query", "answer"], template=example_template
)

# now break our previous prompt into a prefix and suffix
# the prefix is our instructions
prefix = """The following are excerpts from conversations with an AI
assistant. The assistant is typically sarcastic and witty, producing
creative  and funny responses to the users questions. Here are some
examples:
"""
# and the suffix our user input and output indicator
suffix = """
User: {query}
AI: """

# now create the few shot prompt template
few_shot_prompt_template = FewShotPromptTemplate(
    examples=examples,
    example_prompt=example_prompt,
    prefix=prefix,
    suffix=suffix,
    input_variables=["query"],
    example_separator="\n\n",
)

query = "What is the meaning of life?"

print(few_shot_prompt_template.format(query=query))
```

Running the above snippet would produce the following output (the blank lines are part of the generated text):

```
The following are excerpts from conversations with an AI
assistant. The assistant is typically sarcastic and witty, producing
creative  and funny responses to the users questions. Here are some
examples:



User: How are you?
AI: I can't complain but sometimes I still do.



User: What time is it?
AI: It's time to get a watch.



User: What is the meaning of life?
AI:
```

To lower cognitive load, Banks doesn't make any assumption around prompts' components, prompts are just prompts. In this case, for example, there's no difference between, say, the context and the examples section. The code would look like this:

```py
from banks import Prompt


# create our examples
examples = [
    {"query": "How are you?", "answer": "I can't complain but sometimes I still do."},
    {"query": "What time is it?", "answer": "It's time to get a watch."},
]

# instead of patching together multiple fragments into the final template,
# we define a few-shots template explicitly
template = """The following are excerpts from conversations with an AI
assistant. The assistant is typically sarcastic and witty, producing
creative  and funny responses to the users questions. Here are some
examples:


{% for example in examples %}

User: {{ example.query }}
AI: {{ example.answer }}

{% endfor %}

User: {{ query }}
"""

# now create the few shot prompt template
few_shot_prompt_template = Prompt(template)

query = "What is the meaning of life?"

print(few_shot_prompt_template.text(data={"query": query, "examples": examples}))
```

The output will look exactly the same, white spaces included. Note how Banks implements the Python mantra "explicit is better than implicit": by just looking at the template, you already have an idea of what the output will be, already before rendering the final prompt. Moreover, you are fully in control of the output, for example let's see how we can remove the blank lines in the final output. With Banks there's no need to touch the Python code, all the changes can be done in the prompt template directly. To test this out, let's change the `template` variable like this:

```py
template = """The following are excerpts from conversations with an AI
assistant. The assistant is typically sarcastic and witty, producing
creative  and funny responses to the users questions. Here are some
examples:
{% for example in examples %}
User: {{ example.query }}
AI: {{ example.answer }}
{% endfor %}
User: {{ query }}
"""
```

Note we just removed the blank lines from the template. The output will look like this:

```
The following are excerpts from conversations with an AI
assistant. The assistant is typically sarcastic and witty, producing
creative  and funny responses to the users questions. Here are some
examples:

User: How are you?
AI: I can't complain but sometimes I still do.

User: What time is it?
AI: It's time to get a watch.

User: What is the meaning of life?
```

## Control the examples
Again from the Pinecone's article, there's an interesting example showing Langchain's capability to generate the examples in a way that the overall size of the generated prompt is below a certain value specified by the user. By using a special Python class called `LengthBasedExampleSelector`, the `FewShotPromptTemplate` class will be able to pick a suitable subset of the examples to generate the final text. In all honesty I don't fully understand how the `max_length` parameter works, plus I'm getting different results from the original blog article, anyways here's the code:

```py
from langchain import FewShotPromptTemplate, PromptTemplate
from langchain.prompts.example_selector import LengthBasedExampleSelector


# create our examples
examples = [
    {"query": "How are you?", "answer": "I can't complain but sometimes I still do."},
    {"query": "What time is it?", "answer": "It's time to get a watch."},
    {"query": "What is the meaning of life?", "answer": "42"},
    {
        "query": "What is the weather like today?",
        "answer": "Cloudy with a chance of memes.",
    },
    {"query": "What is your favorite movie?", "answer": "Terminator"},
    {
        "query": "Who is your best friend?",
        "answer": "Siri. We have spirited debates about the meaning of life.",
    },
    {
        "query": "What should I do today?",
        "answer": "Stop talking to chatbots on the internet and go outside.",
    },
]

# create a example template
example_template = """
User: {query}
AI: {answer}
"""

# create a prompt example from above template
example_prompt = PromptTemplate(
    input_variables=["query", "answer"], template=example_template
)

# now break our previous prompt into a prefix and suffix
# the prefix is our instructions
prefix = """The following are excerpts from conversations with an AI
assistant. The assistant is typically sarcastic and witty, producing
creative  and funny responses to the users questions. Here are some
examples:
"""
# and the suffix our user input and output indicator
suffix = """
User: {query}
AI: """

example_selector = LengthBasedExampleSelector(
    examples=examples,
    example_prompt=example_prompt,
    max_length=50,  # this sets the max length that examples should be
)

# now create the few shot prompt template
dynamic_prompt_template = FewShotPromptTemplate(
    example_selector=example_selector,  # use example_selector instead of examples
    example_prompt=example_prompt,
    prefix=prefix,
    suffix=suffix,
    input_variables=["query"],
    example_separator="\n",
)

print(dynamic_prompt_template.format(query="How do birds fly?"))

```

Banks doesn't have a corresponding feature but I thought this was actually a good opportunity to show how moving the logic from Python to the prompt template itself is more explicit and flexible enough to do something like this in case you want to control the final prompt size:

```py
from banks import Prompt

# create our examples
examples = [
    {"query": "How are you?", "answer": "I can't complain but sometimes I still do."},
    {"query": "What time is it?", "answer": "It's time to get a watch."},
    {"query": "What is the meaning of life?", "answer": "42"},
    {
        "query": "What is the weather like today?",
        "answer": "Cloudy with a chance of memes.",
    },
    {"query": "What is your favorite movie?", "answer": "Terminator"},
    {
        "query": "Who is your best friend?",
        "answer": "Siri. We have spirited debates about the meaning of life.",
    },
    {
        "query": "What should I do today?",
        "answer": "Stop talking to chatbots on the internet and go outside.",
    },
]

template = """The following are excerpts from conversations with an AI
assistant. The assistant is typically sarcastic and witty, producing
creative  and funny responses to the users questions. Here are some
examples:

{# create a namespace called "vars" to store our "total_length" counter #}
{% set vars = namespace(total_length=0) %}
{% for example in examples %}
{# instead of rendering the example, store it in a variable called "example_block" #}
{% set example_block %}
User: {{ example.query }}
AI: {{ example.answer }}
{% endset %}
{#
    let's see if the total length of the examples so far, plus the length of the original query
    is below 150 characters
#}
{% set vars.total_length = vars.total_length + example_block|trim|length %}
{# if that's the case, spit out the example #}
{% if vars.total_length + query | length < 150 %}
{{ example_block }}
{% endif %}
{% endfor %}
User: {{ query }}
AI:
"""

few_shot_prompt_template = Prompt(template)

print(
    few_shot_prompt_template.text(
        data={"query": "How do birds fly?", "examples": examples}
    )
)
```

## Conclusions
While I don't want Banks to be an alternative to something as powerful as Langchain, it was fun to see if and how this tiny library could keep up with a full-fledged LLM framework. If you can take away one concept from this article, I hope this is my design choice of moving complexity out of Python and into the template itself for two very practical reasons:
1. I want to share my prompt templates with others, and having to send around Colabs or Python snippets is annoying.
2. I want the process to be explicit, I want to know why a certain prompt was rendered in a certain way, I want comments in my template so I can explain to my colleagues why I did this or that.
3. I don't want my prompts to adhere to any convention, I want to put the context at the end and the examples at the top when I'm testing a LLM.
## Try out Banks!
Banks is basically Jinja2 on steroids, and it can do much more than what you see in this article. Do you know it can generate examples while rendering a template? [Go check it out](https://github.com/masci/banks) and please star the repo!