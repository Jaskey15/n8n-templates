# n8n Workflow Templates

Production-tested n8n workflow templates for CRM automation, AI-powered lead nurturing, and more. Built on GoHighLevel (GHL) + OpenAI.

## Templates

| Template | Description | Workflows |
|----------|-------------|-----------|
| [Lead Nurture Pipeline](templates/lead-nurture-pipeline/) | Automated multi-stage SMS nurture system with AI-generated messages | 7 |
| [Call Transcript Summarizer](templates/call-transcript-summarizer/) | Automatically summarize inbound/outbound calls and post to CRM notes | 1 |

## How to Use

1. Pick a template from the table above
2. Read its README for prerequisites and setup steps
3. Import the `.json` workflow files into your n8n instance
4. Search for `YOUR_` and `{{` placeholders and replace with your values
5. Configure credentials and activate

## Prerequisites

All templates assume:
- A self-hosted or cloud [n8n](https://n8n.io) instance
- A [GoHighLevel](https://www.gohighlevel.com/) account (CRM)
- An [OpenAI](https://platform.openai.com/) API key

## Project Structure

```
templates/
  lead-nurture-pipeline/
    README.md              Setup guide & architecture
    workflows/             7 importable workflow JSON files
  call-transcript-summarizer/
    README.md              Setup guide
    workflows/             1 importable workflow JSON file
```

## License

[MIT](LICENSE)
