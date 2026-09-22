# 🚢 OldJeans --- AI-Powered Shipping Document Verification

OldJeans is an AI-powered shipping document verification system built
for the **SDOC Hackathon**.

The system automatically processes shipping-related emails, classifies
their intent, extracts structured information from multiple document
formats, and compares **Shipping Instructions (SI)** against **Bills of
Lading (BL)** to detect discrepancies before documents are finalized.

> **Design principle:** Use AI for understanding. Use deterministic
> logic for verification.

## 🌐 Live Demo

**Human-in-the-Loop Review Desk:**  
https://xintong16.github.io/sdoc-review-demo/

The Review Desk demonstrates how OldJeans presents AI verification results to human reviewers, including `OK`, `MISMATCH`, and `NEEDS_REVIEW` cases.

**Cloud Deployment:**  
https://tenjingyi06.app.n8n.cloud/

The AI verification workflow is deployed on **n8n Cloud**, with workflow
orchestration and Google Gemini integration running in the cloud. The
workflow connects to our **Render-hosted Inbox API**, which provides access
to the 520-email hackathon dataset and its document attachments.

**Cloud Inbox API:**  
https://oldjeans-shipping-document-verification.onrender.com/

**Presentation Slides:**  
https://canva.link/nvzrwdcn5aa39j1

**Video Demo:**  
https://drive.google.com/drive/folders/1Nd1AKirPeL4u-MlS5ZyDcm-BOPnJC8gi?usp=sharing


------------------------------------------------------------------------

## 🎯 Problem

Shipping and logistics teams process large volumes of emails and
documents every day.

Manually checking Shipping Instructions against draft Bills of Lading
is:

-   Time-consuming
-   Repetitive
-   Prone to human error
-   Difficult when documents use different formats and terminology
-   Risky when incorrect information is missed before finalization

Even small differences in consignee information, ports, container
quantities, or cargo weight can create operational problems.

OldJeans automates this verification process while escalating uncertain
cases for human review instead of silently guessing.

------------------------------------------------------------------------

## 💡 Our Solution

OldJeans provides an end-to-end document verification pipeline:

``` text
Shipping Inbox
      │
      ▼
Email Ingestion
      │
      ▼
AI Email Classification
      │
      ├── SI Request
      ├── Invoice Query
      ├── General
      ├── Spam
      └── BL Comparison
              │
              ▼
       Attachment Detection
              │
              ▼
      Document Type Router
       ┌────┬────┬────┐
      TXT  XLSX  PDF  DOCX
       └────┴────┴────┘
              │
              ▼
       Text Extraction
              │
              ▼
     AI Structured Extraction
              │
              ▼
       Data Normalization
              │
              ▼
        SI ↔ BL Comparison
              │
        ┌─────┼───────────┐
        ▼     ▼           ▼
       OK   MISMATCH   NEEDS_REVIEW
              │
              ▼
        Submission JSON
```

------------------------------------------------------------------------

## 🤖 AI Integration

OldJeans uses **Google Gemini 3.5 Flash-Lite** inside an n8n workflow
for two main AI tasks.

### 1. Email Classification

Incoming emails are classified into:

-   `BL_COMPARISON`
-   `SI_REQUEST`
-   `INVOICE_QUERY`
-   `GENERAL`
-   `SPAM`

The classifier analyzes the email subject, body, and attachment context
to determine the appropriate processing route.

### 2. Shipping Document Understanding

For SI and BL documents, AI converts unstructured document content into
seven standardized fields:

``` json
{
  "shipper": "...",
  "consignee": "...",
  "notify_party": "...",
  "port_of_loading": "...",
  "port_of_discharge": "...",
  "container_count": 1,
  "gross_weight_kg": 21577
}
```

The seven fields are:

1.  Shipper
2.  Consignee
3.  Notify Party
4.  Port of Loading
5.  Port of Discharge
6.  Container Count
7.  Gross Weight (kg)

AI is used for **document understanding**, while discrepancy detection
is handled by deterministic comparison logic.

------------------------------------------------------------------------

## 📄 Multi-Format Document Processing

Shipping documents arrive in different file formats, so OldJeans
contains dedicated processing paths for:

  Format   Processing Method
  -------- ----------------------------------------------
  TXT      Direct text extraction
  XLSX     Spreadsheet extraction and row normalization
  PDF      PDF text extraction
  DOCX     DOCX → ZIP → `word/document.xml` extraction

