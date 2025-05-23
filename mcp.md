# End-to-End Guide to Model Context Protocol (MCP)

## Table of Contents

1. Introduction
2. Prerequisites
3. Building the MCP Server
4. Creating MCP Clients
5. Deploying to Hugging Face Spaces
6. Troubleshooting
7. Summary

---

## 1. Introduction

This guide walks you through building, deploying, and connecting to a complete MCP application. You will create a sentiment analysis tool, expose it as an MCP server, and connect to it using both Python and JavaScript clients.

---

## 2. Prerequisites

- Understanding of MCP concepts
- Python and JavaScript/TypeScript basics
- Client-server architecture knowledge
- Python 3.10+ and Node.js 18+ installed
- Hugging Face account for deployment

---

## 3. Building the MCP Server

### Project Setup

Create a new project and install dependencies:

```sh
mkdir mcp-sentiment
cd mcp-sentiment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install "gradio[mcp]" textblob
```

### Create the Server Script

Create `app.py`:

```python
import gradio as gr
from textblob import TextBlob

def sentiment_analysis(text: str) -> dict:
    """Analyze the sentiment of the given text."""
    blob = TextBlob(text)
    sentiment = blob.sentiment
    return {
        "polarity": round(sentiment.polarity, 2),
        "subjectivity": round(sentiment.subjectivity, 2),
        "assessment": (
            "positive" if sentiment.polarity > 0 else
            "negative" if sentiment.polarity < 0 else
            "neutral"
        )
    }

demo = gr.Interface(
    fn=sentiment_analysis,
    inputs=gr.Textbox(placeholder="Enter text to analyze..."),
    outputs=gr.JSON(),
    title="Text Sentiment Analysis",
    description="Analyze the sentiment of text using TextBlob"
)

if __name__ == "__main__":
    demo.launch(mcp_server=True)
```

### Run and Test

```sh
python app.py
```

- Web UI: http://localhost:7860
- MCP Server: http://localhost:7860/gradio_api/mcp/sse
- Schema: http://localhost:7860/gradio_api/mcp/schema

---

## 4. Creating MCP Clients

### Configure `mcp.json`

```json
{
  "servers": [
    {
      "name": "MCP Server",
      "transport": {
        "type": "sse",
        "url": "http://localhost:7860/gradio_api/mcp/sse"
      }
    }
  ]
}
```

### Python Client Example

```python
from smolagents import ToolCollection, CodeAgent
from mcp.client.sse import SSEServerParameters

server_params = SSEServerParameters(url="http://localhost:7860/gradio_api/mcp/sse")

with ToolCollection.from_mcp(server_params, trust_remote_code=True) as tools:
    agent = CodeAgent(tools=[*tools.tools])
    agent.run("What is the sentiment of 'I love working with MCP!'?")
```

### JavaScript Client Example

```js
const response = await mcpClient.call("sentiment_analysis", {
  text: "MCP is amazing!",
});
console.log(response);
```

---

## 5. Deploying to Hugging Face Spaces

### Create `requirements.txt`

```
gradio[mcp]
textblob
```

### Push to Hugging Face

```sh
git init
git add app.py requirements.txt
git commit -m "Initial commit"
git remote add origin https://huggingface.co/spaces/YOUR_USERNAME/mcp-sentiment
git push -u origin main
```

Update your config for the deployed server:

```json
{
  "servers": [
    {
      "name": "Remote MCP",
      "transport": {
        "type": "sse",
        "url": "https://YOUR_USERNAME-mcp-sentiment.hf.space/gradio_api/mcp/sse"
      }
    }
  ]
}
```

---

## 6. Troubleshooting

| Issue                                  | Fix                                           |
| -------------------------------------- | --------------------------------------------- |
| `tools/list` not showing your function | Ensure type hints and docstrings are included |
| Client can’t connect                   | Check server URL and restart server           |
| SSE unsupported                        | Use `mcp-remote` wrapper with Node.js         |

---

## 7. Summary

By following this guide, you have:

- Built and tested an MCP server with Gradio
- Exposed a sentiment analysis tool to LLMs
- Connected using Python and JavaScript clients
- Deployed to Hugging Face Spaces for global use

This end-to-end documentation provides a complete reference for building, deploying, and connecting MCP applications.
