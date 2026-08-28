# Python Study Coach — RAG AI Agent

> An interactive AI study coach built in n8n that helps beginner Python learners understand concepts, practise problems, and learn from their mistakes using retrieval-augmented generation (RAG).

## Overview

Python Study Coach is an AI-powered learning agent designed for beginner-level Python study.

Instead of acting as a simple question-answering chatbot, the agent is designed to behave like a tutor: it explains concepts in beginner-friendly language, uses small practical examples, provides hints before complete solutions when appropriate, identifies mistakes in submitted code, and encourages the learner to attempt problems independently.

The agent also connects to a personal Python study-notes knowledge base stored in Google Drive. The notes are loaded, embedded, stored in an in-memory vector store, and exposed to the agent as a retrieval tool. This allows the agent to ground relevant responses in the learner's own study material while retaining general Python knowledge as a fallback.

The project was built as an interactive RAG agent in **n8n** and includes a separate evaluation path for testing generated responses against reference answers.

---

## Who It Is For

This project is designed primarily for:

- beginner Python learners building strong programming foundations;
- students who want explanations rather than answer-only responses;
- learners practising concepts such as variables, loops, functions, lists, tuples, dictionaries, conditionals, and string operations;
- students who want an AI tutor connected to their own learning material.

The current implementation is intentionally scoped to **Python Foundations** rather than attempting to serve as a general-purpose programming assistant.

---

## What the Agent Does

The Study Coach can:

- explain Python concepts in simple language;
- provide small, practical code examples;
- generate practice questions;
- encourage the learner to attempt a problem before revealing a full solution;
- review submitted code for syntax, logic, and conceptual mistakes;
- explain why an error occurred instead of only correcting it;
- identify both correct and incorrect parts of a learner's attempt;
- gradually increase difficulty as understanding improves;
- retrieve relevant information from connected Python study notes;
- fall back to general Python knowledge when the notes do not contain the required information.

A core guardrail prevents the agent from claiming that information came from the learner's notes when it did not.

---

## Architecture

```text
                         ┌──────────────────────┐
                         │   Learner / Chat UI  │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │      AI Agent        │
                         │ Python Study Coach   │
                         └───────┬──────┬───────┘
                                 │      │
                  Language Model │      │ Retrieval Tool
                                 │      │
                                 ▼      ▼
                        ┌────────────┐  ┌─────────────────┐
                        │   Gemini   │  │  Vector Store   │
                        │ Chat Model │  │   Retrieval     │
                        └────────────┘  └────────┬────────┘
                                               │
                                               ▼
                                      ┌──────────────────┐
                                      │ OpenAI Embeddings│
                                      └────────┬─────────┘
                                               │
                                               ▼
                                      ┌──────────────────┐
                                      │ Python Study     │
                                      │ Notes            │
                                      │ Google Drive     │
                                      └──────────────────┘
```

### Evaluation Path

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
 OpenAI LLM Judge
 (AI-based Correctness)
