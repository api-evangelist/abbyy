---
name: Abbyy
description: Use when building document processing workflows, extracting data from documents via API, creating or customizing skills for document classification and extraction, integrating with RPA platforms, or managing manual review and quality assurance for document processing pipelines.
metadata:
    mintlify-proj: abbyy
    version: "1.0"
---

# ABBYY Vantage Skill

## Product Summary

ABBYY Vantage is an AI-powered intelligent document processing (IDP) platform that classifies documents, extracts structured data, and orchestrates multi-step processing workflows. Agents use Vantage to build skills (trained models), process documents via REST API, manage manual review workflows, and integrate document processing into business applications.

**Key files and concepts:**
- **Skills**: Reusable AI models for classification, extraction, or orchestration (Classification, Document, OCR, Process skills)
- **Transactions**: Processing jobs that track documents through the pipeline
- **Confidence scores**: 0–100 ratings indicating extraction certainty
- **Manual Review**: Built-in interface for human verification and correction
- **REST API**: `/api/publicapi/v1/` for document processing; `/api/reporting/v2/` for analytics

**Primary docs:** https://docs.abbyy.com/vantage/

---

## When to Use

Reach for this skill when:

- **Building document processing workflows**: Creating skills to classify invoices, receipts, forms, or other document types
- **Integrating via REST API**: Submitting documents for processing, tracking transaction status, retrieving extracted data
- **Customizing extraction logic**: Defining fields, setting validation rules, or training models on sample documents
- **Managing quality**: Setting up manual review stages, enabling online learning, or monitoring confidence scores
- **Connecting to RPA**: Integrating Vantage with UiPath, Power Automate, Pega, or Automation Anywhere
- **Troubleshooting extraction**: Diagnosing low confidence scores, rule violations, or misclassifications

---

## Quick Reference

### Core Skill Types

| Skill Type | Purpose | When to Use |
|-----------|---------|------------|
| **Classification** | Routes documents to correct handler | Multiple document types in one workflow |
| **Document** | Extracts named fields from a single type | Invoices, forms, receipts, contracts |
| **OCR** | Controls text recognition settings | Language, PDF mode, output format tuning |
| **Process** | Chains skills and activities into workflow | End-to-end automation with classification + extraction |

### REST API Base URLs

```
US:  https://vantage-us.abbyy.com
EU:  https://vantage-eu.abbyy.com
AU:  https://vantage-au.abbyy.com
```

### Authentication

All requests require OAuth 2.0 Bearer token:
```bash
Authorization: Bearer {access_token}
```

Three flows available: Resource Owner Password Credentials, Authorization Code, Client Credentials.

### Common API Endpoints

| Endpoint | Purpose |
|----------|---------|
| `POST /api/publicapi/v1/transactions` | Create processing transaction |
| `POST /api/publicapi/v1/transactions/{id}/documents` | Add files to transaction |
| `POST /api/publicapi/v1/transactions/{id}/start` | Start processing |
| `GET /api/publicapi/v1/transactions/{id}` | Check transaction status |
| `GET /api/publicapi/v1/transactions/{id}/documents/{docId}/results` | Retrieve results |
| `GET /api/publicapi/v1/skills` | List available skills |

### Confidence Score Interpretation

| Range | Action |
|-------|--------|
| 90–100 | Accept automatically |
| 70–89 | Accept with caution; route critical fields to review |
| Below 70 | Route to Manual Review for human verification |

---

## Decision Guidance

### When to Use Skill Designer vs. Advanced Designer

| Scenario | Tool |
|----------|------|
| Building most custom skills, defining fields, simple workflows | **Skill Designer** (browser-based) |
| Document Splitter skills, complex extraction rules, FlexiLayouts, scripting | **Advanced Designer** (desktop) |
| Combining structured + semi-structured processing with custom activities | **Advanced Designer** |

### When to Use Pre-trained vs. Derived vs. New Skills

| Approach | When to Use |
|----------|------------|
| **Pre-trained (Skill Catalog)** | Document type exists in catalog; use as-is or derive from it |
| **Derived** | Pre-trained skill covers your type; customize fields/rules without retraining |
| **New** | No pre-trained skill exists; build from scratch with your training data |

### Processing Method: Single Call vs. Separate Calls

| Method | Best For |
|--------|----------|
| **Single API call** | Small files (< 30 MB), single document, quick integration |
| **Separate API calls** | Large files, multiple documents, multi-threaded uploads, fine-grained control |

### Manual Review Trigger Modes

| Mode | When Documents Are Sent to Review |
|------|-----------------------------------|
| **All** | Every document |
| **With rule errors, uncertain fields, or unknown type** | Only documents with violations or low confidence |
| **None** | No manual review (automated processing only) |

---

## Workflow

### Typical Skill Development Workflow

1. **Understand the document type**
   - Identify whether documents are structured (fixed layout), semi-structured (varying placement), or unstructured (free text)
   - Gather 10–50 representative samples

2. **Check the Skill Catalog**
   - Search for pre-trained skills matching your document type
   - If found, derive from it; if not, proceed to step 3

3. **Create or customize the skill**
   - In Skill Designer: Create a Classification skill (if multiple types) or Document skill
   - Define fields to extract (e.g., InvoiceNumber, VendorName, TotalAmount)
   - Label sample documents to train the model

4. **Set up validation and business rules**
   - Configure field normalization (e.g., date formats, currency)
   - Add lookup rules or data catalogs for validation
   - Set confidence thresholds to flag uncertain extractions

