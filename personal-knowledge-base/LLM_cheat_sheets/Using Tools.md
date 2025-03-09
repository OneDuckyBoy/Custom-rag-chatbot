# Using Tools with GPT API - Multiple Tools Support

## Tool Function Definitions

First, define the functions that implement your tool functionalities:

```python
ticket_prices = {
    "london": "$799",
    "paris": "$899", 
    "tokyo": "$1400", 
    "berlin": "$499"
}

def get_ticket_price(destination_city):
    print(f"Tool get_ticket_price called for {destination_city}")
    city = destination_city.lower()
    return ticket_prices.get(city, "Unknown")

# Example of another tool function
def get_weather(city):
    print(f"Tool get_weather called for {city}")
    # In a real implementation, this would call a weather API
    weather_data = {
        "london": "Rainy, 15°C",
        "paris": "Sunny, 22°C",
        "tokyo": "Cloudy, 19°C",
        "berlin": "Partly cloudy, 18°C"
    }
    return weather_data.get(city.lower(), "Weather data not available")
```

## Creating Tool Schemas

Create schemas for each tool that the API can understand:

```python
price_function = {
    "name": "get_ticket_price",
    "description": "Get the price of a return ticket to the destination city. Call this whenever you need to know the ticket price, for example when a customer asks 'How much is a ticket to this city?'",
    "parameters": {
        "type": "object",
        "properties": {
            "destination_city": {
                "type": "string",
                "description": "The city that the customer wants to travel to.",
            },
        },
        "required": ["destination_city"],
        "additionalProperties": False
    }
}

weather_function = {
    "name": "get_weather",
    "description": "Get the current weather for a specific city. Call this when a customer asks about weather conditions.",
    "parameters": {
        "type": "object",
        "properties": {
            "city": {
                "type": "string",
                "description": "The city to get weather information for.",
            },
        },
        "required": ["city"],
        "additionalProperties": False
    }
}
```

## Registering Multiple Tools with the API

Create a list of all tools to provide to the AI:

```python
tools = [
    {"type": "function", "function": price_function},
    {"type": "function", "function": weather_function}
]
```

## Chat Function with Support for Multiple Tools

```python
def chat(message, history):
    messages = [{"role": "system", "content": system_message}] + history + [{"role": "user", "content": message}]
    
    # Include tools in the API call
    response = openai.chat.completions.create(model=MODEL, messages=messages, tools=tools)
    
    # Check if the model wants to use tools
    if response.choices[0].finish_reason == "tool_calls":
        assistant_message = response.choices[0].message
        messages.append(assistant_message)
        
        # Process all tool calls
        for tool_call in assistant_message.tool_calls:
            tool_response = handle_tool_call(tool_call)
            messages.append(tool_response)
        
        # Get final response after tool calls
        response = openai.chat.completions.create(model=MODEL, messages=messages)
    
    return response.choices[0].message.content
```

## Improved Tool Call Handler for Multiple Tools

```python
def handle_tool_call(tool_call):
    function_name = tool_call.function.name
    arguments = json.loads(tool_call.function.arguments)
    
    # Route to the appropriate function based on the name
    if function_name == "get_ticket_price":
        city = arguments.get('destination_city')
        result = get_ticket_price(city)
        content = json.dumps({"destination_city": city, "price": result})
    elif function_name == "get_weather":
        city = arguments.get('city')
        result = get_weather(city)
        content = json.dumps({"city": city, "weather": result})
    else:
        content = json.dumps({"error": f"Unknown function: {function_name}"})
    
    # Create the response message
    response = {
        "role": "tool",
        "content": content,
        "tool_call_id": tool_call.id
    }
    
    return response
```

## Process Flow

1. The chat function passes all available tools to the API
2. If the API decides to use one or more tools, it returns a response with `finish_reason="tool_calls"`
3. For each tool call:
    - The handler routes to the appropriate function based on the name
    - The function processes the arguments and returns a result
    - A tool response message is created and added to the conversation
4. After processing all tool calls, a final API call generates the assistant's response based on all tool results
5. The generated content is returned to the user

This implementation properly handles multiple tool calls in a single response, routing each to the appropriate function and collecting all results before generating the final response.