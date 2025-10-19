# Multi-Agent Financial Research & Risk Intelligence Platform

[![Python](https://img.shields.io/badge/Python-3.12-blue.svg)](https://www.python.org/)
[![LangGraph](https://img.shields.io/badge/LangGraph-0.6-green.svg)](https://github.com/langchain-ai/langgraph)
[![Tests](https://img.shields.io/badge/tests-347%20passing-brightgreen.svg)](#testing)
[![FinanceBench](https://img.shields.io/badge/FinanceBench-72.7%25-blue.svg)](#evaluation)

A multi-agent retrieval-augmented generation (RAG) platform for financial document analysis and risk-aware research. The system combines selective agentic retrieval, cross-encoder reranking, role-based access control, human-in-the-loop approval, persistent workflow state, and LLM observability.

The platform is evaluated on the public FinanceBench benchmark and achieves **72.7% correctness pass rate (109/150 questions)**.

---

## Overview

Financial research requires reliable retrieval, multi-step reasoning, access controls, and traceable outputs. This project addresses these requirements through a stateful multi-agent workflow built with LangGraph.

The system can route straightforward questions through a direct retrieval pipeline while sending research-intensive questions through a multi-step workflow that decomposes the task, gathers supporting evidence, evaluates retrieval sufficiency, and synthesizes the final response.

### Core capabilities

* Multi-agent financial research workflows
* Selective routing between direct and research-intensive queries
* Vector retrieval with cross-encoder reranking
* Role-based document access control
* Human-in-the-loop approval for high-stakes responses
* Persistent workflow state and checkpointing
* Conversation-aware follow-up queries
* LLM tracing and observability
* Automated retrieval and answer evaluation
* Containerized local deployment

---

## Architecture

```text
                         User Query
                              │
                              ▼
                    ┌───────────────────┐
                    │   Query Router    │
                    └─────────┬─────────┘
                              │
                 ┌────────────┴────────────┐
                 │                         │
                 ▼                         ▼
          Direct Retrieval          Research Workflow
                 │                         │
                 │                   Query Decomposition
                 │                         │
                 │                   Evidence Retrieval
                 │                         │
                 │                  Relevance Grading
                 │                         │
                 └────────────┬────────────┘
                              ▼
                       Answer Synthesis
                              │
                              ▼
                      Grounding Check
                              │
                              ▼
                    High-Stakes Output?
                         │          │
                        No         Yes
                         │          │
                         ▼          ▼
                      Response   HITL Approval
                                    │
                                    ▼
                                  Response
```

The workflow is implemented as a stateful LangGraph graph. Research-intensive queries are decomposed into sub-questions, retrieved independently, evaluated for evidence sufficiency, and synthesized into a final response.

Access control is enforced during retrieval through document-level metadata filtering.

---

## Financial Research Workflow

### Direct queries

```text
Query → Retrieval → BGE Reranking → Relevance Grading → Answer Generation
```

### Research queries

```text
Query → Question Decomposition → Evidence Retrieval → Evidence Grading → Sufficiency Check → Research Iteration → Answer Synthesis → Grounding Check
```

This routing allows the system to apply deeper reasoning only when the query requires it.

---

## Access Control & Human Review

### Role-Based Access Control

Document permissions are applied at retrieval time using vector-store metadata filters.

```text
User → Authentication → Role Resolution → Authorized Retrieval → Retrieved Evidence
```

This keeps access control outside the language model's decision-making process.

### Human-in-the-Loop Approval

High-stakes responses can pause the workflow for explicit human approval. LangGraph checkpointing preserves workflow state while approval is pending and allows the process to resume without restarting the workflow.

---

## Retrieval Pipeline

```text
Query → Embedding → Vector Retrieval → Candidate Documents → BGE Cross-Encoder Reranking → Relevant Evidence
```

The research workflow can repeat retrieval when evidence is insufficient to answer a decomposed sub-question.

---

## Evaluation

The system was evaluated on the public **FinanceBench** benchmark.

| Metric | Result |
| --- | ---: |
| Correctness pass rate | **72.7% (109/150)** |
| Refusal rate | 6.7% |
| RAGAS faithfulness | 0.747 |
| DeepEval faithfulness | 0.844 |
| DeepEval contextual recall | 0.768 |

The evaluation pipeline separates retrieval quality, grounding, and final-answer correctness. Detailed methodology and reproduction instructions are available in [`docs/evaluation.md`](docs/evaluation.md).

---

## Technology Stack

### Backend

* Python 3.12
* FastAPI
* LangGraph
* PostgreSQL
* Redis
* Qdrant

### Retrieval & AI

* Vector embeddings
* BGE cross-encoder reranking
* Claude
* GPT
* Llama-based models

### Observability

* LiteLLM
* Langfuse
* Redis semantic caching

### Deployment

* Docker
* Docker Compose
* GitHub Actions

---

## Key Engineering Decisions

### Selective Agentic Routing

The system distinguishes between straightforward retrieval tasks and queries that require multi-step research, reducing unnecessary orchestration for simple requests.

### Retrieval-Layer Authorization

Role permissions are enforced during document retrieval rather than relying on generated responses to respect access restrictions.

### Stateful Agent Workflows

Research and approval processes preserve state across multiple execution steps, including human approval and service restarts.

### Explicit Evaluation

The evaluation framework measures faithfulness, contextual recall, refusal behavior, and final-answer correctness instead of relying on a single metric.

---

## Project Structure

```text
.
├── app/
│   ├── api/
│   ├── agents/
│   ├── graph/
│   ├── retrieval/
│   ├── auth/
│   ├── evaluation/
│   └── services/
├── docs/
├── tests/
├── web/
├── Dockerfile
├── docker-compose.yml
└── pyproject.toml
```

---

## Getting Started

### Clone

```bash
git clone https://github.com/tallamSai/Multi-Agent-Financial-Research-Risk-Intelligence-Platform.git
cd Multi-Agent-Financial-Research-Risk-Intelligence-Platform
```

### Install

```bash
pip install -e ".[backend,dev]"
```

### Configure

```bash
cp .env.example .env
```

Add the required model and service configuration to `.env`.

### Start Services

```bash
docker compose up -d
```

Use the project CLI or API documented in [`docs/setup.md`](docs/setup.md).

---

## Testing

```bash
pytest
```

The current repository reports **347 passing tests**.

---

## Documentation

* [`Architecture`](docs/architecture.md)
* [`Evaluation`](docs/evaluation.md)
* [`Deployment`](docs/deploy.md)
* [`Engineering Log`](docs/engineering-log.md)
* [`RBAC Matrix`](docs/rbac-matrix.md)
* [`Setup Guide`](docs/setup.md)
* [`CLI Reference`](docs/cli.md)
* [`API Reference`](docs/api-reference.md)

---

## Limitations

* The platform is designed for financial research and engineering evaluation.
* Model outputs depend on the selected language model and retrieved evidence.
* Benchmark performance should not be interpreted as investment performance.
* Production deployment requires appropriate infrastructure, security controls, monitoring, and data-provider configuration.

---