```

The production/chat path and evaluation path share the same Study Coach agent so the evaluated behaviour reflects the agent being demonstrated.

---

## Tech Stack

| Component | Purpose |
|---|---|
| **n8n** | Agent orchestration and workflow automation |
| **Google Gemini** | Primary chat model used by the Study Coach |
| **OpenAI Embeddings** | Vector embeddings for study-note retrieval |
| **Google Drive** | Source for the Python study-notes knowledge base |
| **n8n Simple Vector Store** | In-memory storage and retrieval of embedded notes |
| **n8n Evaluation** | Dataset-driven evaluation and metric collection |
| **OpenAI Chat Model** | LLM judge for AI-based correctness evaluation |

---

## How the RAG Pipeline Works

The knowledge pipeline has two responsibilities: **indexing** and **retrieval**.

### 1. Load the study material

A Google Drive node downloads the Markdown file containing the Python study notes.

### 2. Prepare the document

The Default Data Loader converts the downloaded binary file into a document that can be processed by the vector pipeline.

### 3. Create embeddings

OpenAI Embeddings convert the study-note content into numerical vector representations.

### 4. Store the notes

The embedded documents are inserted into n8n's Simple Vector Store using a shared vector-store memory key.

### 5. Retrieve relevant material

A second Simple Vector Store node is configured as an AI Agent tool. When a learner asks a relevant question, the agent can retrieve material from the indexed Python notes.

### 6. Generate the response

The Study Coach combines the learner's request, its tutoring instructions, and retrieved context when available to generate the final response.

---

## Agent Behaviour

The agent's system instructions intentionally prioritize **teaching over answer generation**.

The main behavioural rules are:

1. Explain concepts at beginner level.
2. Avoid unnecessarily jumping to advanced material.
3. Prefer small and practical examples.
4. Let the learner attempt practice problems before giving complete solutions when appropriate.
5. Explain syntax, logic, and conceptual mistakes.
6. Highlight what the learner did correctly as well as what needs improvement.
7. Use connected study notes as the primary source when relevant.
8. Never pretend unsupported information came from the notes.
9. Stay focused on Python learning.
10. Avoid irreversible external actions without explicit confirmation.

This design makes the agent closer to an interactive tutor than a conventional answer bot.

---

## Example Usage

### Concept explanation

**Learner**

> What is the difference between a list and a tuple?

**Study Coach behaviour**

The agent explains that lists are mutable while tuples are immutable, shows their `[]` and `()` syntax, gives small examples, and may finish with a conceptual check question.

### Practice-first coaching

**Learner**

> Write a simple function that adds two numbers.

**Study Coach behaviour**

Rather than immediately completing the exercise for the learner, the coach can explain `def`, parameters, and the goal of the function before asking the learner to attempt the implementation.

### Code review

A learner can submit Python code and ask why it is not working. The Study Coach is instructed to identify syntax, logic, or conceptual problems and explain the reason behind the error rather than only replacing the code.

---

# V2 Evaluation

The final agent was tested using a **10-question Python Foundations evaluation dataset**.

Each test row contained:

```text
input
output
expected_output
```

Where:

- `input` is the benchmark question;
- `output` is the response generated by the Study Coach;
- `expected_output` is the reference response used for comparison.

The evaluation workflow feeds each dataset input through the same AI Agent and writes the generated response back as the workflow output. A separate metrics stage then compares the generated answer against the reference answer.

### Final Result

| Metric | Result |
|---|---:|
| Test cases | 10 |
| Evaluation method | AI-based Correctness |
| Scale | 1–5 |
| **Average Correctness** | **4.40 / 5.00** |

The final corrected evaluation run achieved an average **Correctness score of 4.40/5.00**.

The evaluator uses the dataset's `expected_output` as the expected answer and the Study Coach's generated `output` as the actual answer.

### What the Evaluation Showed

The benchmark indicates strong alignment between the agent's explanations and the reference answers across foundational Python concepts.

The evaluation also highlights an important characteristic of the system: **reference-answer similarity and tutoring quality are not always identical goals**.

For example, when directly asked to write a function, the Study Coach may intentionally guide the learner toward attempting the solution rather than immediately returning the complete implementation. That behaviour follows the tutoring design but can differ from a reference answer that contains the final code.

This is a deliberate trade-off between maximizing benchmark similarity and encouraging active learning.

---

## Design Decision: Teaching Before Answering

One of the most important design decisions was to make the agent **practice-first rather than answer-first**.

A normal chatbot can immediately return a finished solution. For a learning agent, that behaviour can reduce the learner's opportunity to reason through the problem.

The Study Coach therefore prefers hints, explanations, and learner attempts before complete solutions when appropriate.

This decision makes the interaction more educational, but it can also lower strict reference-answer evaluation scores when a benchmark expects an immediate final answer.

---

## Limitations

The current version has several intentional and technical limitations.

**In-memory vector storage.** The project uses n8n's Simple Vector Store. It is appropriate for this prototype, but a persistent vector database would be more suitable for a production system.

**Narrow subject scope.** The agent is designed specifically for Python Foundations. Its prompts and knowledge source are not intended to provide comprehensive tutoring across every programming language or advanced computer-science topic.

**Retrieval depends on the available notes.** The quality and coverage of note-grounded responses depend on what is present in the connected study material.

**General-knowledge fallback.** When the notes do not contain relevant material, the agent can use general Python knowledge. Although the prompt explicitly prevents it from falsely attributing that information to the notes, the response is no longer grounded in the personal knowledge base.

**Practice-first responses can differ from reference answers.** The agent may deliberately avoid immediately giving a full solution when the learning objective is better served by allowing the learner to attempt it first.

**LLM-based evaluation is not a deterministic correctness proof.** The V2 Correctness metric is generated by an AI evaluator. It is useful for systematic comparison, but scores can still depend on the evaluator model and rubric.

---

## Setup

The repository contains exported n8n workflow files. Credentials are not bundled as usable secrets, so anyone reproducing the project must connect their own accounts and API credentials.

### Prerequisites

You will need:

- an n8n workspace or self-hosted n8n instance;
- access to a Google Gemini model supported by your n8n installation;
- OpenAI credentials for embeddings;
- a Google Drive account;
- a Markdown or supported document containing your Python study notes.

The evaluation workflow additionally requires an LLM connection for the AI-based correctness evaluator.

### 1. Clone the repository

```bash
git clone <YOUR-REPOSITORY-URL>
cd python-study-coach
```

Alternatively, download the repository as a ZIP file.

### 2. Import the main workflow

In n8n:

1. create or open a project;
2. choose the option to import a workflow from a file;
3. import:

```text
workflow/python-study-coach.json
```

### 3. Configure the Gemini Chat Model

Open the **Google Gemini Chat Model** node and connect your own Gemini credentials.

If the exact model used in the exported workflow is unavailable in your n8n environment, select an available compatible Gemini chat model and test the workflow before continuing.

### 4. Configure OpenAI Embeddings

Open the **Embeddings OpenAI** node and connect your own OpenAI credentials.

The embeddings node must remain connected to both the indexing and retrieval sides of the vector-store setup.

### 5. Connect Google Drive

Open the **Download file** node and connect your own Google Drive account.

Select the document that contains the study material you want the agent to use.

The original project uses a Markdown file of Python study notes. A reproducing user should select their own file rather than relying on the original private Drive connection.

### 6. Verify the indexing pipeline

Confirm the following path is connected:

```text
Google Drive
    ↓
