---
description: Going back and forth between agents
---

# The debate recipe

If you want to challenge yourself, pause and see if you can use the pieces we've seen so far to write a recipe that has agents take turns at a debate about a question.

Once you're ready, or if you just want to see the result, take a look at this recipe:

{% code title="debate/recipe.py" %}
```python
from ice.agents.base import Agent
from ice.recipe import recipe
from ice.recipes.primer.debate.prompt import *


async def turn(debate: Debate, agent: Agent, agent_name: Name, turns_left: int):
    prompt = render_debate_prompt(agent_name, debate, turns_left)
    answer = await agent.complete(prompt=prompt, stop="\n")
    return (agent_name, answer.strip('" '))


async def debate(question: str = "Should we legalize all drugs?"):
    agents = [recipe.agent(), recipe.agent()]
    agent_names = ["Alice", "Bob"]
    debate = initialize_debate(question)
    turns_left = 8
    while turns_left > 0:
        for agent, agent_name in zip(agents, agent_names):
            response = await turn(debate, agent, agent_name, turns_left)
            debate.append(response)
            turns_left -= 1
    return render_debate(debate)


recipe.main(debate)
```
{% endcode %}

Once you've saved the recipe you can run it as usual:

```shell
python debate/recipe.py
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

The trace looks like this:

<figure><img src="../../.gitbook/assets/Screenshot kwP5dN7n@2x.png" alt=""><figcaption><p>Execution trace (<a href="https://ice.ought.org/traces/01GE0VA96FWT7SSQNXD6CQH4BT">view online</a>)</p></figcaption></figure>

{% hint style="info" %}
In `agents = [recipe.agent(), recipe.agent()]` we're creating two agents. This doesn't actually matter since all the agents we're using in ICE right now don't have implicit state (except for humans), so we could just have created agents on the fly in the `turn` function.
{% endhint %}

## Exercises

1. Add a judge agent at the end that decides which agent won the debate. In the original debate proposal, these judgments would be used to RL-finetune the parameters of the debate agents.
2. Generate model judgments directly (only given the question) and after debate. Are there systematic differences between these judgments? You could also use models to generate the questions if you need a larger input set.

<details>

<summary>Get feedback on exercise solutions</summary>

If you want feedback on your exercise solutions, submit them through [this form](https://docs.google.com/forms/d/e/1FAIpQLSdNNHeQAT7GIzn4tdsVYCkrVEPMNaZmBFkZCAJdvTvLzUAnzQ/viewform). We—the team at Ought—are happy to give our quick take on whether you missed any interesting ideas.

</details>

## References

1. Irving, Geoffrey, Paul Christiano, and Dario Amodei. [AI Safety via Debate](https://arxiv.org/abs/1805.00899). May 2, 2018.
