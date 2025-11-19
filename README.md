# Grant Email Triage – README

## Overview

This project implements an automated grant-email triage workflow using **n8n**, **LLM-powered classification**, **entity extraction**, and **Gmail** integration.
The workflow processes incoming messages, classifies them into meaningful categories, extracts structured entities, and automatically replies to **New Application** emails.

The exported workflow is included in `workflow.json`.

---

## Architecture

### Components

**1. n8n (Workflow Engine)**
All orchestration happens inside n8n. It manages:

* Email intake
* LLM-based text classification
* Entity extraction (organization, contact person, project title)
* Email drafting
* Auto-replies (only for New Application inquiries)

**2. Gmail Integration**
The workflow uses Gmail for both:

* Reading new incoming emails (`Gmail Trigger`)
* Sending auto-reply messages (`Gmail` Send node)

**3. LLM Processing (Groq + Google Gemini)**
The workflow uses two different LLM providers directly through n8n nodes:

* **Groq-powered LangChain Classifier** – determines the email’s intent category
* **Information Extractor nodes** – extract essential fields from the email text
* **Google Gemini (“Message a Model” node)** – generates the final reply email body

This satisfies the case-study requirement to integrate an LLM of your choice.

---

## Workflow Logic

### 1. Trigger: Gmail Email Intake

The workflow starts when the **Gmail Trigger** detects a new incoming email.
It captures the subject, plaintext body, sender, and metadata.

### 2. LLM-Powered Classification

A **LangChain Text Classifier** (Groq backend) analyzes the subject and body and classifies the email as:

* **New Application**
* **Status Update**
* **General Question**
* **Irrelevant**

This classification determines the workflow’s routing.

### 3. Entity Extraction

Several **Information Extractor** nodes process the email and extract:

* **Contact Person**
* **Organization Name**
* **Project Title**

Each entity node outputs clean, structured JSON.

### 4. Auto-Reply Generation (New Application only)

If the email is classified as **New Application**:

1. Extracted entities and relevant context are passed to **Google Gemini** via the `Message a Model` node.
2. The model generates a polished, professional reply acknowledging the inquiry and providing the application portal link.
3. The reply is sent back using the **Gmail Send** node.

### 5. Other Categories

If the email is classified as:

* Status Update
* General Question
* Irrelevant

…the workflow does not send a reply.
Instead, it logs the extracted information and continues.

---

## Assumptions

* The Gmail account used for triggering and sending emails is dedicated to this workflow.
* Incoming emails contain identifiable plaintext body content (`message.text`).
* The LLM prompts ensure clean extraction and prevent hallucinated data.
* API keys for Groq, Gemini, and Gmail are configured inside n8n’s credentials manager.
* Application URL and other constants are stored as n8n environment variables.

---

## Setup Instructions

### Requirements

* An n8n instance (local or cloud-hosted such as Azure App Service Container)
* Gmail OAuth2 credentials
* Groq API key (for LangChain classifier)
* Google Gemini API key (for email generation)

### 1. Import the Workflow

1. Open **n8n → Workflows**
2. Click **Import**
3. Upload the included `workflow.json` file

### 2. Configure Credentials in n8n

Configure the following:

| Node                     | Credential Needed |
| ------------------------ | ----------------- |
| Gmail Trigger            | Gmail OAuth2      |
| Gmail Send               | Gmail OAuth2      |
| Text Classifier (Groq)   | Groq API Key      |
| Message a Model (Gemini) | Gemini API Key    |

### 3. Environment Variables (Optional)

Inside n8n (Settings → Variables):

```
APPLICATION_PORTAL_URL=https://example.org/apply
```

### 4. Activate the Workflow

Once activated:

* New emails automatically trigger processing
* New Application emails get an auto-reply
* All other categories are logged

---

## Repository Contents

| File                         | Description                                            |
| ---------------------------- | ------------------------------------------------------ |
| `README.md`                  | Documentation for architecture, setup, and assumptions |
| `workflow.json`              | Exported n8n workflow used in this project             |
| `azure-function/` (optional) | Placeholder if you later add custom Azure Functions    |

---

## Possible Extensions

* Replace in-workflow LLM calls with a centralized Azure Function API (recommended for production).
* Add persistent logging via PostgreSQL or Azure Cosmos DB.
* Introduce human-in-the-loop approval for borderline classifications.
* Add HTML email templates for richer replies.
