# SEO Audit Automation Workflow

An AI-powered SEO audit automation built with n8n, OpenAI, Google Sheets, and Gmail.

This workflow automatically reads website URLs and emails from Google Sheets, performs AI-based SEO audits, generates structured reports, converts them into HTML, and emails the final audit report to each corresponding business.

---

# Workflow Overview

```text
Google Sheets
      ↓
Loop Through Rows
      ↓
HTTP Request (Fetch Website)
      ↓
AI SEO Audit Agent
      ↓
Structured Output Parser
      ↓
Aggregate Results
      ↓
Markdown → HTML
      ↓
Send Report via Gmail
```

---

# Features

- Automated SEO audits using AI
- Reads website URLs directly from Google Sheets
- Sends personalized SEO reports automatically
- Structured JSON-based audit analysis
- HTML email report generation
- Bulk website auditing support
- Easily scalable and customizable

---

# Tech Stack

- **n8n**
- **OpenAI GPT-4o**
- **Google Sheets**
- **Gmail API**
- **Markdown to HTML**
- **HTTP Request Nodes**

---

# Google Sheets Structure

The workflow reads data from a Google Sheet with the following columns:

| Website URL | Email |
|---|---|
| https://example.com | hello@example.com |
| https://company.com | info@company.com |

Each row represents:
- A website to audit
- The email address where the SEO report should be sent

---

# Workflow Nodes

## 1. Google Sheets Node

Fetches website URLs and email addresses from Google Sheets.

### Purpose
Reads all rows containing:
- Website URLs
- Target email addresses

### Example Columns

```text
website_url
email
```

### Output Example

```json
{
  "website_url": "https://example.com",
  "email": "hello@example.com"
}
```

---

## 2. Loop Through Items

Processes each row individually.

For every website:
1. Fetch website content
2. Generate SEO audit
3. Create report
4. Send email to corresponding address

---

## 3. HTTP Request

Fetches raw HTML content from the website.

### Configuration

```bash
Method: GET
URL: {{ $json.website_url }}
Response Format: HTML / Text
```

---

## 4. AI SEO Audit Agent

Analyzes website content using OpenAI.

### Audit Includes

- Title tags
- Meta descriptions
- Heading structure
- Keyword optimization
- Internal & external links
- Image alt attributes
- Technical SEO issues
- Mobile-friendliness
- Structured data
- Content quality

### Suggested Prompt

```text
You are an expert SEO auditor.

Analyze the provided website content and generate a professional SEO audit report with actionable recommendations.

Return the output in structured JSON format.
```

---

## 5. OpenAI Chat Model

### Recommended Configuration

```yaml
Model: gpt-4o
Temperature: 0.3
```

---

## 6. Structured Output Parser

Parses the AI response into structured fields.

### Example Output

```json
{
  "score": 72,
  "title_tag": {
    "status": "ok",
    "recommendation": "Good title structure"
  },
  "meta_description": {
    "status": "warning",
    "recommendation": "Add meta description"
  },
  "technical_issues": [
    "Missing canonical tag"
  ]
}
```

---

## 7. Aggregate

Combines all SEO findings into a single object before report generation.

---

## 8. Markdown to HTML

Converts the generated markdown report into clean HTML email format.

---

## 9. Gmail Node

Sends the SEO audit report to the email address from the Google Sheet.

### Email Configuration

```bash
To: {{ $json.email }}
Subject: Free SEO Audit Report — {{ $json.website_url }}
Body: HTML Report
```

---

# Workflow Logic

For every row in Google Sheets:

```text
Read Website URL + Email
        ↓
Fetch Website Content
        ↓
Generate AI SEO Audit
        ↓
Create HTML Report
        ↓
Send Report to Matching Email
```

Example:

| Website | Email |
|---|---|
| example.com | hello@example.com |

The workflow:
- Audits `example.com`
- Sends the report to `hello@example.com`

---

# Example SEO Report

```md
# SEO Audit Report

URL: https://example.com

Overall Score: 72/100

## Title Tag
✅ Optimized properly

## Meta Description
⚠️ Missing meta description

## Technical Issues
- Missing canonical tag
- Missing schema markup

## Recommendations
1. Add meta description
2. Fix missing alt tags
3. Add structured data
```

---

# Setup

## Requirements

- OpenAI API Key
- Google Sheets Credentials
- Gmail OAuth Credentials
- n8n v1.30+

---

## Environment Variables

```env
OPENAI_API_KEY=your_api_key
```

---

# Recommended Google Sheet Format

```text
| website_url         | email              |
|---------------------|--------------------|
| https://site1.com   | hello@site1.com    |
| https://site2.com   | info@site2.com     |
```

---

# Customization Ideas

- Add automatic follow-up emails
- Integrate Google Search Console
- Add SEO scoring system
- Export reports as PDFs
- Store audit history in databases
- Add Slack notifications
- Schedule recurring audits

---

# Troubleshooting

| Issue | Solution |
|---|---|
| Website blocks request | Add browser-like User-Agent headers |
| Invalid AI response | Strengthen JSON prompt instructions |
| Gmail not sending | Reconnect Gmail OAuth |
| Empty website content | Use Puppeteer/headless browser |
| Google Sheets not loading | Reconnect Google Sheets credentials |

---

# Future Improvements

- Full website crawling
- Competitor SEO analysis
- AI-generated SEO fixes
- Dashboard analytics
- Multi-language SEO audits

# Author

Built with using n8n + OpenAI + Google Sheets