All document paths eventually produce the same standardized
representation before being sent to the AI extraction layer.

------------------------------------------------------------------------

## 🔍 Intelligent SI ↔ BL Comparison

A direct string comparison creates false mismatches because shipping
documents often represent the same information differently.

OldJeans therefore normalizes values before comparison.

### Port Normalization

The system handles variations such as:

``` text
Port Klang, Malaysia
PORT KLANG (WESTPORT), MALAYSIA (MYPKG)
MYPKG
```

### Weight Normalization

Different weight units are converted into kilograms.

``` text
22 MT       → 22000 kg
22000 KG    → 22000 kg
48501.7 LB  → approximately 22000 kg
```

### Container Normalization

Different container representations are converted into numeric counts.

``` text
2 containers
2 x 40HC
2X40'
```

become:

``` text
2
```

------------------------------------------------------------------------

## 🚨 Human-in-the-Loop Reliability

OldJeans does not assume every document can be processed reliably.

When information is missing, corrupted, unreadable, or insufficient for
a safe comparison, the system can return `NEEDS_REVIEW`.

Possible outputs include:

-   `OK` --- SI and BL agree after normalization
-   `MISMATCH` --- one or more verified fields differ
-   `NEEDS_REVIEW` --- reliable automatic verification cannot be
    completed

Every mismatch, missing-attachment, or unreadable-document case is routed to a human reviewer before any corrective action is taken, thus AI never auto-sends a correction on its own.

------------------------------------------------------------------------

## ⚙️ Technology Stack

  -----------------------------------------------------------------------
  Technology                          Purpose
  ----------------------------------- -----------------------------------
  n8n                                 Workflow orchestration

  Google Gemini 3.5 Flash-Lite        Email classification and document
                                      understanding

  JavaScript                          Normalization, comparison and
                                      workflow logic

  Docker                              Local containerized environment

  Python/FastAPI-based Inbox API      Hackathon email/document data
                                      source

  Git & GitHub                        Version control and collaboration

  GitHub Pages                        Public Human-in-the-Loop Review Desk

  Render                              Cloud deployment of Inbox API

  JSON                                Structured outputs and evaluation
                                      submission
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 🏗️ System Architecture

OldJeans separates AI reasoning from deterministic business rules.

``` text
                    ┌──────────────────┐
                    │  Shipping Inbox  │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │ Email Classifier │
                    │      Gemini      │
                    └────────┬─────────┘
                             │
                             ▼
                   ┌────────────────────┐
                   │ Attachment Router  │
                   └─────────┬──────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
             TXT          PDF/DOCX         XLSX
              │              │              │
              └──────────────┼──────────────┘
                             ▼
                  ┌─────────────────────┐
                  │ Structured AI       │
                  │ Extraction          │
                  └─────────┬───────────┘
                            │
                            ▼
                  ┌─────────────────────┐
                  │ Normalization Layer │
                  └─────────┬───────────┘
                            │
                            ▼
                  ┌─────────────────────┐
                  │ SI ↔ BL Comparator  │
                  └─────────┬───────────┘
                            │
                 ┌──────────┼──────────┐
                 ▼          ▼          ▼
                OK      MISMATCH   NEEDS_REVIEW
```

------------------------------------------------------------------------

## 📊 Evaluation Results

The final workflow successfully produced predictions for the complete
**520-email evaluation dataset**.

  Metric                                            Result
  ---------------------------------------- ---------------
  Emails Processed                           **520 / 520**
  Stage 1 Accuracy                              **90.77%**
  Classification Macro-F1                       **83.73%**
  Defect Precision                              **84.00%**
  Defect Recall                                 **91.30%**
  Defect F1                                     **87.50%**
  Field-Level F1                                **81.16%**
  Exact Match Rate                              **88.00%**
  Escalation Recall                             **95.00%**
  End-to-End Defects Successfully Caught       **30 / 46**
  Final Evaluation Score                        **75.23%**

------------------------------------------------------------------------

## 🧪 Example Output

``` json
{
  "email_004": {
    "category": "BL_COMPARISON",
    "status": "MISMATCH",
    "review_reason": null,
    "has_defect": true,
    "defect_fields": [
      "consignee",
      "notify_party"
    ]
  }
}
```

