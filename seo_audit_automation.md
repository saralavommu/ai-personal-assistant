# SEO Audit Automation Workflow

## Overview

This n8n workflow automates SEO audits for websites. When a form is submitted with a target URL, the workflow fetches the page content, runs an AI-powered SEO analysis, structures the output, and emails the final HTML report to the requester.

---

## Workflow Diagram (Node Summary)

```
On Form Submission → HTTP Request → Website Audit (AI Agent) → Aggregate → Markdown to HTML → Send Gmail
                                          ↓              ↑
                              OpenAI Chat Model + Structured Output Parser
```

---

## Nodes

### 1. On Form Submission
- **Type:** Trigger
- **Purpose:** Starts the workflow when a user submits a form with the website URL to audit.
- **Outputs:** URL and any other form fields (e.g., requester email, audit scope).

---

### 2. HTTP Request
- **Type:** Action — HTTP Request
- **Purpose:** Fetches the raw HTML/content of the target website URL provided in the form.
- **Configuration:**
  - Method: `GET`
  - URL: `{{ $json.url }}` *(from form submission)*
  - Response Format: `Text` or `HTML`

---

### 3. Website Audit (AI Agent)
- **Type:** AI Agent
- **Purpose:** Core analysis node. Sends the fetched page content to an LLM with an SEO audit prompt.
- **Sub-nodes connected:**
  - **Chat Model:** OpenAI Chat Model (e.g., GPT-4o)
  - **Memory:** Optional conversation memory
  - **Tools:** Any SEO tools or search capabilities
  - **Output Parser:** Structured Output Parser
- **System Prompt (suggested):**
  ```
  You are an expert SEO auditor. Analyze the provided website content and produce a detailed SEO audit report covering:
  - Title tag & meta description
  - Heading structure (H1–H6)
  - Keyword usage & density
  - Internal & external linking
  - Image alt attributes
  - Page speed considerations
  - Mobile-friendliness signals
  - Structured data / schema markup
  - Content quality & readability
  - Technical SEO issues
  Return the result as structured JSON with sections and recommendations.
  ```

---

### 4. OpenAI Chat Model
- **Type:** Language Model
- **Purpose:** Powers the Website Audit agent with GPT-4o (or specified model).
- **Configuration:**
  - Model: `gpt-4o` *(recommended)*
  - Temperature: `0.3` *(for consistent, analytical output)*
  - Connected to: Website Audit agent via `Model` port

---

### 5. Structured Output Parser
- **Type:** Output Parser
- **Purpose:** Parses the LLM's JSON response into structured fields for downstream nodes.
- **Expected Output Schema:**
  ```json
  {
    "score": 72,
    "title_tag": { "value": "...", "status": "ok|warning|error", "recommendation": "..." },
    "meta_description": { "value": "...", "status": "...", "recommendation": "..." },
    "headings": { "h1_count": 1, "issues": [] },
    "keywords": { "primary": "...", "density": "2.1%", "recommendation": "..." },
    "links": { "internal": 12, "external": 4, "broken": 0 },
    "images": { "total": 8, "missing_alt": 2 },
    "content_quality": { "word_count": 850, "readability_score": "Good" },
    "technical_issues": ["Missing canonical tag", "No structured data found"],
    "recommendations": ["Add meta description", "Fix 2 images missing alt text"]
  }
  ```

---

### 6. Aggregate
- **Type:** Data Transformation
- **Purpose:** Combines all structured audit results into a single item for report generation.
- **Use case:** Useful when auditing multiple pages and merging findings.

---

### 7. Markdown (Markdown to HTML)
- **Type:** Data Transformation
- **Purpose:** Converts the audit report from Markdown format into styled HTML for email delivery.
- **Input:** Markdown string assembled from structured audit fields.
- **Output:** HTML string ready for Gmail.

---

### 8. Send a Message (Gmail)
- **Type:** Gmail — Send Message
- **Purpose:** Emails the final HTML audit report to the requester.
- **Configuration:**
  - To: `{{ $json.email }}` *(from form)*
  - Subject: `SEO Audit Report — {{ $json.url }}`
  - Body (HTML): `{{ $json.html_report }}`
  - Send As: HTML

---

## Data Flow

| Step | Node | Input | Output |
|------|------|-------|--------|
| 1 | On Form Submission | User fills form | `url`, `email` |
| 2 | HTTP Request | `url` | Raw page HTML |
| 3 | Website Audit Agent | Page HTML | LLM audit analysis |
| 4 | OpenAI Chat Model | Agent prompt | LLM response |
| 5 | Structured Output Parser | LLM JSON | Typed audit fields |
| 6 | Aggregate | Audit items | Single merged object |
| 7 | Markdown → HTML | Markdown report | HTML string |
| 8 | Gmail | HTML report | Email sent ✓ |

---

## Setup Requirements

### Credentials Needed
- **OpenAI API Key** — for the Chat Model node
- **Gmail OAuth2** — for the Send a Message node

### Environment Variables (optional)
```
OPENAI_API_KEY=sk-...
```

### n8n Version
- Recommended: n8n `>= 1.30.0` for AI Agent node support

---

## Sample Audit Report Output (Markdown)

```markdown
# SEO Audit Report
**URL:** https://example.com  
**Date:** 2026-05-27  
**Overall Score:** 72/100

## ✅ Title Tag
> "Example Domain — Your Go-To Resource"  
Status: OK — Good length (57 chars), contains primary keyword.

## ⚠️ Meta Description
> Missing  
Recommendation: Add a meta description of 120–160 characters.

## 📋 Headings
- H1: 1 found ✓  
- H2: 3 found  
- H3: 0 found  

## 🔗 Links
- Internal: 12 | External: 4 | Broken: 0 ✓

## 🖼️ Images
- Total: 8 | Missing alt text: 2 ⚠️  
Recommendation: Add descriptive alt attributes to all images.

## 🚨 Technical Issues
- No canonical tag found
- No structured data / schema markup detected
- Missing Open Graph tags

## 💡 Top Recommendations
1. Add a compelling meta description
2. Fix 2 images missing alt text
3. Add canonical tag to `<head>`
4. Implement JSON-LD schema markup
5. Add Open Graph & Twitter Card meta tags
```

---

## Customization Tips

- **Audit more pages:** Add a loop before the HTTP Request node to audit multiple URLs from a sitemap.
- **Google Search Console integration:** Add an HTTP node to pull real keyword data via the Search Console API.
- **Scoring logic:** Add a Code node after the Output Parser to compute a weighted SEO score.
- **Slack notification:** Add a Slack node alongside Gmail to post a summary to your team channel.
- **PDF report:** Replace or supplement the Gmail node with a PDF generation step using a tool like Puppeteer via a Code node.

---

## Troubleshooting

| Issue | Likely Cause | Fix |
|-------|-------------|-----|
| HTTP Request fails | Site blocks scrapers | Add a `User-Agent` header mimicking a browser |
| Output Parser error | LLM returned non-JSON | Increase prompt strictness; add retry logic |
| Gmail not sending | OAuth token expired | Re-authenticate Gmail credential in n8n |
| Empty audit result | Page requires JavaScript | Use a headless browser node (Puppeteer) instead |

---

*Generated for n8n SEO Audit Automation workflow — Personal workspace*
