# Web search

Web searches matter especially for questions where the answer can change between when the language model was trained and today. For example:

- What was the weather on this date?
- What is the market cap of Google?
- Who is the president of the United States?

If you run the last question using the question-answerer, you might get an answer like:

```
The current president of the United States is Donald Trump.
```

Let's start by simply providing the list of search results as additional context before answering a question. To do this, let's write a helper function that uses [SerpAPI](https://serpapi.com/) to retrieve the search results. (You could similarly use the Bing API. In either case you need an API key.)

## Running web searches

```python
import httpx

from ice.recipe import recipe


def make_qa_prompt(context: str, question: str) -> str:
    return f"""
Background text: "{context}"

Answer the following question about the background text above:

Question: "{question}"
Answer: "
""".strip()


async def search(query: str) -> dict:
    async with httpx.AsyncClient() as client:
        params = {
        "q": query,
        "hl": "en",
        "gl": "us",
        "api_key": "e29...b4c"
        }
        response = await client.get("https://serpapi.com/search", params=params)
        return response.json()

@recipe.main
async def answer_by_search(
    *, question: str = "Who is the president of the United States?",
) -> dict:
    return await search(question)
```

Running `python search.py` returns a large JSON object:

{% code overflow="wrap" %}

```json
{
    'organic_results': [
        {
            'position': 1,
            'title': 'Presidents - The White House',
            'link': 'https://www.whitehouse.gov/about-the-white-house/presidents/',
            'displayed_link': 'https://www.whitehouse.gov › about-the-white-house',
            'about_this_result': {
                'source': {
                    'description': 'The White House is the official residence and workplace of the president of the United States. It is located at 1600 Pennsylvania Avenue NW in Washington, D.C., and has been the residence of every U.S. president since John Adams in 1800.',
     ...
}
```

{% endcode %}

## Rendering search results to prompts

We add a method to render the search results to a string:

```python
async def search(query: str) -> dict:
    async with httpx.AsyncClient() as client:
        params = {
        "q": query,
        "hl": "en",
        "gl": "us",
        "api_key": "e29...b4c"
        }
        response = await client.get("https://serpapi.com/search", params=params)
        return response.json()

def render_results(data: dict) -> str:
    if not data or not data.get("organic_results"):
        return "No results found"

    results = []
    for result in data["organic_results"]:
        title = result.get("title")
        link = result.get("link")
        snippet = result.get("snippet")
        if not title or not link or not snippet:
            continue
        results.append(f"{title}\n{link}\n{snippet}\n")

    return "\n".join(results)

@recipe.main
async def answer_by_search(
    *, question: str = "Who is the president of the United States?",
) -> str:
    results = await search(question)
    return render_results(results)
```

Now the results are much more manageable:

{% code overflow="wrap" %}

```
President of the United States - Wikipedia
https://en.wikipedia.org/wiki/President_of_the_United_States
Joe Biden is the 46th and current president of the United States, having assumed office on January 20, 2021.

Presidents, Vice Presidents, and First Ladies of the United ...
https://www.usa.gov/presidents
The 46th and current president of the United States is Joseph R. Biden, Jr. He was sworn in on January 20, 2021.

Donald J. Trump – The White House
https://trumpwhitehouse.archives.gov/people/donald-j-trump/
Donald J. Trump is the 45th President of the United States. He believes the United States has incredible potential and will go on to exceed even its remarkable ...

President of the United States - Ballotpedia
https://ballotpedia.org/President_of_the_United_States
The President of the United States (POTUS) is the head of the United States government. Article II of the U.S. Constitution laid out the requirements and roles ...

Presidents of the United States: Resource Guides
https://www.loc.gov/rr/program/bib/presidents/
List of U.S. Presidents · 1. George Washington · 2. John Adams · 3. Thomas Jefferson · 4. James Madison · 5. James Monroe · 6. John Quincy Adams · 7. Andrew Jackson · 8 ...
```

{% endcode %}

## Answering questions given search results

Now all we need to do is stick the search results into the Q\&A prompt:

```python
import httpx

from ice.recipe import recipe


def make_search_result_prompt(context: str, question: str) -> str:
    return f"""
Search results from Google: "{context}"

Answer the following question, using the search results if helpful:

Question: "{question}"
Answer: "
""".strip()


async def search(query: str) -> dict:
    async with httpx.AsyncClient() as client:
        params = {
        "q": query,
        "hl": "en",
        "gl": "us",
        "api_key": "e29...b4c"
        }
        response = await client.get("https://serpapi.com/search", params=params)
        return response.json()

def render_results(data: dict) -> str:
    if not data or not data.get("organic_results"):
        return "No results found"

    results = []
    for result in data["organic_results"]:
        title = result.get("title")
        link = result.get("link")
        snippet = result.get("snippet")
        if not title or not link or not snippet:
            continue
        results.append(f"{title}\n{link}\n{snippet}\n")

    return "\n".join(results)

@recipe.main
async def answer_by_search(
    *, question: str = "Who is the president of the United States?",
) -> str:
    results = await search(question)
    results_str = render_results(results)
    prompt = make_search_result_prompt(results_str, question)
    answer = (await recipe.agent().answer(prompt=prompt, max_tokens=100)).strip('" ')
    return answer
```

If we run this file...

```shell
python web.py
```

...we get:

{% code overflow="wrap" %}

```
Joe Biden is the 46th and current president of the United States, having assumed office on January 20, 2021.
```

{% endcode %}

Much better!

## Choosing better queries

There's still something unsatisfying--we're directly searching for the question, but it could be better to let the model control what search terms we use. This is especially true for complex questions that we don't expect to get a full answer to through Google, like:

> Based on the weather on Sep 14th 2022, how many people went do you think went to the beach in San Francisco?

Here it's probably better to just research the weather on that date using Google, not to enter the whole question. So let's introduce a `choose_query` method:

{% code overflow="wrap" %}

```python
import httpx

from ice.recipe import recipe


def make_search_result_prompt(context: str, query: str, question: str) -> str:
    return f"""
Search results from Google for the query "{query}": "{context}"

Answer the following question, using the search results if helpful:

Question: "{question}"
Answer: "
""".strip()


def make_search_query_prompt(question: str) -> str:
    return f"""
You're trying to answer the question {question}. You get to type in a search query to Google, and then you'll be shown the results. What query do you want to search for?

Query: "
""".strip('" ')


async def search(query: str) -> dict:
    async with httpx.AsyncClient() as client:
        params = {
        "q": query,
        "hl": "en",
        "gl": "us",
        "api_key": "e29...7b4c"
        }
        response = await client.get("https://serpapi.com/search", params=params)
        return response.json()


def render_results(data: dict) -> str:
    if not data or not data.get("organic_results"):
        return "No results found"

    results = []
    for result in data["organic_results"]:
        title = result.get("title")
        link = result.get("link")
        snippet = result.get("snippet")
        if not title or not link or not snippet:
            continue
        results.append(f"{title}\n{link}\n{snippet}\n")

    return "\n".join(results)


async def choose_query(squestion: str) -> str:
    prompt = make_search_query_prompt(question)
    query = (await recipe.agent().answer(prompt=prompt)).strip('" ')
    return query


@recipe.main
async def answer_by_search(
    *, question: str = "Who is the president of the United States?",
) -> str:
    query = await choose_query(question)
    results = await search(query)
    results_str = render_results(results)
    prompt = make_search_result_prompt(results_str, query, question)
    answer = (await recipe.agent().answer(prompt=prompt, multiline=False)).strip('" ')
    return answer
```

{% endcode %}

If we run our question...

{% code overflow="wrap" %}

```shell
python web.py --question "Based on the weather on Sep 12th 2022, how many people went do you think went to the beach in San Francisco?"
```

{% endcode %}

...we get:

{% code overflow="wrap" %}

```
I couldn't find an exact answer to your question, but based on the weather forecast for that day, it looks like the weather will be nice and the beaches will be busy.
```

{% endcode %}

The query chosen by the model was "beach weather san francisco september 12th 2022".

## Exercises

1. It's nice to look at search results, but often the results are in the actual web pages. Extend the recipe to add the text of the first web page.
2. Use the model to decide which of the search results to expand.
