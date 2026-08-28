# Python Study Coach — RAG AI Agent

- **Author:** Yumna Kashif
- **Program:** FlyRank AI Fluency
- **Track:** AI Fluency
- **Project Type:** AI Study Agent
- **Platform:** n8n
- **Focus:** Python Foundations · RAG · Agent Evaluation

> **Personal Python notes → retrieval-augmented tutoring → evaluated beginner-friendly learning support**

---

## 🔗 Project Links

* 💻 **[GitHub Repository](https://github.com/yumna-09/Python-Study-Coach)**
* ⚙️ **[Main Workflow](https://github.com/yumna-09/Python-Study-Coach/blob/main/workflow/python-study-coach.json)**
* 🧪 **[Evaluation Workflow](https://github.com/yumna-09/Python-Study-Coach/blob/main/workflow/python-study-coach-evaluation.json)**

---

## Overview

Python Study Coach is an interactive AI learning agent designed to help beginner Python learners understand concepts, practise problems, and learn from their mistakes.

Rather than behaving like an answer-only chatbot, the agent follows a **teaching-first approach**. It explains concepts in beginner-friendly language, uses small practical examples, provides hints before complete solutions when appropriate, reviews mistakes, and encourages the learner to attempt problems independently.

The agent is connected to a personal Python study-notes knowledge base stored in Google Drive. The notes are loaded into a retrieval pipeline using OpenAI embeddings and an in-memory vector store, allowing the agent to retrieve relevant learning material when answering questions.

When the notes do not contain the required information, the agent can fall back to general Python knowledge without falsely claiming that the answer came from the notes.

---

## The Problem

The question guiding this project was:

> **How can an AI study assistant help a beginner learn Python without simply giving away every answer?**

General-purpose AI assistants can explain programming concepts, but they are not necessarily designed around a learner's current material or learning process.

For a study coach, useful behaviour means more than returning technically correct code. The agent should also:

* explain *why* something works;
* identify mistakes without replacing the learner's thinking;
* use the learner's own notes when relevant;
* provide hints before full solutions when appropriate;
* stay within the intended learning scope.

Python Study Coach was built around these requirements.

---

## Who Is This For?

This project is intended for:

* Beginner Python learners
* Students building programming foundations
* Learners who want explanations rather than answer-only responses
* Students who want an AI tutor connected to their own study material

The current version is intentionally scoped to **Python Foundations** rather than acting as a general-purpose programming assistant.

---

# Results at a Glance

| Result | Value |
| --- | ---: |
| Evaluation test cases | **10** |
| Evaluation method | **AI-based Correctness** |
| Correctness scale | **1–5** |
| Final V2 correctness | **4.40 / 5.00** |
| Knowledge source | **Personal Python study notes** |
| Retrieval approach | **RAG** |
| Primary agent model | **Google Gemini** |

The final corrected V2 evaluation produced an average **Correctness score of 4.40 / 5.00** across the 10-question Python Foundations benchmark.

The evaluation compares the Study Coach's generated answer against a reference answer for each test case.

---

# Project Workflow

```text
Learner
   │
   ▼
n8n Chat Trigger
   │
   ▼
Python Study Coach AI Agent
   │
   ├──────────────► Google Gemini Chat Model
   │
   │
   └──────────────► Study Notes Retrieval Tool
                         │
                         ▼
                  Simple Vector Store
                         ▲
                         │
                  OpenAI Embeddings
                         ▲
                         │
                  Python Study Notes
                         ▲
                         │
                     Google Drive
```

---

# Architecture and Design

The project contains two connected processes: the **knowledge pipeline** and the **interactive tutoring pipeline**.

## Knowledge Pipeline

```text
Google Drive
     │
     ▼
Python Study Notes
     │
     ▼
Default Data Loader
     │
     ▼
OpenAI Embeddings
     │
     ▼
Simple Vector Store
```

The study notes are downloaded from Google Drive, loaded as a document, converted into embeddings, and inserted into an in-memory vector store.

## Tutoring Pipeline

```text
Learner Question
      │
      ▼
   AI Agent
   │      │
   │      └────► Gemini Chat Model
   │
   └───────────► Vector Store Retrieval Tool
                       │
                       ▼
                Relevant Study Notes
                       │
                       ▼
                Tutor Response
```

The vector store is exposed to the AI Agent as a retrieval tool with instructions to search the learner's Python notes.

This allows the agent to combine conversational reasoning with retrieved study material when relevant.

---

# Agent Behaviour

The Study Coach is instructed to:

1. Explain Python concepts in simple, beginner-friendly language.
2. Match the learner's current level.
3. Use small, practical examples.
4. Allow the learner to attempt practice questions before revealing complete solutions when appropriate.
5. Prefer hints and explanations over immediate answers.
6. Identify syntax, logic, and conceptual errors in submitted code.
7. Explain why an error occurred.
8. Point out what the learner did correctly as well as what needs improvement.
9. Gradually increase difficulty as understanding improves.
10. Stay focused on Python learning.

For knowledge grounding, the agent is instructed to use the connected study notes as the primary source when they are available and relevant.

It must **not pretend that information came from the notes when it did not**.

---

# Tech Stack

| Component | Role |
| --- | --- |
| **n8n** | Workflow orchestration and agent interface |
| **Google Gemini** | Primary language model for the Study Coach |
| **OpenAI Embeddings** | Embedding generation for retrieval |
| **Google Drive** | Source for the Python study-notes file |
| **Simple Vector Store** | In-memory storage and retrieval |
| **n8n Evaluations** | Dataset-driven agent evaluation |
| **OpenAI Chat Model** | LLM judge for AI-based correctness |

---

# Setup

The repository contains exported n8n workflows so the project can be reproduced with your own credentials and study material.

## Prerequisites

You will need:

* n8n
* Google Gemini credentials
* OpenAI credentials
* Google Drive access
* A Python study-notes document

For the evaluation workflow, you will also need access to a supported LLM for AI-based correctness scoring.

---

## 1. Clone the Repository

```bash
git clone https://github.com/yumna-09/Python-Study-Coach.git
cd Python-Study-Coach
```

---

## 2. Import the Main Workflow

Open n8n and import:

```text
workflow/python-study-coach.json
```

After importing, reconnect the required credentials because exported workflows do not provide reusable access to the original accounts.

---

## 3. Configure the Gemini Model

Open the **Google Gemini Chat Model** node.

Connect your own Gemini credentials and select a compatible Gemini chat model available in your n8n environment.

The Gemini model provides the primary language-model capability for the Study Coach.

---

## 4. Configure OpenAI Embeddings

Open the **Embeddings OpenAI** node and connect your own OpenAI credentials.

The embeddings node is used by both sides of the retrieval pipeline:

```text
Study Notes → Vector Store

and

Vector Store → AI Agent Retrieval
```

---

## 5. Connect Your Study Notes

Open the **Download file** Google Drive node.

Connect your own Google Drive account and select the document containing your Python study notes.

The original implementation uses a Markdown study-notes file.

A reproducing user should select their own document rather than relying on the original development Drive resource.

---

## 6. Verify the Vector Store

The insertion side should:

1. download the study-notes file;
2. load the document;
3. generate embeddings;
4. insert the embedded content into the Simple Vector Store.

The retrieval-side Simple Vector Store should be connected to the AI Agent as a tool.

Both vector-store nodes should use the same memory key so the agent can retrieve the indexed material.

---

## 7. Test the Agent

Open the n8n chat interface and try questions such as:

```text
What is a variable in Python?
```

```text
Explain a for loop with an example.
```

```text
What is the difference between a list and a tuple?
```

You can also ask the coach for a practice problem or provide your own Python code for review.

---

# Usage

The intended learning flow is:

```text
Ask a Python Question
        ↓
Agent Interprets the Request
        ↓
Retrieve Relevant Notes When Available
        ↓
Generate Beginner-Friendly Explanation
        ↓
Example / Hint / Feedback
        ↓
Encourage Learner Attempt
```

For example, when asked about a Python concept, the agent may explain the concept, provide a small example, and finish with a question that checks understanding.

When asked for practice, the agent is designed to avoid immediately revealing the complete solution where doing so would remove the learning opportunity.

---

# V2 Evaluation

A separate evaluation workflow was created to test the final Study Coach against a small Python Foundations benchmark.

The dataset contains **10 test cases** with three fields:

```text
input
output
expected_output
```

Where:

* `input` contains the benchmark question;
* `output` contains the generated Study Coach response;
* `expected_output` contains the reference answer.

## Evaluation Workflow

```text
Python Evaluation Dataset
          │
          ▼
When Fetching a Dataset Row
          │
          ▼
       AI Agent
          │
          ▼
         Wait
          │
          ▼
Evaluation — Set Outputs
          │
          ▼
Evaluation — Set Metrics
          │
          ▼
AI-Based Correctness Judge
```

The final configuration maps:

```text
Expected Answer → expected_output
Actual Answer   → AI Agent output
```

This ensures that the evaluator compares the generated response against the intended reference response rather than comparing two generated values.

---

## Final V2 Result

| Metric | Result |
| --- | ---: |
| Test cases | **10** |
| Metric | **Correctness (AI-based)** |
| Scale | **1–5** |
| **Average Correctness** | **4.40 / 5.00** |

The final corrected evaluation run achieved an average **4.40 / 5.00** correctness score.

This result was recorded after correcting the evaluation mapping so that the benchmark's `expected_output` was used as the expected answer and the AI Agent's generated `output` was used as the actual answer.

---

## What the Evaluation Revealed

The agent performed strongly across foundational Python questions, but the evaluation also exposed an important trade-off.

A reference answer may expect the complete solution immediately.

The Study Coach, however, is deliberately instructed to sometimes **teach first and let the learner attempt the solution**.

For example, when asked to write a function, the coach may explain `def` and parameters and ask the learner to attempt the implementation instead of immediately returning the finished function.

That behaviour may differ from a strict reference answer even though it is consistent with the tutoring objective.

---

# Project Structure

```text
Python-Study-Coach/
│
├── workflow/
│   ├── python-study-coach.json
│   └── python-study-coach-evaluation.json
│
└── README.md
```

### `python-study-coach.json`

The main interactive RAG Study Coach workflow.

### `python-study-coach-evaluation.json`

The evaluation-enabled workflow containing the dataset trigger, output mapping, and AI-based Correctness metric.

---

# Limitations

This project has several important limitations.

## 1. In-Memory Vector Storage

The current implementation uses n8n's Simple Vector Store.

This is suitable for a prototype, but a persistent vector database would be more appropriate for a production system or a larger knowledge base.

## 2. Limited Learning Scope

The agent is intentionally designed around **Python Foundations**.

It has not been evaluated as a general-purpose programming tutor or an advanced Python assistant.

## 3. Retrieval Depends on the Study Notes

The quality and coverage of note-grounded answers depend on the information contained in the connected study material.

If relevant information is missing, the agent may use general Python knowledge instead.

## 4. General-Knowledge Fallback Is Not Note-Grounded

When the agent falls back to general Python knowledge, the answer is no longer grounded in the personal knowledge base.

The system prompt therefore explicitly prevents the agent from claiming that unsupported information came from the notes.

## 5. Tutoring Behaviour Can Differ From Reference Answers

The agent's practice-first behaviour may intentionally avoid immediately giving a full solution.

This can create differences between a pedagogically useful response and a benchmark's expected answer.

## 6. AI-Based Evaluation Is Not Ground Truth

The V2 Correctness metric uses an LLM judge.

It provides a systematic evaluation signal, but the score can still depend on the evaluator model, rubric, and generated responses.

---

# Key Design Decision

A major design decision was to make the agent **teaching-first rather than answer-first**.

The easiest version of a Python chatbot would simply return the requested code.

That was deliberately avoided.

For practice-oriented requests, the Study Coach can explain the relevant concept, provide a hint, and allow the learner to attempt the problem before revealing the complete solution.

This keeps the learner active in the problem-solving process rather than turning the agent into an automatic answer generator.

---

# Guardrails

The workflow includes several behavioural guardrails.

The agent is instructed to:

* stay focused on Python learning;
* avoid unnecessarily advanced explanations;
* distinguish between retrieved study material and general knowledge;
* never falsely attribute unsupported information to the learner's notes;
* avoid irreversible actions on files or external services without explicit confirmation.

These guardrails are implemented through the agent's system instructions rather than through a separate moderation layer.

---

# Security & Reproducibility

API keys and access tokens should **never be committed to this repository**.

The exported n8n JSON files may contain references to credential names, workflow resources, or development-environment identifiers, but another user must connect their own accounts after importing the workflow.

The Google Drive document used during development is also not intended to act as a public credential or shared private knowledge source.

Before publishing future workflow exports, the files should always be checked for exposed secrets.

---

# Future Improvements

Future versions could:

* replace the in-memory vector store with persistent storage;
* expand the evaluation benchmark beyond 10 questions;
* test debugging and code-review behaviour separately;
* add retrieval-specific evaluation;
* measure teaching quality separately from answer correctness;
* introduce learner progress tracking;
* adapt question difficulty based on performance;
* support a larger structured Python knowledge base;
* evaluate multi-turn tutoring conversations.

---

# AI Transparency

AI tools were used during the project for:

* workflow development support;
* debugging;
* evaluation setup;
* documentation;
* iteration.

The final workflow configuration, evaluation mappings, testing decisions, limitations, and project presentation were reviewed during development rather than treating generated suggestions as automatically correct.

---

# Repository

This repository contains the exported workflows and documentation for the Python Study Coach.

The project demonstrates an end-to-end agent workflow:

```text
Learning Problem
      ↓
Agent Behaviour Design
      ↓
Knowledge Connection
      ↓
RAG Retrieval
      ↓
Interactive Tutoring
      ↓
Evaluation Dataset
      ↓
AI-Based Correctness Evaluation
      ↓
Documented Agent
```

---

# Acknowledgments

This project was completed as part of the **FlyRank AI Fluency Track**.

The Python Study Coach was developed as a practical AI agent project, bringing together workflow orchestration, retrieval-augmented generation (RAG), external knowledge integration, guardrails, and evaluation in a working n8n system.

The project reflects the learning and iteration completed throughout the track, from defining the agent's purpose and connecting a real knowledge source to testing its behaviour and building a V2 evaluation workflow.

Thank you to **FlyRank** and the AI Fluency program for the learning framework, resources, and project structure that supported the development of this agent.

The final workflow design, implementation decisions, testing, evaluation, limitations, and documentation represent my work completed during the track.
