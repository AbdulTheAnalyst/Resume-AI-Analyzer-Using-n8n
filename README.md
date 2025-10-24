# 🧠 Resume AI Analyzer — n8n Workflow Documentation

## Overview
The **Resume AI Analyzer** automates the process of **collecting resumes**, **extracting key candidate data**, **analyzing CVs using AI**, and **saving structured results** into a Google Sheet.  
It integrates Gmail, Google Drive, OpenAI, and LangChain nodes for full pipeline automation.

---

## 🔁 Workflow Summary

| Step | Function | Description |
|------|-----------|--------------|
| 1 | **Trigger** | Watches for new resumes via Gmail or Slack trigger. |
| 2 | **Upload to Drive** | Stores the uploaded resume in Google Drive. |
| 3 | **Extract Text** | Reads resume PDF content. |
| 4 | **Information Extraction** | Extracts name, email, and phone number. |
| 5 | **AI Evaluation** | Uses GPT-4o to summarize, extract qualifications, and score relevance to a job. |
| 6 | **Data Structuring** | Formats the data into a consistent JSON schema. |
| 7 | **Save Results** | Appends structured results into a Google Sheet. |

---

## 🧩 Node Breakdown

### 1. **Gmail Trigger**
- **Type:** `n8n-nodes-base.gmailTrigger`
- **Purpose:** Monitors a Gmail inbox for messages containing resumes.
- **Configuration:**
  - Polls every minute.
  - Filters emails with the keyword “resume”.
  - Downloads attachments (CVs).

---

### 2. **Upload File**
- **Type:** `n8n-nodes-base.googleDrive`
- **Purpose:** Uploads resume attachments to a specific Google Drive folder.
- **Folder:** `Job Resumes`
- **Input:** `cv0` (file from Gmail)
- **Output:** Google Drive file ID.

---

### 3. **Extract from File**
- **Type:** `n8n-nodes-base.extractFromFile`
- **Operation:** `pdf`
- **Purpose:** Extracts text content from the uploaded PDF resume.
- **Output:** JSON containing text data.

---

### 4. **Information Extractor**
- **Type:** `@n8n/n8n-nodes-langchain.informationExtractor`
- **Model:** `GPT-4o`
- **Schema:** Extracts basic contact information.
  ```json
  {
    "type": "object",
    "properties": {
      "name": { "type": "string" },
      "email": { "type": "string", "format": "email" },
      "phone_number": { "type": "string", "pattern": "^(\\+\\d{1,3}[- ]?)?\\d{10}$" }
    },
    "required": ["name", "email", "phone_number"]
  }
  ```

---

### 5. **AI Agent (LangChain)**
- **Type:** `@n8n/n8n-nodes-langchain.agent`
- **Purpose:** Evaluates resumes against a job description using structured reasoning.
- **Prompt Summary:**
  - Extract educational qualifications, job history, and skill set.
  - Compare candidate profile to the job role.
  - Assign a **relevance score (1–10)**.
  - Provide **justification** for the score.
- **Outputs:**
  - `educationalQualifications`
  - `jobHistory`
  - `skillSet`
  - `score`
  - `justification`

---

### 6. **Edit Fields**
- **Type:** `n8n-nodes-base.set`
- **Purpose:** Normalizes and renames fields before processing.

---

### 7. **Code Node**
- **Type:** `n8n-nodes-base.code`
- **Language:** JavaScript
- **Purpose:** Extracts sections of the AI response using RegEx and formats as JSON.
- **Outputs:**
  ```json
  {
    "educationalQualifications": "",
    "jobHistory": "",
    "skillSet": "",
    "score": "",
    "justification": ""
  }
  ```

---

### 8. **Append Row in Google Sheet**
- **Type:** `n8n-nodes-base.googleSheets`
- **Purpose:** Appends candidate results into a Google Sheet.
- **Target Sheet:** `List of Candidates`
- **Columns:**
  - Name
  - Email
  - Number
  - Skill Set
  - Qualifications
  - Job History
  - Score
  - Justification

---

### 9. **Optional Triggers**
- **Slack Trigger:** Handles resumes shared via Slack (channel event: `file_share`).
- **Form Trigger:** Allows resume uploads through a public form titled *“Job Application”*.

---

## ⚙️ Data Flow Diagram (Simplified)

```
[Gmail Trigger] 
      ↓
[Upload File → Google Drive] 
      ↓
[Extract from File → PDF Text] 
      ↓
[Information Extractor] 
      ↓
[AI Agent → Resume Evaluation] 
      ↓
[Code → JSON Structuring] 
      ↓
[Google Sheets → Append Results]
```

---

## 🧾 Output Example

```json
{
  "Name": "John Doe",
  "Email": "johndoe@gmail.com",
  "Number": "+2348012345678",
  "Skill Set": "Python, SQL, Machine Learning",
  "Qualifications": "B.Sc. Computer Science - University of Lagos (2018)",
  "Job History": "Data Analyst - XYZ Tech (2019–2023)",
  "Score": "8",
  "Justification": "Strong technical background with relevant experience; minor gap in leadership skills."
}
```

---

## ✅ Integration Summary
| Platform | Purpose |
|-----------|----------|
| **Gmail** | Resume intake |
| **Google Drive** | Resume storage |
| **LangChain + OpenAI** | AI analysis & summarization |
| **Google Sheets** | Result storage |
| **Slack/Form** | Alternative input methods |

---

## 🔒 Credentials Used
- Google Drive OAuth2
- Gmail OAuth2
- Google Sheets OAuth2
- OpenAI API key
- Slack API key

---

## 🧩 Version Info
- **Workflow Version:** v4.6
- **Execution Order:** Sequential (v1)
- **Active:** No (requires manual activation)
- **Created in:** n8n Automation Platform
