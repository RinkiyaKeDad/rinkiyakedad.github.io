---
title: "You Don’t Need To Pay for the OpenAI API During Development"
read_time: true
date: 2024-07-10T00:00:00+05:30
tags:
  - ai
  - python
---

If you're developing or thinking of creating an app that leverages the OpenAI API but don't want to put down your credit card until you're sure you have something, I have good news for you. In this article I’m going to show you how you can create a backend API which is compatiable with the OpenAI API. However, when running this API locally during development, we will use the [Llama3](https://llama.meta.com/llama3/) model **running locally on your laptop**. When you want to use this in production, all you need to do is change the environment variables, and everything else will remain the same :)

## Why would you want to do this?

[Ollama](https://ollama.com/), for those of you who don’t know, is one of the simplest ways to get started with running large language models locally on your machine. Earlier this year, they announced compatibility with the OpenAI [Chat Completions API](https://github.com/ollama/ollama/blob/main/docs/openai.md). This means you can keep writing code for your application as if you were developing using the OpenAI API, but when running things locally, you can use a language model provided by Ollama running locally on your machine.

This is very useful because it allows you to get started easily with developing AI applications without creating an account or paying for the OpenAI API. This means that during development, your API costs would be zero, so you can iterate freely and as much as you want without worrying about it. When you’re ready to take your application to production, you don’t need to change any of your code because it’s already compatible with the OpenAI. All you need to do is swap out your API keys and endpoints, and you’re ready to go!

Let’s see this in action in this blog :)

## Setting up the project

Create a new folder, and inside it, create a new virtual environment by running:

```bash
python -m venv openai-api
```

Activate the virtual environment by running (on MacOS or Unix):

```bash
source openai-env/bin/activate
```

and (on Windows):

```bash
openai-env\Scripts\activate
```

Install the dependencies we’ll need for this project:

```bash
pip install --upgrade openai Flask python-dotenv
```

We need `openai` to interact with the OpenAI/Ollama API, `Flask` to create our backend API server, and `python-dotenv` to automatically load the `.env` file we will create, which will allow us to easily swap environment variables.

Once you have all this, we’re ready to move to the development phase.

## Writing the code for our server

Here’s what the code for a simple backend that talks to the OpenAI API could look like:

```python
import os
from flask import Flask, request, jsonify
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv()  # take environment variables from .env.

app = Flask(__name__)

client = OpenAI(
    api_key=os.environ.get("OPENAI_API_KEY"),
    base_url=os.environ.get("LLM_ENDPOINT")
)

@app.route("/api/chat", methods=["POST"])
def chat():
    try:
        data = request.get_json()
        input_message = data["input"]

        # Create chat completion
        response = client.chat.completions.create(
            model=os.environ.get("MODEL"),
            messages=[
                {"role": "system", "content": "You are an AI chatbot who specializes in writing poems."},
                {"role": "user", "content": input_message}
            ]
        )

        output = response.choices[0].message.content
        return jsonify({"output": output})
    except Exception as e:
        print("Error:", str(e))
        return jsonify({"error": "Internal Server Error"}), 500

if __name__ == "__main__":
    port = int(os.getenv("PORT", 3000))
    app.run(host="0.0.0.0", port=port)
```

The code should be fairly easy to understand if you’ve ever worked with the OpenAI API before. I want to keep the focus of this article on showing you how to use Ollama to develop AI apps, so I won’t go into detail about this code. You can refer to the [OpenAI docs](https://platform.openai.com/docs/quickstart/quickstart-language-selection) for that.

## Setting the right environment variables

f you noticed in the above code, we used three environment variables. These make all the difference in enabling you to use the Ollama API during development and easily switching to OpenAI when you’re done building an MVP.

Create a `.env` file and put the following in it:

```
# for development

OPENAI_API_KEY=doesntmatteryaar
LLM_ENDPOINT="http://localhost:11434/v1"
MODEL=llama3

# for production

# OPENAI_API_KEY=youropenapikey
# LLM_ENDPOINT="https://api.openai.com/v1"
# MODEL=gpt-3.5-turbo
```

The production section should be obvious if you’ve worked with the OpenAI API before. In the development section, we provide a dummy key. Unlike OpenAI, the Ollama API doesn’t care what key you provide; it just needs there to be some value. The endpoint we provide is the default one where Ollama serves. For the model, you can pick any of the ones Ollama supports. Now that we have this, we’re ready to get our API up and running!

## Running our code

Before we run the code we wrote, we need to start our Ollama server. Installing Ollama is very simple, and you can find the instructions for that [on their website](https://ollama.com/download). Once you have it installed and have pulled an LLM model (in this case, `llama3`), run the following command to start the server:

```bash
ollama serve
```

Once this is done, we’re all ready to go. Start the Python server by running:

```bash
python server.py
```

Now our server, which talks with the LLM model, should be up and running. We can test that by running the following `curl` command:

```bash
curl -X POST http://localhost:3000/api/chat \
     -H "Content-Type: application/json" \
     -d '{"input": "Compose a poem on LLMs."}'
```

And ta-da! This is the response I got back if you want to have a chuckle as well:

```json
{"output":"In silicon halls, where data reigns,
Linguistic Lightweights weave their refrains.
With weights that dance upon the scales,
They grasp the nuances of human gales.

Their language flows, both pure and bright,
A tapestry of meaning, woven with might.
From words to wisdom, they unfold the page,
A lexicon of knowledge, in digital age.

Like sages of old, they hold the key,
To secrets hidden, yet to be free.
In realms of reason, they guide our way,
Through mazes of complexity, night and day.


With each new prompt, a challenge laid,
They summon forth, their cognitive shade.
A whirlwind of words, in rapid fire,
As meaning blooms, like petals on a pyre.


Linguistic Lightweights, oh so fine,
Weaving worlds from code, and data's shrine.
In harmony with humans, they entwine,
The threads of thought, where understanding is divine.


Their poetic grasp, on language true,
A reflection of our own, anew.
For in their digital heartbeats lies,
A promise of progress, for humanity's sighs."}
```

Now, if you swap the environment variables with the OpenAI ones and restart your server, you should still see a response. Except this time, you’d be pinging the OpenAI API instead of the locally running Llama3 model via Ollama. So you can easily continue developing your applications locally without paying for any API costs, and when ready to push your app to production, easily switch to the OpenAI API :)

## Conclusion

I hope this was useful in making it easier for you to get started developing AI powered applications without incurring any costs for the model during development. If this was helpful, subscribe to my newsletter to get notified whenever I post more such content! :)

{{< rawhtml >}}

<iframe
scrolling="no"
style="width:100%!important;height:220px;border:1px #ccc solid !important"
src="https://buttondown.email/arsh?as_embed=true"
></iframe><br /><br />
{{< /rawhtml >}}

Thanks for reading!
