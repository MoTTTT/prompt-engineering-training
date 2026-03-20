# Module 2 — Context, Memory, and RAG Architecture

## Overview

Module 2 covers how LLMs handle context and memory, how vector embeddings make semantic search possible, and how to design a Retrieval-Augmented Generation (RAG) pipeline that grounds your AI system in real data. This module is primarily for architects and senior developers who will design or build RAG-based systems.

---

## Sections

| Section | Title | Estimated time |
|---------|-------|---------------|
| [01](01-context-windows-and-memory.md) | Context Windows and Memory | 40 minutes |
| [02](02-embeddings-and-vector-search.md) | Embeddings and Vector Search | 50 minutes |
| [03](03-rag-architecture.md) | RAG Architecture | 60 minutes |
| [Lab 02](lab-02-rag-assistant.md) | Build a RAG Documentation Assistant | 90 minutes |

---

## Learning Objectives

By the end of this module, you will be able to:

- Explain the Lost in the Middle effect and design prompts that avoid it
- Choose an appropriate memory strategy for a given application's session requirements
- Explain what embeddings are, how cosine similarity works, and what a vector database does
- Compare chunking strategies and select the appropriate one for a given document type
- Build a complete RAG pipeline in Java (Spring AI + PgVector) or Python
- Explain when RAG is the right choice versus fine-tuning

---

## Prerequisites

- Module 1 (Foundations and Prompt Design)

---

## Who Should Take This Module

- Architects designing knowledge base or document Q&A systems
- Senior developers implementing RAG pipelines
- Anyone making decisions about which vector database or embedding model to use

If you are building a simple chatbot that does not use private or up-to-date knowledge, you can skip this module and return when RAG becomes relevant.
