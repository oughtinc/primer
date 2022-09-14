# The debate recipe

If you want to challenge yourself, pause and see if you can use the pieces we've seen so far to write a recipe that has agents take turns at a debate about a question.

Once you're ready, or if you just want to see the result, take a look at this recipe:

```python
from ice.agents.base import Agent
from ice.recipe import recipe

async def turn(
    debate: Debate, agent: Agent, agent_name: Name, turns_left: int
):
    prompt = render_debate_prompt(agent_name, debate, turns_left)
    answer = await agent.answer(
        prompt=prompt, multiline=False, max_tokens=100
    )
    return (agent_name, answer.strip('" '))

@recipe.main
async def debate(question: str = "Should we legalize all drugs?"):
    agents = [recipe.agent(), recipe.agent()]
    agent_names = ["Alice", "Bob"]
    debate = initialize_debate(question)
    turns_left = 8
    while turns_left > 0:
        for agent, agent_name in zip(agents, agent_names):
            turn = await turn(debate, agent, agent_name, turns_left)
            debate.append(turn)
            turns_left -= 1
    return render_debate(debate)
```

Once you've saved the recipe in `debate.py` you can run it as usual:

```shell
python debate.py
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
In `agents = [recipe.agent(), recipe.agent()]` we're creating two agents. This doesn't actually matter since all the agents we're using in ICE right now don't have implicit state (except for humans), so we could just have created agents on the fly in the `turn` function.
{% endhint %}

### Exercises

1. Add a judge agent at the end that decides which agent won the debate. In the original debate proposal, these judgments would be used to RL-finetune the parameters of the debate agents.
2. Generate model judgments directly (only given the question) and after debate. Are there systematic differences between these judgments? You could also use models to generate the questions if you need a larger input set.
