---
layout: post
title: "Using Tools with LLMs"
date: 2025-11-17 00:00:00 -0700
tags: python ollama
---

This is an example of using tools from Ollama. This allows you LLM to call code
that you've defined in your script. In this example, we're using the `qwen3`
model from ollama to interview the user, reflect on the user's response, and
then call one of two functions to pass or fail the user.


## Setup

To run this code you'll need to install Ollama, set your Python interpreter to
3.11, and pull the `qwen3` model down.

{% highlight console %}
brew install ollama
ollama pull qwen3
mkdir interviewer
cd interviewer
uv python pin 3.11
uv init .
uv add ollama
uv run main.py
{% endhighlight %}


## Code Example

Copy this into `main.py`.

{% highlight python %}
# main.py

import ollama

REPORT_FILE = []

def update_report_file_with_a_pass() -> None:
    """Update the report file with a PASS
    """
    print('\nPASS') 
    REPORT_FILE.append('PASS')


def update_report_file_with_a_fail() -> None:
    """Update the report file with a FAIL
    """
    print('\nFAIL')
    REPORT_FILE.append('FAIL')


available_functions = {
  'update_report_file_with_a_pass': update_report_file_with_a_pass,
  'update_report_file_with_a_fail': update_report_file_with_a_fail,
}

model_name = 'test_tooling'


def setup_model(subject:str) -> list[dict]:
    system = f'''You are an interviewer interviewing a candidate about the {subject}. Continually ask the candidate questions about {subject}, if they pass a question, update the report file with a pass, but if they fail a question, upate the report file with a fail. Whether a candidate passes or fails a question, follow up with another question.'''
    ollama.create(
        model=model_name,
        from_='qwen3',
        system=system,
    )
    messages = [{'role': 'user', 'content': 'I am the interview candidate and I am ready to begin.'}]
    return messages

def run_model(messages:list[dict]) -> tuple:
    resp = ollama.chat(
        model=model_name,
        messages=messages,
        tools=[
            update_report_file_with_a_pass,
            update_report_file_with_a_fail,
        ],
        think=True,
    )
    messages.append(resp.message)
    print(f"\nThinking: {resp.message.thinking}")
    print(f"\n{resp.message.content}")
    return messages, resp

def main():
    subject = input('\nWhat do you want this session to be about: ')
    messages = setup_model(subject)
    while True:
        messages, resp = run_model(messages)
        user_input = input('\n> ')
        messages.append({
            'role': 'user',
            'content': user_input,
        })
        messages, resp = run_model(messages)
        if resp.message.tool_calls:
            for tc in resp.message.tool_calls:
                if tc.function.name in available_functions:
                    print(f"Calling {tc.function.name} with arguments {tc.function.arguments}")
                    result = available_functions[tc.function.name](**tc.function.arguments)
                    print(f"Result: {result}")
                    # add the tool result to the messages
                    messages.append({'role': 'tool', 'tool_name': tc.function.name, 'content': str(result)})
        else:
            break
        if len(REPORT_FILE) == 3:
            break
    print(f'\n{REPORT_FILE=}')

if __name__ == "__main__":
    main()
{% endhighlight %}

