# LLM API Examples


---

## Conda Environment Activation

```bash
conda activate llms
```

---

## OpenAI Chat Completion Examples

### Basic Chat Completion

```python
response = openai.chat.completions.create(
    model=MODEL,
    messages=[
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": user_prompt}
    ],
)
result = response.choices[0].message.content

display(Markdown(result))
```

### JSON Response Format

```python
response = openai.chat.completions.create(
    model=MODEL,
    messages=[
        {"role": "system", "content": link_system_prompt},
        {"role": "user", "content": user_prompt}
    ],
    response_format={"type": "json_object"}
)
result = response.choices[0].message.content
return json.loads(result)
```

### Streaming Response

```python
stream = openai.chat.completions.create(
    model=MODEL,
    messages=[
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": get_brochure_user_prompt(company_name, url)}
    ],
    stream=True
)

response = ""
display_handle = display(Markdown(""), display_id=True)

for chunk in stream:
    response += chunk.choices[0].delta.content or ''
    response = response.replace("```", "").replace("markdown", "")
    update_display(Markdown(response), display_id=display_handle.display_id)
```

---

## Claude Chat Completion Examples

### Claude Basic Chat Completion

```python
message = claude.messages.create(
    model="claude-3-5-sonnet-latest",
    max_tokens=200,
    temperature=0.7,
    system=system_message,
    messages=[
        {"role": "user", "content": user_prompt},
    ],
)

print(message.content[0].text)
```

### Claude Streaming Response

```python
result = claude.messages.stream(
    model="claude-3-5-sonnet-latest",
    max_tokens=200,
    temperature=0.7,
    system=system_message,
    messages=[
        {"role": "user", "content": user_prompt},
    ],
)

with result as stream:
    for text in stream.text_stream:
        clean_text = text.replace("\n", " ").replace("\r", " ")
        print(clean_text, end="", flush=True)
```






## Gemini Chat Completion Examples
``` python
#import statement
import google.generativeai
# getting api key from .env file
google_api_key = os.getenv('GOOGLE_API_KEY')
#configuring the api
google.generativeai.configure()

#calling the api
gemini = google.generativeai.GenerativeModel(
    model_name='gemini-2.0-flash-exp',
    system_instruction=system_message
)
#getting response from the api
response = gemini.generate_content(user_prompt)
#printing the response
print(response.text)
```

## Other cheat sheets
[[Gradio Implementation Guide]]