Simple Vector Store — Insert
    ↑
Default Data Loader
    ↑
OpenAI Embeddings
```

The exact visual arrangement can differ; the important part is that the downloaded study material is loaded, embedded, and inserted into the vector store.

### 7. Verify retrieval

The retrieval Simple Vector Store should be configured as a tool for the AI Agent and use the same vector-store memory key as the insertion side.

Its tool description should clearly tell the agent that it retrieves the connected Python study notes.

### 8. Test the agent

Open the n8n chat interface and try questions such as:

```text
What is a variable in Python?
```

```text
Explain a for loop with an example.
```

```text
Give me a practice question about dictionaries.
```

Also test a question that requires the connected study material to confirm that retrieval is working.

---

## Reproducing the Evaluation

The repository also contains:

```text
workflow/python-study-coach-evaluation.json
```

Import this workflow into n8n separately if you want to reproduce the evaluation configuration.

Create an evaluation Data Table with the following fields:

```text
input
output
expected_output
```

Add benchmark questions to `input` and reference answers to `expected_output`.

The evaluation flow should use:

```text
When fetching a dataset row
        ↓
AI Agent
        ↓
Evaluation — Set Outputs
        ↓
Evaluation — Set Metrics
```

Configure the mappings so that:

```text
Set Outputs
Name:  output
Value: generated AI Agent output
```

and:

```text
Correctness — Expected Answer:
expected_output from the evaluation dataset

Correctness — Actual Answer:
output generated by the AI Agent
```

Connect an LLM to the metrics node for the AI-based Correctness evaluation, then run the test from n8n's evaluation interface.

---

## Repository Structure

```text
python-study-coach/
│
├── README.md
│
└── workflow/
    ├── python-study-coach.json
    └── python-study-coach-evaluation.json
```

`python-study-coach.json` contains the primary interactive Study Coach workflow.

`python-study-coach-evaluation.json` contains the dataset-driven V2 evaluation configuration and AI-based correctness metric.

---

## Security & Credentials

No API keys should be committed to this repository.

The exported n8n workflows may contain credential references or resource identifiers from the development environment, but a new user must configure their own credentials after importing the workflow.

Before committing future workflow exports, always verify that no API keys, access tokens, private documents, or other secrets have been included.

---

## Future Improvements

Potential V2+ improvements include:

- replace the in-memory vector store with persistent vector storage;
- expand the evaluation dataset beyond foundational definition questions;
- add tests for debugging, code review, and multi-turn tutoring;
- evaluate retrieval quality separately from answer correctness;
- introduce additional metrics for teaching quality and hint usefulness;
- support learner progress tracking;
- adapt difficulty based on demonstrated performance;
- expand the knowledge base while preserving clear source attribution.

---

## Project Status

**Current version:** Working prototype with RAG retrieval and evaluation pipeline.

The core Study Coach can run interactively, retrieve from connected Python study notes, and be tested through a dataset-driven evaluation workflow.

---

## Author

**Yumna Kashif**

Built as an AI agent project focused on practical workflow orchestration, retrieval-augmented generation, evaluation, and human-centered learning design.
