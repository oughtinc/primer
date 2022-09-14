# Loading paper text

ICE has built-in functionality for parsing and loading papers, and includes some example papers in its `papers` folder. Here's a minimal recipe that loads a paper and prints out the first paragraph (often the abstract):

```python
from ice.recipe import recipe
from ice.paper import Paper

@recipe.main
async def answer_for_paper(*, paper: Paper):
    return paper.paragraphs[0]
```

If you have this recipe as `paperqa.py`, you can run it as follows, providing the paper as a keyword argument:

```shell
python paperqa.py --paper papers/keenan-2018.pdf
```

You'll see a result like this:

{% code overflow="wrap" %}

```python
Paragraph(sentences=['We hypothesized that mass distribution of a broad-spectrum antibiotic agent to preschool children would reduce mortality in areas of sub-Saharan Africa that are currently far from meeting the Sustainable Development Goals of the United Nations.'], sections=[Section(title='Abstract', number=None)], section_type='abstract')
```

{% endcode %}

Note that:

- Papers are represented as lists of paragraphs
- Paragraphs are represented as lists of sentences
- Each paragraph has information about which section it's from

Try it with your own PDF papers!
