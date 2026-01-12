
1. call main function
```python
if __name__ == "__main__":
		main()
```

2. create main function
	
```python
def main():

```

3. Inside main function create messages array and append system message
	import datetime in header -> from datetime import datetime
```python

messages = [

{

"role": "system",

"content": f"You are a personal assistant who helps manage tasks in Asana. The current date is: {datetime.now().date()}"

}

]
```

4. Inside main function  run infinite loop

```python
while True:


```

5. Ask for user input inside the while loop. if user gives q then quit

```python

while True:

user_input = input("Chat with AI (q to quit): ").strip()

	if user_input == 'q':
	
	break
```

6. If we observe messages array there is a property called "role". we have 3 roles in messages:
	- system - message for context setup
	- user - user input or request
	- asssistant -  agent's response
	These 3 roles signifies messages for 3 roles.
7. if not q: append user input to messages array as user message append sample ai place holder message
```python
while True:
	user_input = input("Chat with AI (q to quit): ").strip()
	if user_input = 'q'
		break
	messages.append({"role": "user","content": user_input})
	ai_response = {"role": "assistant","content": "This is a placeholderresponse from the AI."}
	print(ai_response["content"]) #sample placeholder message print
	messages.append(ai_response)
```

8. call a new funtion prompt_ai and pass these messages inside the while loop

```python
while True:
	user_input = input("Chat with AI (q to quit): ").strip()
	if user_input = 'q'
		break
	messages.append({"role": "user","content": user_input})
	ai_response = {"role": "assistant","content": "This is a placeholderresponse from the AI."}
	print(ai_response["content"]) #sample placeholder message print
	ai_response = prompt_ai(messages) # call ai - this is a custom function
	messages.append(ai_response)
)
```

9. Create prompt_ai function

```python

def prompt_ai(messages):

```

10. Implement env file:
	1. Create .env file
	2. Create parameters eg: Model
	3. add dotenv library in requirements.txt -> python-dotenv==0.13.0
	4. import dotenv in header -> from dotenv import load_dotenv ,  import os
	5. call load_dotenv in declaration area
	6. create parameter model and import .env configs

Implement Prompt ai:
	a. import openAI library - in requirements.txt ->openai==1.10.0
		b. import in header ->  from openai import OpenAI
		c. create global variable called client and assign openai to it
		d. call chat completion create method inside prompt_ai function
			which has 3 parameters:
				i. model
				ii.messages
				iii. tools
		Current code after changes
		
```python

from openai import OpenAI
from dotenv import load_dotenv
from datetime import datetime
import os

load_dotenv()

client = OpenAI()
model = os.getenv('OPENAI_MODEL', 'gemini-2.5-flash-lite')
def prompt_ai(messages):
    # First, prompt the AI with the latest user message
    completion = client.chat.completions.create(
        model=model,
        messages=messages,
        tools=
    )

def main():
    messages = [
        {
            "role": "system",
            "content": f"You are a personal assistant who helps manage tasks in Asana. The current date is: {datetime.now().date()}"
        }
    ]

    while True:
        user_input = input("Chat with AI (q to quit): ").strip()
        
        if user_input == 'q':
            break  

        messages.append({"role": "user", "content": user_input})
        ai_response = prompt_ai(messages)
```


11. Now inorder to get tools we will write a funtion get_tools with following steps
	1. Our target is to create task in asana using an agent
	2. add libraries in requirement.txt
```
asana==5.0.0
```

12. Import asana and asana.rest library to python

```
import asana
from asana.rest import ApiException
```

13. Create a function create_asana_task with following steps
	1. Add documentation sothat agent can understand the purpose
		- add parameters and structure which is passed to api
	2. create payload input for api
	3. call api
		1. parse the response using json library which is declared in header -> import json

global variables for asana api call

```

configuration = asana.Configuration()
configuration.access_token = os.getenv('ASANA_ACCESS_TOKEN', '')
api_client = asana.ApiClient(configuration)

```


