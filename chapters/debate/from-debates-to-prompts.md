# From debates to prompts



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