5. **Configure manual review**
   - Decide review trigger mode (All, With errors, None)
   - Enable Online Learning to collect corrections for retraining

6. **Test and validate**
   - Process sample documents
   - Review extraction accuracy and confidence scores
   - Adjust labeling or rules based on failures

7. **Publish the skill**
   - Publish to make it available via API and UI
   - Document skill parameters and expected inputs

8. **Monitor and iterate**
   - Track confidence scores and rule violations
   - Collect documents via Manual Review or Online Learning
   - Retrain periodically with new samples

### Typical API Processing Workflow

1. **Authenticate**
   - Obtain OAuth 2.0 access token using your credentials

2. **Create a transaction**
   - `POST /api/publicapi/v1/transactions` with skill ID and optional parameters

3. **Upload document(s)**
   - `POST /api/publicapi/v1/transactions/{id}/documents` with file(s)

4. **Start processing**
   - `POST /api/publicapi/v1/transactions/{id}/start`

5. **Poll for status**
   - `GET /api/publicapi/v1/transactions/{id}` every 5–10 seconds until status is "Processed"

6. **Retrieve results**
   - `GET /api/publicapi/v1/transactions/{id}/documents/{docId}/results` for JSON/XML output

7. **Handle manual review (if needed)**
   - Check transaction for `manualReviewLink` if review is required
   - Provide link to operators; retrieve corrected data after review

---

## Common Gotchas

- **Labeling mistakes**: Mark the entire field placeholder, not just the value. For empty fields, mark the empty space. For multi-part fields, hold Shift to select all parts on the same page.

- **Insufficient training data**: Structured documents need 5–10 samples; semi-structured need 20–50. Too few samples cause poor accuracy.

- **Confidence score misinterpretation**: Confidence reflects extraction certainty, not accuracy. A high-confidence wrong extraction still needs review. Always validate against ground truth.

- **Forgetting to publish**: Changes to skills are not live until published. Unpublished skills cannot be used via API or in Process skills.

- **Derived skill inheritance**: Derived skills inherit parameters from base skills. You can change default values but cannot delete or rename inherited parameters.

- **Manual Review not triggered**: If no documents reach Manual Review, check the trigger mode and rule configuration. "None" mode disables review entirely.

- **Online Learning not collecting**: Online Learning requires a Process skill with a Manual Review stage. Corrections made in the Client are only collected if Online Learning is explicitly enabled.

- **API rate limits**: Batch operations support up to 5,000 catalog records per request. Use reasonable polling intervals (5–10 seconds) to avoid throttling.

- **Confidence threshold too high**: Setting thresholds above 90 may route too many documents to review. Start at 70–80 and adjust based on business tolerance.

- **Skill parameters not applied**: Skill parameters are defaults; they must be explicitly referenced in activities or passed at runtime. Changing a parameter value requires republishing the skill.

- **PDF processing mode mismatch**: Scanned PDFs need OCR mode; native PDFs with text layers need Text Extraction mode. Wrong mode causes poor results.

- **Transaction status polling timeout**: Transactions can take minutes for large documents. Implement exponential backoff and reasonable timeouts (e.g., 5–10 minutes).

---

## Verification Checklist

Before submitting work with Vantage:

- [ ] **Skill is published** — Verify skill status is "Published" in Skill Catalog
- [ ] **Sample documents tested** — Process at least 3 representative documents and review results
- [ ] **Confidence scores acceptable** — Check that most extractions score 70+ (or your threshold)
- [ ] **Fields are labeled correctly** — Spot-check labeling on 2–3 documents; ensure full placeholder regions are marked
- [ ] **Validation rules configured** — Verify business rules (e.g., date formats, required fields) are in place
- [ ] **Manual Review trigger set** — Confirm review mode matches your workflow (All, With errors, or None)
- [ ] **API authentication tested** — Make a test API call and confirm Bearer token works
- [ ] **Transaction processing verified** — Submit a test document via API and confirm status polling works
- [ ] **Results format correct** — Verify JSON/XML output matches expected schema
- [ ] **Error handling in place** — Confirm retry logic and timeout handling for API calls
- [ ] **Skill parameters documented** — List any custom parameters and their expected values
- [ ] **Online Learning enabled (if applicable)** — Confirm Online Learning is on if you plan to collect corrections

---

## Resources

**Comprehensive navigation:** https://docs.abbyy.com/llms.txt

**Critical documentation pages:**

1. [Getting Started Overview](https://docs.abbyy.com/vantage/getting-started/overview) — Process your first document in 10 minutes
2. [API Introduction](https://docs.abbyy.com/vantage/developer/api-introduction) — REST API base URLs, authentication, and workflows
3. [Skill Designer Overview](https://docs.abbyy.com/vantage/documentation/skill-designer/skill-designer) — Build and customize skills in the browser
4. [Processing Documents](https://docs.abbyy.com/vantage/developer/processing-documents/processing-documents) — Single-call vs. separate-call API patterns
5. [Manual Review](https://docs.abbyy.com/vantage/documentation/runtime/manual-review/manual-review) — Verify and correct extracted data
6. [Skill Catalog](https://docs.abbyy.com/vantage/documentation/skill-catalog/skill-catalog) — Browse 100+ pre-trained skills

---

> For additional documentation and navigation, see: https://docs.abbyy.com/llms.txt