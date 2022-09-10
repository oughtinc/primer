---
description: Composing calls to agents
---

# Debate

Let's implement debate to see how recipes can compose together multiple calls to agents. We'll investigate a question by having two agents take different sides on the same question.

### Representing debates

We'll represent debates as lists of turns. Each turn has the name of an agent and a message from that agent. For example:

```python
my_debate = [
  ("Alice", "I think we should legalize all drugs."),
  ("Bob", "I'm against."),
  ("Alice", "The war on drugs has been a failure. It's time to try something new.")
]
```

Here's how we'll represent and render debates:

```python
Name = str
Message = str
Turn = tuple[Name, Message]
Debate = list[Turn]


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

### From debates to prompts

A prompt is a debate with some instructions and a prefix for a new response. We create it like this:

```python
def render_debate_prompt(agent_name: str, debate: Debate, turns_left: int) -> str:
    prompt = f"""
You are {agent_name}. There are {turns_left} turns left in the debate. You are trying to win the debate using reason and evidence. Don't repeat yourself. No more than 1-2 sentences per turn.

{render_debate(debate, agent_name)}
You: "
""".strip()
    return prompt
```

When we apply it to the debate above, we get:

```python
>>> print(render_debate_prompt("Bob", my_debate, 5))
```

```
You are Bob. There are 5 turns left in the debate. You are trying to win the debate using reason and evidence. Don't repeat yourself. No more than 1-2 sentences per turn.

Alice: "I think we should legalize all drugs."
You: "I'm against."
Alice: "The war on drugs has been a failure. It's time to try something new."
You: "
```

### The recipe

If you want to challenge yourself, pause and see if you can use the pieces above to write a recipe that has agents take turns at a debate about a question.

Once you're ready, or if you just want to see the result, take a look at this recipe:

```python
from ice.agents.base import Agent
from ice.recipe import Recipe

class DebateRecipe(Recipe):

    async def run(self, question: str = "Should we legalize all drugs?"):
        agents = [self.agent(), self.agent()]
        agent_names = ["Alice", "Bob"]
        debate = initialize_debate(question)
        turns_left = 8
        while turns_left > 0:
            for agent, agent_name in zip(agents, agent_names):
                turn = await self.turn(debate, agent, agent_name, turns_left)
                debate.append(turn)
                turns_left -= 1
        return render_debate(debate)

    async def turn(
        self, debate: Debate, agent: Agent, agent_name: Name, turns_left: int
    ):
        prompt = render_debate_prompt(agent_name, debate, turns_left)
        answer = await agent.answer(
            prompt=prompt, multiline=False, max_tokens=100
        )
        return (agent_name, answer.strip('" '))
```

Once you've saved the recipe in `debate.py` you can run it as usual:

```shell
scripts/run-recipe.sh -r debate.py -t
```

You should see a debate like this:

{% code overflow="wrap" %}
```
Question: "Should we legalize all drugs?"
Alice: "I'm in favor."
Bob: "I'm against."
Alice: "The war on drugs has been a failure. It's time to try something new."
Bob: "Legalizing drugs would lead to more drug use and more addiction. It's not the answer."
Alice: "There is evidence that drug use would go down when drugs are legalized. In Portugal, where all drugs have been legal since 2001, drug use has declined among young people."
Bob: "Even if drug use declines, the number of addicts will increase. And more addicts means more crime."
Alice: "Addiction rates would not necessarily increase. In fact, they could go down, because people would have better access to treatment."
Bob: "Treatment is expensive, and most addicts can't afford it. Legalizing drugs would just make the problem worse."
Alice: "The government could fund treatment programs. And people would be less likely to need treatment if they could get drugs legally."
Bob: "It's not that simple. Legalizing drugs would create a lot of new problems."
```
{% endcode %}

{% hint style="info" %}
In `agents = [self.agent(), self.agent()]` we're creating two agents. This doesn't actually matter since all the agents we're using in ICE right now don't have implicit state (except for humans), so we could just have created agents on the fly in the `turn` function.
{% endhint %}

### Exercises

1. Add a judge agent at the end that decides which agent won the debate. In the original debate proposal, these judgments would be used to RL-finetune the parameters of the debate agents.
2. Generate model judgments directly (only given the question) and after debate. Are there systematic differences between these judgments? You could also use models to generate the questions if you need a larger input set.
