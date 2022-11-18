---
description: Subquestions and so much more
---

# Amplification Revisited

Now that we’ve seen that language models can do many things besides just answer questions directly—

* [Check answers and reasoning](verifiers/)
* [Do a web search](tool-use/web-search.md)
* [Run a Python program](tool-use/interpreters.md)
* [Write out some reasoning steps](deduction/chain-of-thought.md)

—let’s revisit [our recursive question-answerer](amplification/recursive-amplification.md). Instead of either directly answering the question or asking more subquestions, let’s also provide the ability of doing any of the steps above.

While we’re at it, let’s take the opportunity to switch from asking subquestions in parallel to asking them sequentially, so that later subquestions can depend on the results of earlier subquestions.

This is a good opportunity to use subrecipes—we’d like to turn our work from earlier chapters into generalizable components.

## Exercise

Implement generalized amplification as outlined above.

Hint: This is a fairly small tweak to [iterative action selection](action-selection/iterative-action-selection.md).&#x20;

<details>

<summary>Get feedback on exercise solutions</summary>

If you want feedback on your exercise solutions, submit them through [this form](https://docs.google.com/forms/d/e/1FAIpQLSdNNHeQAT7GIzn4tdsVYCkrVEPMNaZmBFkZCAJdvTvLzUAnzQ/viewform). We—the team at Ought—are happy to give our quick take on whether you missed any interesting ideas.

</details>

