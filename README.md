# Simple AI Agent - Kestra Workflow

A Kestra workflow that demonstrates a basic AI agent for text summarization with configurable length and language options. This project is part of the LLM Zoomcamp bonus module.

## Overview

This workflow showcases:
- **AI Agent Architecture**: How to structure and configure AI agent prompts in Kestra
- **Multi-Language Support**: Generate summaries in English, French, German, Spanish, Italian, Portuguese, and Japanese
- **Configurable Output**: Control summary length (short, medium, long)
- **Task Chaining**: Demonstrates how to chain multiple AI agent tasks
- **Token Usage Tracking**: Monitor and log token consumption for cost analysis
- **DRY Configuration**: Uses `pluginDefaults` to avoid repetition and maintain consistency

## Features

✨ **Key Capabilities:**
- Text summarization with AI-powered agents powered by Google Gemini
- Multi-language output in 7 languages
- Configurable summary lengths (short: 1-2 sentences, medium: 2-5 sentences, long: 1-3 paragraphs)
- Two-stage processing: multi-language summary + English brevity refinement
- Real-time token usage monitoring for cost transparency
- Docker-based local development environment with PostgreSQL

## Architecture

```
Input (Text to Summarize)
        ↓
    Multilingual Agent
    (Summarize in selected language/length)
        ↓
    English Brevity Agent
    (Refine to exactly 3 sentences)
        ↓
    Token Usage Logger
    (Track and display resource consumption)
```

## Prerequisites

- **Docker & Docker Compose**: For running the Kestra stack locally
- **Google Gemini API Key**: Required for AI model access
- **Optional APIs**: Tavily Search API and OpenAI API keys (for other workflows)

## Quick Start

### 1. Clone & Setup Environment

```bash
cd c:\Projects\orchaestration
```

### 2. Configure API Keys

Create a `.env` file in the project root:

```env
SECRET_GEMINI_API_KEY=your_gemini_api_key_here
SECRET_TAVILY_API_KEY=your_tavily_api_key_here
SECRET_OPENAI_API_KEY=your_openai_api_key_here
```

