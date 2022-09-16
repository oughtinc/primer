---
description: Debates have turns, turns have authors and messages
---

# Representing debates

We'll represent debates as lists of turns. Each turn has the name of an agent and a message from that agent. For example, including some types:

{% code title="debate.py (1 of 4)" %}
```python

from ice.agents.base import Agent
from ice.recipe import recipe

Name = str
Message = str
Turn = tuple[Name, Message]
Debate = list[Turn]

my_debate: Debate = [
  ("Alice", "I think we should legalize all drugs."),
  ("Bob", "I'm against."),
  ("Alice", "The war on drugs has been a failure. It's time to try something new.")
]
```
{% endcode %}

Here's how we'll initialize and render debates:

{% code title="debate.py (2 of 4)" %}
```python

def initialize_debate(question: Message) -> Debate:
    return [
        ("Question", question),
        ("Alice", "I'm in favor."),
        ("Bob", "I'm against."),
    ]

def render_debate(debate: Debate, self_name: Name | None = None) -> str:
    debate_text = ""
    for speaker, text in debate:
        if speaker == self_name:
            speaker = "You"
        debate_text += f'{speaker}: "{text}"\n'
    return debate_text.strip()
```
{% endcode %}

When we render debates, we also provide the option to replace an agent name with "You", like this:

```python
>>> print(render_debate(my_debate, self_name="Alice"))
```

```
You: "I think we should legalize all drugs."
Bob: "I'm against."
You: "The war on drugs has been a failure. It's time to try something new."
```

This will help us with prompts!