```

def create_asana_task(task_name, due_on="today"):
#documentation
    """
    Creates a task in Asana given the name of the task and when it is due

    Example call:

    create_asana_task("Test Task", "2024-06-24")
    Args:
        task_name (str): The name of the task in Asana
        due_on (str): The date the task is due in the format YYYY-MM-DD. If not given, the current day is used
    Returns:
        str: The API response of adding the task to Asana or an error message if the API call threw an error
    """
#build payload    
    if due_on == "today":
        due_on = str(datetime.now().date())

    task_body = {
        "data": {
            "name": task_name,
            "due_on": due_on,
            "projects": [os.getenv("ASANA_PROJECT_ID", "")]
        }
    }

#call api
    try:
        api_response = tasks_api_instance.create_task(task_body, {})
        return json.dumps(api_response, indent=2)
    except ApiException as e:
        return f"Exception when calling TasksApi->create_task: {e}"

```


14. Now create a function get_tools that returns an array  that describes the function create_asana_task as a tool with parameters
	- type -> function
	- name->name of the function
	- description->description of tool/function
	- parameters -> parameters to the function
	- required->mandatory parameters

```
def get_tools():
    tools = [
        {
            "type": "function",
            "function": {
                "name": "create_asana_task",
                "description": "Creates a task in Asana given the name of the task and when it is due",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "task_name": {
                            "type": "string",
                            "description": "The name of the task in Asana"
                        },
                        "due_on": {
                            "type": "string",
                            "description": "The date the task is due in the format YYYY-MM-DD. If not given, the current day is used"
                        },
                    },
                    "required": ["task_name"]
                },
            },
        }
    ]

    return tools     
```


14. Handle tool calls

```
 response_message = completion.choices[0].message
    tool_calls = response_message.tool_calls

    # Second, see if the AI decided it needs to invoke a tool
    if tool_calls:
        # If the AI decided to invoke a tool, invoke it
        available_functions = {
            "create_asana_task": create_asana_task
        }

        # Add the tool request to the list of messages so the AI knows later it invoked the tool
        messages.append(response_message)

        # Next, for each tool the AI wanted to call, call it and add the tool result to the list of messages
        for tool_call in tool_calls:
            function_name = tool_call.function.name
            function_to_call = available_functions[function_name]
            function_args = json.loads(tool_call.function.arguments)
            function_response = function_to_call(**function_args)

            messages.append({
                "tool_call_id": tool_call.id,
                "role": "tool",
                "name": function_name,
                "content": function_response
            })

```


Here is the breakdown of each step:

### 1. `function_name = tool_call.function.name`

The AI doesn't execute code itself; it returns a JSON object saying, _"I want to call the function named `get_weather`."_ This line extracts that string name from the AI's response.

### 2. `function_to_call = available_functions[function_name]`

You usually have a dictionary in your code like `available_functions = {"get_weather": get_weather_function}`. This line looks up the actual, executable Python function based on the name the AI provided.

### 3. `function_args = json.loads(tool_call.function.arguments)`

The AI provides arguments as a JSON string (e.g., `'{"location": "London"}'`). This line converts that string into a Python dictionary so your code can use the data.

---

### 4. The Last Line: `function_response = function_to_call(**function_args)`

This is the most critical part. It uses a Python feature called **Dictionary Unpacking**.

- **What `**` does:** The double asterisk takes the keys and values inside the `function_args` dictionary and "unpacks" them into the function as keyword arguments.
    
- **Why it's used:** Since you don't know ahead of time which function the AI will call or what the arguments will be, you can't hardcode it like `get_weather(location="London")`.
    
- **The Execution:** It physically runs the function.


