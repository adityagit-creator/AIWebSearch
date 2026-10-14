# AIWebSearch for exploring particular business and what  someone can bring to the table

# LocalLead AI

### Open-Weight AI-Powered Business Research & Lead Discovery

LocalLead AI is a lightweight, open-weight AI application that analyzes public business websites, understands their services and digital capabilities, and identifies potential software development and automation opportunities.

The application is designed for freelancers, software development agencies, and small businesses looking to identify potential clients without spending hours manually researching websites.

**Built for DEV Hacktoberfest 2026 — Week 2: Side Hustle**

## 1. Problem Statement

Finding potential clients is often a time-consuming process for freelancers and software agencies.

A typical lead research workflow involves:

1. Finding a business website.
2. Reading its homepage.
3. Navigating through its About, Services, Products, and Contact pages.
4. Understanding the business model.
5. Identifying possible digital transformation opportunities.
6. Preparing a summary before approaching the business.

This process becomes repetitive when researching dozens of companies.

**The problem:** How can we automate the initial business research process using open-weight AI while keeping the implementation lightweight and affordable?

## 2. Thought Process

### Initial idea: Browser-based AI agent

The project was inspired by the [Browser MCP Agent](https://github.com/Shubhamsaboo/awesome-llm-apps/tree/main/mcp_ai_agents/browser_mcp_agent) from the Awesome LLM Apps repository.

The original idea was to use a browser-capable AI agent to navigate websites and extract business information.

However, building a fully autonomous browser agent introduces several complexities:

- Browser automation overhead
- Multiple tool-calling operations
- Additional dependencies
- More complicated deployment
- Increased maintenance

For our use case, we realized that most public business information can initially be extracted using ordinary HTTP requests and HTML parsing.

**Decision:** Replace the complex browser-agent architecture with a lightweight website crawler and a dedicated language model.

### Second idea: Use a hosted AI API

Initially, we experimented with Hugging Face Inference Providers to access Qwen models without loading them locally.

This offered several advantages:

- No GPU setup required
- Minimal application memory
- Simple API integration

However, we encountered model availability restrictions and exhausted the available free inference credits.

**Decision:** Move inference to a locally loaded open-weight model in Kaggle rather than relying on paid inference requests.

### Final approach: Qwen3-4B on Kaggle

We selected **Qwen3-4B**, an open-weight language model, and used Kaggle's GPU environment to run inference.

The available environment included:

- Two NVIDIA Tesla T4 GPUs
- Approximately 15 GiB VRAM per GPU
- Python and PyTorch
- Qwen3-4B model weights available through Kaggle Models

The model was loaded onto one GPU using Hugging Face Transformers.

This eliminated the need for a paid model API during development.

The second GPU remained unused because the 4B model could run on a single device.

## 3. Solution Architecture

The final pipeline consists of six stages.

```text
                Business Website URL
                         |
                         v
                Website Crawler
              Requests + BeautifulSoup
                         |
                         v
             Extract Relevant Pages
          Home / About / Services / Contact
                 Navbar / Footer
                         |
                         v
              Clean HTML and Save Data
                 website_content.json
                         |
                         v
               Qwen3-4B Summarization
               website_summaries.json
                         |
                         v
              AI Lead Opportunity Analysis
                   lead_report.json
                         |
                         v
                   Gradio Interface
              Overview / JSON / Export
```

### Stage 1: Website discovery

The user enters a public website URL.

The crawler downloads the homepage and discovers internal links, including links found in navigation menus and footers.

To keep execution lightweight, the initial implementation limits crawling to a maximum of ten successfully extracted pages.

### Stage 2: Content extraction

BeautifulSoup converts the HTML into a structured document.

The system removes elements such as:

- JavaScript
- CSS
- Hidden templates
- SVG markup

Unlike the initial prototype, navigation and footer text are preserved because they may contain valuable information about business services, locations, and contact methods.

### Stage 3: Data preservation

Extracted content is saved before AI processing.

Each stored page includes:

- Source URL
- Page title
- Cleaned text content

This allows the original extracted information to be inspected without crawling the website again.

### Stage 4: Website summarization

Each page is summarized individually using Qwen3-4B.

The model is instructed to identify:

- Business identity
- Products and services
- Target customers
- Booking and contact methods
- Existing digital features
- Relevant operational information

Summarizing individual pages before combining their information reduces the amount of text needed for final lead analysis.

### Stage 5: Lead opportunity generation

The summarized information is passed to Qwen3-4B again.

The model generates a structured JSON report containing:

```json
{
  "company_name": "Example Dental Clinic",
  "business_category": "Healthcare",
  "business_summary": "A dental clinic providing routine and specialist treatments.",
  "target_customers": [
    "Dental patients"
  ],
  "existing_digital_features": [],
  "potential_software_needs": [
    {
      "solution": "Online Appointment Booking",
      "reason": "The supplied example describes phone-based appointment scheduling.",
      "supporting_source_url": "https://example.com",
      "confidence": "medium"
    }
  ],
  "suggested_services": [
    "Appointment management system"
  ],
  "lead_priority": "medium"
}
```

*This is illustrative data, not an actual researched lead.*

The system distinguishes possible software opportunities from confirmed business requirements.

A recommendation does not mean the company is actively seeking to purchase that software.

### Stage 6: User interface and exports

The application uses Gradio to provide a lightweight interactive interface.

Users can:

- Enter a business website
- Start the analysis
- View a formatted lead report
- Inspect structured JSON
- Download extracted content
- Download page summaries
- Download the final lead report

The interface uses a minimal dark design inspired by modern SaaS dashboards.

## 4. Technology Stack

| Technology | Purpose |
|---|---|
| Python | Core application |
| Qwen3-4B | Open-weight language model |
| Hugging Face Transformers | Model loading and inference |
| PyTorch | GPU execution |
| Kaggle | Development and GPU environment |
| Requests | HTTP website fetching |
| BeautifulSoup | HTML parsing and extraction |
| Gradio | Interactive web interface |
| JSON | Structured data storage |

No external vector database, paid LLM API, or agent orchestration framework is required.

## 5. Why Open Innovation Matters

Open-weight AI is central to LocalLead AI.

### Model flexibility

The application is not tightly coupled to a proprietary model provider.

Qwen3-4B can potentially be replaced with another compatible open-weight instruction model.

### Lower inference costs

Running inference directly on available GPU resources avoids paying a hosted LLM provider for each request.

However, computation is not inherently free. Kaggle provides limited GPU resources, and production hosting would have infrastructure costs.

### Greater control

We can customize model prompts, generation settings, output formats, and inference behavior.

The project currently uses a non-thinking Qwen3 inference configuration for structured information extraction.

### Reproducibility

The main processing pipeline uses accessible Python libraries and open-weight model files.

Other developers can study, modify, and reproduce the implementation in a compatible environment, subject to model licensing and compute availability.

### Data processing flexibility

Website text is processed by a model loaded within the runtime rather than sent to a paid third-party LLM endpoint.

However, Kaggle itself is a hosted environment. The application should not be described as fully offline or as guaranteeing that all data remains on a user's personal computer.

## 6. Important Implementation Decisions

### Why not Playwright?

Playwright is useful for browser automation and JavaScript-heavy websites.

For this prototype, Requests and BeautifulSoup provide a smaller dependency footprint.

The trade-off is that JavaScript-rendered content may not be captured.

### Why summarize pages individually?

Sending all extracted website text into a single prompt can increase context length and inference costs.

Page-level summarization provides a smaller input for final business analysis while preserving intermediate outputs for inspection.

However, summaries can omit details, so recommendations should be checked against their original source pages.

### Why store JSON files?

JSON provides a simple, structured output format without requiring PostgreSQL, MongoDB, or another database.

This is sufficient for the current single-website prototype.

### Why Gradio?

Gradio enables us to create an interactive interface directly from Python.

It avoids the overhead of a separate JavaScript frontend.

## 7. Running the Project

### Requirements

- Python
- Kaggle Notebook or equivalent GPU environment
- Qwen3-4B model weights
- Internet access for public website fetching

### Install dependencies

```bash
pip install torch transformers accelerate requests beautifulsoup4 gradio
```

### Load Qwen3-4B

Configure the model path to the locally available Qwen3-4B weights.

The current prototype uses the Kaggle model directory:

```text
/kaggle/input/models/qwen-lm/qwen-3/transformers/4b/1
```

### Analyze a website

After initializing the model and pipeline functions:

```python
url = "https://cheeti.tech"

result = analyze_website(url)

print(result)
```

### Launch the Gradio interface

```python
demo.queue().launch(
    share=True,
    debug=True,
    show_error=True
)
```

The Gradio sharing link is temporary and requires the Kaggle notebook session to remain active.

## 8. Generated Files

For each analyzed website, LocalLead AI stores:

```text
locallead/
└── example_com/
    ├── website_content.json
    ├── website_summaries.json
    └── lead_report.json
```

**website_content.json:** Extracted page content with original source URLs.

**website_summaries.json:** AI-generated summaries of individual pages.

**lead_report.json:** Business profile and potential software opportunities.

## 9. Challenges Encountered

### Hosted model availability

Several Qwen models were listed in the Hugging Face model catalog but were unavailable through the enabled inference providers.

We resolved this by running Qwen3-4B directly in Kaggle.

### GPU memory management

The model was loaded onto one NVIDIA Tesla T4 GPU.

We used inference mode to avoid gradient calculation and limited simultaneous Gradio analyses to reduce resource contention.

### Transformers generation input error

An early implementation passed a `BatchEncoding` object directly into `model.generate()`.

We fixed this by passing named token tensors using:

```python
model.generate(**inputs)
```

### Structured JSON generation

AI-generated JSON may occasionally be malformed.

The application attempts to parse the output and records an error if parsing fails.

More robust schema validation remains a future improvement.

### Interface readability

The first Gradio interface exposed file downloads using the default file component.

Custom dark styling caused contrast issues.

We redesigned the export section using individual download buttons and clearer typography.

## 10. Current Limitations

LocalLead AI is an experimental prototype, not a production-ready lead generation system.

Current limitations include:

- The crawler does not execute JavaScript.
- It does not comprehensively discover every page on large websites.
- A ten-page crawl limit may omit relevant information.
- AI-generated opportunities may contain unsupported assumptions.
- Similarity or lead-priority scoring is not independently validated.
- It does not verify whether companies are actively purchasing software.
- Kaggle sessions and GPU quotas are temporary.
- Public deployment requires stronger URL validation and security protections.
- Website crawling must respect site policies, access restrictions, and applicable requirements.

## 11. Future Improvements

Potential future enhancements include:

1. Prioritizing About, Services, Products, Pricing, and Contact pages.
2. Adding stronger URL validation and safe redirect handling.
3. Supporting JavaScript-rendered websites when necessary.
4. Adding schema validation for AI-generated JSON.
5. Exporting lead reports to CSV.
6. Analyzing multiple businesses in one batch.
7. Ranking opportunities using transparent criteria.
8. Adding an evidence-verification stage.
9. Supporting alternative open-weight models.
10. Building a persistent database for saved leads.
11. Collecting feedback from actual freelancers and agencies.

## 12. Potential Business Model

LocalLead AI targets freelancers and small software development agencies.

Possible monetization approaches include:

- Subscription-based lead research
- Pay-per-business analysis
- Customized prospect research reports
- White-label research tools for agencies

These are potential business models, not demonstrated revenue streams.

Before commercializing the system, we would need to validate the quality of recommendations, customer willingness to pay, hosting costs, and compliance with website access rules.

## 13. Project Inspiration and Attribution

This project was inspired by the Browser MCP Agent in the Awesome LLM Apps repository.

Original project:

https://github.com/Shubhamsaboo/awesome-llm-apps/tree/main/mcp_ai_agents/browser_mcp_agent

LocalLead AI uses a simplified, purpose-specific architecture based on Requests, BeautifulSoup, Qwen3-4B, and Gradio.

Any code directly reused from third-party repositories must retain the applicable license notices and attribution.

## 14. Hacktoberfest 2026 — Side Hustle Challenge

LocalLead AI was developed around the theme:

**Build something with open-weight models or open-source AI that someone would actually pay for.**

The project demonstrates how a small, commercially relevant research tool can be built using open-weight AI without relying on proprietary model APIs.

Its central idea is simple:

**Turn publicly available business information into structured research that helps freelancers and agencies discover potential customers more efficiently.**

---

**Status:** Working prototype under development

**Model:** Qwen3-4B

**Environment:** Kaggle GPU

**Interface:** Gradio

**Challenge:** DEV Hacktoberfest 2026 — Week 2: Side Hustle