------------------------------------------------------------------------

## 🛡️ Failure Handling

During development we encountered:

-   Missing attachments
-   Corrupted or malformed PDFs
-   Empty PDF extraction results
-   Missing document fields
-   Non-SI/BL attachments
-   Inconsistent formatting
-   AI structured-output failures

OldJeans includes error paths and review states so individual failures
do not terminate the entire workflow.

------------------------------------------------------------------------

## What data the pipeline touches

The pipeline processes:

- Shipping Instruction (SI) and Bill of Lading (BL) documents attached to inbound emails
- Shipping details such as shipper/consignee company names, ports, container counts and weights
- Email metadata including sender, subject and body, used only to classify intent and route the correct handling branch
- No payment details, ID numbers, or health/personal data are processed anywhere in the pipeline.

------------------------------------------------------------------------

## Where data lives

- All processing happens inside the n8n workflow, orchestrated in a local/self-hosted environment except to the Gemini API for extraction.
- Extracted structured fields (shipper, consignee, ports, weights, etc.) are held in-memory during the workflow run and written only to the final submission JSON. 
- The HITL review UI reads only the submission output; any reviewer decisions (approve/flag) are stored locally in the reviewer's own browser (localStorage), not sent to a shared server.

------------------------------------------------------------------------

## 🧩 Key Technical Challenges

### Multi-format processing

TXT, XLSX, PDF and DOCX require different extraction strategies.
Format-specific branches converge into one common structured extraction
pipeline.

### DOCX processing in n8n

Native DOCX extraction was unavailable in our workflow environment. DOCX
files are handled as ZIP archives, extracting `word/document.xml` and
converting the XML into text.

### Preventing false mismatches

Normalization is applied to port names/codes, weight units, container
counts, capitalization and whitespace before comparison.

### Maintaining metadata

`email_id`, `document_type`, and `attachment_path` are preserved across
processing branches so each SI and BL can be paired correctly.

------------------------------------------------------------------------

## 🚀 Running the Project

### Prerequisites

-   Docker
-   n8n
-   Google Gemini API credentials

### Start the Local Environment

``` bash
docker compose up -d
```

The hackathon Inbox API is available locally on port `8080`, while n8n
is available on port `5678`.

### Import the Workflow

1.  Open n8n.
2.  Import the workflow JSON from this repository.
3.  Configure your own Google Gemini credential.
4.  Verify the Inbox API endpoint.
5.  Execute the workflow.

> API credentials are intentionally not included in this repository.

------------------------------------------------------------------------

## 📦 Submission Format

The system produces one JSON object keyed by email ID:

``` json
{
  "email_001": {
    "category": "BL_COMPARISON",
    "status": "OK",
    "review_reason": null,
    "has_defect": false,
    "defect_fields": []
  }
}
```

Each email contains `category`, `status`, `review_reason`, `has_defect`,
and `defect_fields`.

------------------------------------------------------------------------

## 🌱 Future Improvements

-   OCR and vision support for scanned shipping documents
-   Confidence-aware field extraction
-   Improved human-review precision
-   Connect the deployed Human-in-the-Loop Review Desk directly to live workflow outputs
-   Persistent reviewer decisions and audit history
-   Detailed audit trails
-   Additional shipping document types
-   Improved multilingual document understanding
-   Production email and logistics-system integration
-   Historical discrepancy analytics
-   Automated notifications for high-risk mismatches

------------------------------------------------------------------------

## 👥 Team

**Team OldJeans**

Built during the SDOC Hackathon.

The project was developed collaboratively across workflow architecture
and integration, AI email classification, multi-format document
extraction, data normalization and SI/BL comparison, and
testing/evaluation.

------------------------------------------------------------------------

## 📌 Project Status

**Hackathon Prototype**

The current prototype demonstrates an end-to-end AI-assisted shipping
document verification workflow capable of processing the complete
520-email evaluation dataset and detecting field-level discrepancies
between Shipping Instructions and Bills of Lading.

------------------------------------------------------------------------

## 🔐 Security

- No API keys or private credentials are stored in this repository.
- Users deploying the project must configure their own credentials through n8n's
credential management system.

------------------------------------------------------------------------

## 📄 License

This project was developed for hackathon and educational purposes.