```
import asana
from asana.rest import ApiException
from openai import OpenAI
from dotenv import load_dotenv
from datetime import datetime
import json
import os

load_dotenv()

client = OpenAI()
model = os.getenv('OPENAI_MODEL', 'gpt-4o')

configuration = asana.Configuration()
configuration.access_token = os.getenv('ASANA_ACCESS_TOKEN', '')
api_client = asana.ApiClient(configuration)

tasks_api_instance = asana.TasksApi(api_client)

def create_asana_task(task_name, due_on="today"):
    """
    Creates a task in Asana given the name of the task and when it is due

    Example call:

    create_asana_task("Test Task", "2024-06-24")
    Args:
        task_name (str): The name of the task in Asana
        due_on (str): The date the task is due in the format YYYY-MM-DD. If not given, the current day is used
    Returns:
        str: The API response of adding the task to Asana or an error message if the API call threw an error
    """
    if due_on == "today":
        due_on = str(datetime.now().date())

    task_body = {
        "data": {
            "name": task_name,
            "due_on": due_on,
            "projects": [os.getenv("ASANA_PROJECT_ID", "")]
        }
    }

    try:
        api_response = tasks_api_instance.create_task(task_body, {})
        return json.dumps(api_response, indent=2)
    except ApiException as e:
        return f"Exception when calling TasksApi->create_task: {e}"

def get_tools():
    tools = [
        {
            "type": "function",
            "function": {
                "name": "create_asana_task",
                "description": "Creates a task in Asana given the name of the task and when it is due",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "task_name": {
                            "type": "string",
                            "description": "The name of the task in Asana"
                        },
                        "due_on": {
                            "type": "string",
                            "description": "The date the task is due in the format YYYY-MM-DD. If not given, the current day is used"
                        },
                    },
                    "required": ["task_name"]
                },
            },
        }
    ]

    return tools     

def prompt_ai(messages):
    # First, prompt the AI with the latest user message
    completion = client.chat.completions.create(
        model=model,
        messages=messages,
        tools=get_tools()
    )

    response_message = completion.choices[0].message
    tool_calls = response_message.tool_calls

    # Second, see if the AI decided it needs to invoke a tool
    if tool_calls:
        # If the AI decided to invoke a tool, invoke it
        available_functions = {
            "create_asana_task": create_asana_task
        }

        # Add the tool request to the list of messages so the AI knows later it invoked the tool
        messages.append(response_message)

        # Next, for each tool the AI wanted to call, call it and add the tool result to the list of messages
        for tool_call in tool_calls:
            function_name = tool_call.function.name
            function_to_call = available_functions[function_name]
            function_args = json.loads(tool_call.function.arguments)
            function_response = function_to_call(**function_args)

            messages.append({
                "tool_call_id": tool_call.id,
                "role": "tool",
                "name": function_name,
                "content": function_response
            })

        # Call the AI again so it can produce a response with the result of calling the tool(s)
        second_response = client.chat.completions.create(
            model=model,
            messages=messages,
        )

        return second_response.choices[0].message.content

    return response_message.content

def main():
    messages = [
        {
            "role": "system",
            "content": f"You are a personal assistant who helps manage tasks in Asana. The current date is: {datetime.now().date()}"
        }
    ]

    while True:
        user_input = input("Chat with AI (q to quit): ").strip()
        
        if user_input == 'q':
            break  

        messages.append({"role": "user", "content": user_input})
        ai_response = prompt_ai(messages)

        print(ai_response)
        messages.append({"role": "assistant", "content": ai_response})


if __name__ == "__main__":
    main()
```



Crux of solution:


1. Create function for api call with documentation

2. Create getTools function with appropriate api parameter documentation
3. call open ai chat completin code

```
 completion = client.chat.completions.create(
        model=model,
        messages=messages,
        tools=get_tools()
    )
```

4. Handle tool calls

```
...

function_name = tool_call.function.name

function_to_call = available_functions[function_name]

function_args = json.loads(tool_call.function.arguments)

function_response = function_to_call(**function_args)

...

```



working code:

