
1. Update messages array - instead of json use SystemMessage, HumanMessage from declaratioin
```python

from langchain_core.tools import tool

from langchain_openai import ChatOpenAI

from langchain_anthropic import ChatAnthropic

from langchain_core.messages import SystemMessage, HumanMessage, ToolMessage
```

2. Add @tools before function and take of getTools function

![[Pasted image 20260112052446.png]]


3. in prompt_ai function :
	1. create array for tools
	2. create chatbot by calling imported class ChatOpenAI
	3. bind tools to object
```
tools = [create_asana_task]

#asana_chatbot = ChatOpenAI(model_name=model, openai_api_key=os.getenv('OPENAI_API_KEY', ''),base_url="https://generativelanguage.googleapis.com/v1beta/openai/") if "gpt" in model.lower() else ChatAnthropic(model_name=model, anthropic_api_key=os.getenv('ANTHROPIC_API_KEY', ''))

#added gemini base_url model and key

asana_chatbot = ChatOpenAI(model=model,

openai_api_key=os.getenv('OPENAI_API_KEY', ''),

base_url="https://generativelanguage.googleapis.com/v1beta/openai/")

asana_chatbot_with_tools = asana_chatbot.bind_tools(tools)
```