**Obtaining API Keys:**
- [Google Gemini API](https://ai.google.dev/): Get your free API key
- [Tavily Search API](https://tavily.com/): Optional, for web search capabilities
- [OpenAI API](https://platform.openai.com/): Optional, for alternative LLM models

### 3. Start the Kestra Stack

```bash
docker-compose up -d
```

This will start:
- **Kestra Server**: Available at `http://localhost:8080`
- **PostgreSQL Database**: For workflow persistence and state management

### 4. Access Kestra UI

Open your browser and navigate to:
```
http://localhost:8080
```

**Credentials:**
- Email: `admin@kestra.io`
- Password: `Admin1234!`

### 5. Deploy the Workflow

In the Kestra UI:
1. Go to **Flows** section
2. Create a new flow or upload `4_simple_agent.yaml`
3. Ensure `SECRET_GEMINI_API_KEY` is configured in settings
4. Save and trigger the workflow

## Usage

### Running the Workflow

**Via Kestra UI:**
1. Navigate to the `4_simple_agent` flow
2. Click **Execute**
3. Fill in the form:
   - **Text to summarize**: Paste your content
   - **Summary Length**: Choose short/medium/long
   - **Language**: Select desired output language
4. Click **Submit**

**Input Parameters:**
| Parameter | Type | Options | Default |
|-----------|------|---------|---------|
| `text` | String | Any text | Sample Kestra description |
| `summary_length` | Select | short, medium, long | medium |
| `language` | Select | en, fr, de, es, it, pt, ja | en |

### Output

The workflow produces:
1. **Multilingual Summary** (`outputs.multilingual_agent.textOutput`): Summary in your selected language and length
2. **English Brevity Summary** (`outputs.english_brevity.textOutput`): 3-sentence English refinement
3. **Token Usage Report**: Detailed token consumption for both agents

### Example Output

```
📊 Token Usage Summary:

Multilingual Agent:
- Input tokens: 1,234
- Output tokens: 156
- Total tokens: 1,390

English Brevity Agent:
- Input tokens: 312
- Output tokens: 48
- Total tokens: 360

💡 Tip: Monitor token usage to understand costs and optimize prompts!
```

## Configuration

### Plugin Defaults

The workflow uses `pluginDefaults` to configure the Google Gemini provider for all AI agents:

```yaml
pluginDefaults:
  - type: io.kestra.plugin.ai.agent.AIAgent
    values:
      provider:
        type: io.kestra.plugin.ai.provider.GoogleGemini
        modelName: gemini-2.5-flash
        apiKey: "{{ secret('GEMINI_API_KEY') }}"
```

**To use a different model:**
1. Edit `4_simple_agent.yaml`
2. Change `modelName` to your preferred Gemini model
3. Re-deploy the flow

### System Prompts

The workflow contains two agent definitions:

**Multilingual Agent:**
- Generates summaries in the requested language
- Respects summary length guidelines
- Removes marketing language and focuses on facts

**English Brevity Agent:**
- Refines the multilingual summary
- Produces exactly 3 sentences
- Ensures concise English output

Customize these prompts directly in the YAML for different behavior.

## Docker Compose Services

### kestra_postgres
- **Image**: postgres:18
- **Port**: 5432 (internal)
- **Credentials**: kestra / k3str4
- **Volume**: Persistent PostgreSQL data

### kestra
- **Image**: kestra/kestra:v1.3.21
- **UI Port**: 8080
- **API Port**: 8081
- **Mount Points**: 
  - `/app/storage`: Workflow execution history
  - `/var/run/docker.sock`: Docker socket for container tasks
  - `/tmp/kestra-wd`: Temporary working directory

## Managing the Stack

### View Logs
```bash
docker-compose logs -f kestra
docker-compose logs -f kestra_postgres
```

### Stop Services
```bash
docker-compose down
```

### Remove Data (Warning: Deletes all history)
```bash
docker-compose down -v
```

## Troubleshooting

### Issue: "GEMINI_API_KEY not found"
- **Solution**: Ensure `.env` file is in the project root and contains `SECRET_GEMINI_API_KEY`
- Verify it's accessible to Docker: `echo $SECRET_GEMINI_API_KEY`

### Issue: Database connection errors
- **Solution**: Check PostgreSQL health: `docker-compose ps`
- Wait 30-60 seconds for DB initialization
- View logs: `docker-compose logs kestra_postgres`

### Issue: Port 8080 already in use
- **Solution**: Change port in `docker-compose.yml`:
  ```yaml
  ports:
    - "8888:8080"  # Access at localhost:8888
  ```

### Issue: Out of memory
- **Solution**: Increase Docker memory allocation in Docker Desktop settings
- Or reduce log retention and execution history in Kestra settings

## Advanced Usage

### Custom Prompts
Edit the `systemMessage` and `prompt` fields in task definitions to customize behavior.

### Scheduling
Add a `triggers` section to run the workflow on a schedule:
```yaml
triggers:
  - type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 9 * * MON"  # Every Monday at 9 AM
```

### Error Handling
Add retry logic or conditional branching:
```yaml
retry:
  type: exponential
  interval: PT10S
  maxAttempts: 3
```

## Cost Optimization

- Monitor `tokenUsage` outputs to track Gemini API costs
- Adjust `summary_length` to reduce token consumption
- Consider batch processing for multiple documents
- Use shorter prompts when possible

## Learning Resources

- [Kestra Documentation](https://kestra.io/docs)
- [Google Gemini API Guide](https://ai.google.dev/docs)
- [LLM Zoomcamp](https://llm-zoomcamp.readthedocs.io/)

## License

This project is part of the LLM Zoomcamp educational content.

## Support

For issues and questions:
- Check [Kestra Community Slack](https://slack.kestra.io)
- Review [Kestra GitHub Issues](https://github.com/kestra-io/kestra/issues)
- Consult [Gemini API Documentation](https://ai.google.dev/)