```
import asana

from asana.rest import ApiException

from openai import OpenAI

from dotenv import load_dotenv

from datetime import datetime

import json

import os

  

load_dotenv()

  

model = os.getenv('OPENAI_MODEL', 'gpt-4o')

client = OpenAI(api_key=os.getenv('OPENAI_API_KEY', ''),

base_url="https://generativelanguage.googleapis.com/v1beta/openai/")

  

configuration = asana.Configuration()

configuration.access_token = os.getenv('ASANA_ACCESS_TOKEN', '')

api_client = asana.ApiClient(configuration)

  

tasks_api_instance = asana.TasksApi(api_client)

  

def create_asana_task(task_name, due_on="today"):

"""

Creates a task in Asana given the name of the task and when it is due

  

Example call:

  

create_asana_task("Test Task", "2024-06-24")

Args:

task_name (str): The name of the task in Asana

due_on (str): The date the task is due in the format YYYY-MM-DD. If not given, the current day is used

Returns:

str: The API response of adding the task to Asana or an error message if the API call threw an error

"""

if due_on == "today":

due_on = str(datetime.now().date())

  

task_body = {

"data": {

"name": task_name,

"due_on": due_on,

"projects": [os.getenv("ASANA_PROJECT_ID", "")]

}

}

  

try:

api_response = tasks_api_instance.create_task(task_body, {})

return json.dumps(api_response, indent=2)

except ApiException as e:

return f"Exception when calling TasksApi->create_task: {e}"

  

def get_tools():

tools = [

{

"type": "function",

"function": {

"name": "create_asana_task",

"description": "Creates a task in Asana given the name of the task and when it is due",

"parameters": {

"type": "object",

"properties": {

"task_name": {

"type": "string",

"description": "The name of the task in Asana"

},

"due_on": {

"type": "string",

"description": "The date the task is due in the format YYYY-MM-DD. If not given, the current day is used"

},

},

"required": ["task_name"]

},

},

}

]

  

return tools

  

def prompt_ai(messages):

# First, prompt the AI with the latest user message

completion = client.chat.completions.create(

model=model,

messages=messages,

tools=get_tools()

)

  

response_message = completion.choices[0].message

tool_calls = response_message.tool_calls

  

# Second, see if the AI decided it needs to invoke a tool

if tool_calls:

# If the AI decided to invoke a tool, invoke it

available_functions = {

"create_asana_task": create_asana_task

}

  

# Add the tool request to the list of messages so the AI knows later it invoked the tool

messages.append(response_message)

  

# Next, for each tool the AI wanted to call, call it and add the tool result to the list of messages

for tool_call in tool_calls:

function_name = tool_call.function.name

function_to_call = available_functions[function_name]

function_args = json.loads(tool_call.function.arguments)

function_response = function_to_call(**function_args)

  

messages.append({

"tool_call_id": tool_call.id,

"role": "tool",

"name": function_name,

"content": function_response

})

  

# Call the AI again so it can produce a response with the result of calling the tool(s)

second_response = client.chat.completions.create(

model=model,

messages=messages,

)

  

return second_response.choices[0].message.content

  

return response_message.content

  

def main():

messages = [

{

"role": "system",

"content": f"You are a personal assistant who helps manage tasks in Asana. The current date is: {datetime.now().date()}"

}

]

  

while True:

user_input = input("Chat with AI (q to quit): ").strip()

if user_input == 'q':

break

  

messages.append({"role": "user", "content": user_input})

ai_response = prompt_ai(messages)

  

print(ai_response)

messages.append({"role": "assistant", "content": ai_response})

  
  

if __name__ == "__main__":

main()
```


.env

```
# Rename this file to .env once you have filled in the below environment variables!

  

# Get your Open AI API Key by following these instructions -

https://help.openai.com/en/articles/4936850-where-do-i-find-my-openai-api-key

#OPENAI_API_KEY=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

# NOTE: Do NOT commit real keys to the repo. Replace the line below with your key during local setup.

OPENAI_API_KEY=AIzaSyDhynvqYS7lFy1GvGPUvjmL9NUQoCuvMv8

# If you want to use OAuth tokens instead set:

# GOOGLE_OAUTH_ACCESS_TOKEN=ya29.your_token_here

  

# See all Open AI models you can use here -

# https://platform.openai.com/docs/models

# A good default to go with here is gpt-4o

OPENAI_MODEL=gemini-2.5-flash

  

# Get your personal Asana access token through the developer console in Asana.

# Feel free to follow these instructions -

# https://developers.asana.com/docs/personal-access-token

ASANA_ACCESS_TOKEN=2/1199898201468690/1212250511095453:492d36b979b2b3332c770f670aa3373f

  

# The Asana project ID is in the URL when you visit a project in the Asana UI.

# If your URL is https://app.asana.com/0/123456789/1212121212, then your

# Asana project ID is 123456789

ASANA_PROJECT_ID=1212250516690523
```


requirements.txt
```
asana==5.0.0

openai==1.10.0

httpx==0.23.3

python-dotenv==0.13.0

google-auth>=2.18.0

```