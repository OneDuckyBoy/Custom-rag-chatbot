## Basic Gradio Setup

### Importing Gradio

```python
import gradio as gr
```

### Simple Method Example

```python
def shout(text):
    print(f"Shout has been called with input {text}")
    return text.upper()
```

### Basic Implementation

```python
gr.Interface(fn=shout, inputs="textbox", outputs="textbox").launch()
```

## Configuration Options

### Disable Flagging

```python
gr.Interface(
    fn=shout, 
    inputs="textbox", 
    outputs="textbox", 
    flagging_mode="never"
).launch()
```

### Global Sharing (72 hours)

```python
gr.Interface(
    fn=shout, 
    inputs="textbox", 
    outputs="textbox", 
    flagging_mode="never"
).launch(share=True)
```

### Open in Browser Tab

```python
gr.Interface(
    fn=shout, 
    inputs="textbox", 
    outputs="textbox", 
    flagging_mode="never"
).launch(inbrowser=True)
```

## Customizing UI

### Force Dark Mode

```python
force_dark_mode = """
function refresh() {
    const url = new URL(window.location);
    if (url.searchParams.get('__theme') !== 'dark') {
        url.searchParams.set('__theme', 'dark');
        window.location.href = url.href;
    }
}
"""

gr.Interface(
    fn=shout, 
    inputs="textbox", 
    outputs="textbox", 
    flagging_mode="never", 
    js=force_dark_mode
).launch()
```

### Custom Input/Output Components

```python
view = gr.Interface(
    fn=shout,
    inputs=[gr.Textbox(label="Your message:", lines=6)],
    outputs=[gr.Textbox(label="Response:", lines=8)],
    flagging_mode="never"
)
view.launch()
```

## Integration with AI Models

### Basic OpenAI Integration

```python
system_message = "You are a helpful assistant that responds in markdown"

def message_gpt(prompt):
    messages = [
        {"role": "system", "content": system_message},
        {"role": "user", "content": prompt}
    ]
    completion = openai.chat.completions.create(
        model='gpt-4o-mini',
        messages=messages,
    )
    return completion.choices[0].message.content

gr.Interface(
    fn=message_gpt,
    inputs=[gr.Textbox(label="Your message:")],
    outputs=[gr.Markdown(label="Response:")],
    flagging_mode="never"
).launch()
```

### Streaming Responses

#### Streaming with OpenAI

```python
def stream_gpt(prompt):
    messages = [
        {"role": "system", "content": system_message},
        {"role": "user", "content": prompt}
    ]
    stream = openai.chat.completions.create(
        model='gpt-4o-mini',
        messages=messages,
        stream=True
    )
    result = ""
    for chunk in stream:
        result += chunk.choices[0].delta.content or ""
        yield result
    # We have to return the new chunk with the older chunks
    # Otherwise it will only display the latest chunk

view = gr.Interface(
    fn=stream_gpt,
    inputs=[gr.Textbox(label="Your message:")],
    outputs=[gr.Markdown(label="Response:")],
    flagging_mode="never"
)
view.launch()
```

#### Streaming with Claude

```python
def stream_claude(prompt):
    result = claude.messages.stream(
        model="claude-3-haiku-20240307",
        max_tokens=1000,
        temperature=0.7,
        system=system_message,
        messages=[
            {"role": "user", "content": prompt},
        ],
    )
    response = ""
    with result as stream:
        for text in stream.text_stream:
            response += text or ""
            yield response
```

### Model Selector with Dropdown

```python
def stream_model(prompt, model):
    if model == "GPT":
        result = stream_gpt(prompt)
    elif model == "Claude":
        result = stream_claude(prompt)
    else:
        raise ValueError("Unknown model")
    yield from result

gr.Interface(
    fn=stream_model,
    inputs=[
        gr.Textbox(label="Your message:"), 
        gr.Dropdown(["GPT", "Claude"], label="Select model", value="GPT")
    ],
    outputs=[gr.Markdown(label="Response:")],
    flagging_mode="never"
).launch()
```

## Component Reference

### Dropdown Menu

```python
gr.Dropdown(
    ["GPT", "Claude"], 
    label="Select model", 
    value="GPT"  # Default value
)
```

### Markdown Output

```python
gr.Markdown(label="Response:")
```