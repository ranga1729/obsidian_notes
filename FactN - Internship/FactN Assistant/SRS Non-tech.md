# ChatN — Multi-Tenant AI API Gateway Platform
## Software Requirements Specification (Non-Technical Edition)

> **Audience:** Product stakeholders, business leads, non-technical evaluators, investors, and any reader who wants to understand *what* the platform does and *why* it exists — without requiring engineering knowledge.

---

## Table of Contents

1. [What Is This Platform?](#1-what-is-this-platform)
2. [The Problem We Are Solving](#2-the-problem-we-are-solving)
3. [Our Solution](#3-our-solution)
4. [Who Uses This Platform?](#4-who-uses-this-platform)
5. [What Can Users Do? — Feature Overview](#5-what-can-users-do--feature-overview)
6. [What Is Out of Scope (Version 1.0)](#6-what-is-out-of-scope-version-10)
7. [Use Cases — Plain English](#7-use-cases--plain-english)
8. [Security and Privacy Commitments](#8-security-and-privacy-commitments)
9. [Subscription Plans and Pricing](#9-subscription-plans-and-pricing)
10. [Key Terms Glossary](#10-key-terms-glossary)

---

## 1. What Is This Platform?

ChatN is a **managed gateway platform** that allows developers and organisations to build AI-powered chat and voice applications without being locked in to a single AI provider (such as OpenAI or Google). Instead of writing provider-specific code, developers connect to ChatN once and can swap AI providers, models, or settings from a web dashboard — no code changes, no redeployment.

Think of it like a **universal adapter plug for AI**: your application always connects the same way, and ChatN handles all the translation work behind the scenes.

---

## 2. The Problem We Are Solving

Building AI applications today is harder than it should be for four key reasons:

### 2.1 Every AI Provider Speaks a Different Language
OpenAI, Google, Anthropic, and Mistral each have their own data formats, message structures, and API conventions. A developer who builds an application using OpenAI today cannot simply "switch to Google" without rewriting large parts of their codebase.

### 2.2 Even One Provider's Own APIs Are Incompatible With Each Other
OpenAI, for example, offers a Chat Completions API, a Responses API, and a Realtime API — and switching between them still requires substantial code changes.

### 2.3 Real-Time Voice Is Especially Complex
Building voice-based AI agents (like a customer service bot that speaks) requires handling live audio streams, detecting when a speaker stops talking, managing connection state, and dealing with binary audio data. Every provider does this differently, raising the technical bar enormously.

### 2.4 Configurations Live Inside Code
When an AI agent's personality, instructions, or voice settings are embedded in source code, a non-technical stakeholder (product manager, content writer, business owner) cannot update them without involving a developer and triggering a new software release.

---

## 3. Our Solution

ChatN solves all four problems by sitting between your application and the AI provider:

- **One connection standard:** your application always connects to ChatN the same way, regardless of which AI provider powers it.
- **Provider switching in a few clicks:** change from OpenAI to Google, or from a chat model to a voice model, directly in the ChatN dashboard — your application code stays the same.
- **Prompt and behaviour management via a web UI:** business stakeholders can update the agent's personality, tone, or instructions from a browser without touching code.
- **Built-in knowledge base (RAG):** upload your own documents; the AI will answer questions based on your content.
- **External tool connections (MCP):** connect the AI to your own services and data sources so it can take real actions, not just generate text.
- **Voice support out of the box:** stream live audio to and from the AI without managing the low-level complexity.

---

## 4. Who Uses This Platform?

| Actor | Description |
|---|---|
| **Platform User** | A developer or organisation that signs up, creates projects, and builds AI-powered applications using ChatN. |
| **End Client** | The end-user of the Platform User's application — the human (or system) actually chatting with the AI. They connect through the Platform User's app; they never sign up on ChatN directly. |
| **Platform Operator / Admin** | Internal ChatN staff who manage the platform: user accounts, subscription plans, AI provider catalogue, and system health. |

---

## 5. What Can Users Do? — Feature Overview

### 5.1 Connect to Multiple AI Providers From One Place
Version 1 supports five distinct AI APIs across two providers:
- **OpenAI:** Realtime API (voice), Chat Completions API (text), Responses API (stateful text)
- **Google:** Gemini Live API (voice), Gemini Generate Content API (text)

### 5.2 Organise Work Into Projects
Each AI application you build lives in its own **Project**. A project holds all settings for that application: which AI to use, how it should behave, what documents it can reference, and what external tools it can use.

### 5.3 Configure Everything From a Web Dashboard
A step-by-step configuration wizard lets you set:
- Which AI provider and model to use
- The agent's personality (system prompt)
- Whether it uses text or voice
- Voice and transcription settings (for voice-capable APIs)
- Knowledge base documents
- External tool connections
- Model behaviour parameters (e.g. creativity level)

### 5.4 Manage Agent Behaviour Without Code
You can write multiple "system prompts" (the instructions that define your agent's personality) and switch between them instantly from the dashboard. No developer involvement, no redeployment. Useful when you want to test a "formal" tone versus a "friendly" tone, or update seasonal messaging.

### 5.5 Real-Time Voice Applications
For voice-enabled projects, end users can speak to the AI and receive spoken responses in real time. ChatN handles all the complex audio streaming and provider-specific formatting.

### 5.6 Knowledge Base (RAG — Retrieval-Augmented Generation)
Upload your own documents (PDFs, Word files, text files, Markdown). When an end user asks a question, the AI searches your documents and answers based on your content — not just its general training. This is called RAG.

### 5.7 Connect External Tools and Services (MCP Servers)
Register external services that the AI can call during a conversation (for example: a product catalogue lookup, a booking system, or a database query). ChatN automatically discovers what the service can do and lets the AI use it. The end user's conversation stays private — tool calls are proxied through ChatN.

### 5.8 Secure API Key and Token Management
- **Long-lived API keys** are used by your server-side backend to securely request connection tokens.
- **Ephemeral (short-lived) tokens** are passed to the end user's device. They expire in 60 seconds and can only be used once — so even if intercepted, they are useless.

### 5.9 Session Monitoring
See all currently active conversations on your project in real time: duration, model used, transport type, system prompt in use, and live token count. View historical sessions with filters and drill into any session for a full usage breakdown.

### 5.10 Usage and Cost Reporting
Track token consumption, audio usage, tool calls, and estimated costs — at the session, project, and account level. View trends over time (daily / monthly) and receive notifications when approaching plan limits.

### 5.11 Admin Portal
Platform Operators have a separate admin area to:
- Manage user accounts (search, activate, deactivate)
- Manage subscription plans
- Manage the AI provider catalogue (add/update providers, models)
- View platform health and error rates
- Review audit logs
- Force-close any active session

---

## 6. What Is Out of Scope (Version 1.0)

The following items are **not** included in the first release but are planned for the future:

| Not in v1.0 | Notes |
|---|---|
| Mobile apps (iOS / Android) | Not planned for v1.0 |
| Payment processing and invoicing | Usage data is captured; billing integration planned later |
| Anthropic Claude, Mistral, or other providers | Planned for v1.1 |
| Multi-user team accounts | One account per tenant in v1.0; teams planned for v2.0 |
| Webhook delivery for session events | Planned for v1.1 |
| A/B testing between system prompts | Planned post-v1.0; architecture already supports it |
| OpenAI Assistants API | Deprecated by OpenAI and shutting down August 26, 2026 — will not be integrated |

---

## 7. Use Cases — Plain English

### For Platform Users (Developers / Organisations)

| # | What You Can Do |
|---|---|
| UC-01 | **Sign up and verify your email** to get started. |
| UC-02 | **Log in and log out** securely. |
| UC-03 | **Reset your password** via a one-time email link. |
| UC-04 | **Create a new project** and configure it through a step-by-step wizard — pick your AI provider, model, personality, tools, and more. |
| UC-05 | **Manage API keys** — create keys for your application, see when they were last used, and revoke them if needed. |
| UC-06 | **Issue connection tokens** — your backend automatically requests a short-lived token that your end user's device uses to open a single session. |
| UC-07 | **Manage system prompts** — write multiple versions of your agent's personality and switch between them instantly without code changes. |
| UC-08 | **Monitor active sessions** and review past session history. |
| UC-09 | **View usage and cost reports** at the session, project, or account level. |
| UC-10 | **Upload documents** to power a knowledge-base (RAG); delete them when no longer needed. |
| UC-11 | **Register external tools** (MCP servers) — add a URL, and ChatN automatically discovers what the tool can do and makes it available to your AI. |

### For End Clients (End Users of Your Application)

| # | What They Can Do |
|---|---|
| UC-12 | Connect to your AI agent via a **real-time WebSocket** (persistent, two-way connection). |
| UC-13 | Connect to your AI agent via **SSE** (simpler streaming connection for text-based agents). |
| UC-14 | **Send a text message and receive a streaming text response** as the AI generates it word by word. |
| UC-15 | **Speak and receive a spoken response** in real time (for voice-enabled projects). |
| UC-16 | **Trigger tool calls** — the AI can request your application to perform an action (e.g. look up a product price) and wait for the result before continuing. |
| UC-17 | See a **"Searching knowledge base…" notification** when the AI is looking up documents on their behalf. |
| UC-18 | Receive **live transcripts** of spoken conversations (when transcription is enabled). |

### For Platform Operators (ChatN Internal Staff)

| # | What They Can Do |
|---|---|
| UC-19 | **Manage the AI provider catalogue** — add new providers, API types, and models. |
| UC-20 | **Manage subscription plans** — create and update pricing tiers and feature limits. |
| UC-21 | **Manage user accounts** — search, view, activate, or deactivate any user. |
| UC-22 | **Force-close any active session** — the session is terminated within 5 seconds. |
| UC-23 | **Review the full audit log** — every significant action on the platform is recorded and cannot be altered. |

---

## 8. Security and Privacy Commitments

ChatN is built with security and user privacy as foundational concerns, not afterthoughts.

### 8.1 Your Conversations Are Never Stored
Conversation content — message text, audio, tool arguments, and AI responses — is **never saved to any long-term database**. It exists only in memory while a session is active and is explicitly deleted the moment the session ends.

### 8.2 You Never Handle AI Provider Credentials
ChatN holds all OpenAI and Google API credentials centrally. Platform Users never see, manage, or risk exposing these keys. This removes a major security burden from developers.

### 8.3 Short-Lived, Single-Use Tokens
End users connect using tokens that expire in 60 seconds and can only be used once. Even if a token were intercepted, it would be worthless.

### 8.4 API Keys Are Never Stored in Plain Text
When you create an API key, the raw value is shown to you once and never stored. Only a cryptographic fingerprint is kept. If you lose the key, you create a new one.

### 8.5 Every Action Is Logged
An immutable audit trail records every significant action on the platform — who did what, when, and from which IP address. This log cannot be modified or deleted.

### 8.6 Strong Tenant Isolation
Every data query is scoped to your account and your projects. It is architecturally impossible for one Platform User to access another's data, documents, or session information.

### 8.7 Protection Against Common Attacks
The platform is built to resist:
- **Brute-force attacks:** rate limiting on login and registration endpoints.
- **DDoS attacks:** connection rate limiting at the gateway.
- **SSRF attacks:** external tool URLs are validated; connections to internal network addresses are blocked.
- **Unauthorised data access (IDOR):** every database query is scoped to the requesting user's account.

---

## 9. Subscription Plans and Pricing

ChatN offers four plans designed for different stages of growth.

### Plan Overview

| Plan | Price | Best For |
|---|---|---|
| **14-Day Free Trial** | $0 | Developers evaluating the platform |
| **Starter** | $19 / month | Solo developers and side projects |
| **Pro** | $79 / month | Small teams and growing startups |
| **Enterprise** | Custom pricing | Large organisations with high volume needs |

### Plan Comparison

| Feature | Free Trial | Starter | Pro | Enterprise |
|---|---|---|---|---|
| Projects | 1 | 2 | 10 | Custom |
| API Keys per Project | 2 | 5 | 10 | Custom |
| System Prompts per Project | 3 | 10 | 25 | Custom |
| Concurrent Sessions | 3 | 10 | 50 | Custom |
| Knowledge Base (RAG) | ✅ | ❌ | ✅ | ✅ |
| Max Documents | 5 | 0 | 100 | Custom |
| External Tools (MCP) | ✅ | ❌ | ✅ | ✅ |
| MCP Servers per Project | 1 | 0 | 5 | Custom |
| Voice Support | ✅ | ❌ | ✅ | ✅ |
| Ephemeral Tokens | ✅ | ✅ | ✅ | ✅ |

> **Note:** AI provider usage costs (OpenAI, Google token fees) are separate and vary by provider and model. See the cost estimation section of the technical documentation for detailed breakdowns.

---

## 10. Key Terms Glossary

| Term | Plain-English Definition |
|---|---|
| **Platform** | The ChatN AI API Gateway system described in this document. |
| **Platform User** | A developer or organisation registered on ChatN to build AI applications. |
| **End Client** | The end user of the Platform User's application — the human chatting with the AI. |
| **Project** | A self-contained configuration unit representing one AI application. |
| **Session** | A single active connection between an end user and the AI, lasting from "hello" to "goodbye". |
| **Provider** | An external AI company (e.g. OpenAI, Google) whose models power the AI responses. |
| **API Type** | A specific service offered by a provider (e.g. voice chat, text chat, stateful conversations). |
| **System Prompt** | The set of instructions that defines how the AI behaves — its personality, role, tone, and constraints. |
| **RAG** | "Retrieval-Augmented Generation" — the ability to upload your own documents and have the AI answer questions based on them. |
| **MCP Server** | An external service that the AI can call during a conversation (e.g. a product database, a calendar tool). |
| **Ephemeral Token** | A short-lived, single-use access pass (60-second expiry) that lets an end user open one session. |
| **WebSocket (WSS)** | A persistent, two-way connection used for real-time voice and interactive text sessions. |
| **SSE** | Server-Sent Events — a simpler, one-way streaming connection used for text-based sessions. |
| **Adapter** | An internal component that translates between ChatN's standard format and a specific AI provider's format. |
| **Audit Log** | An immutable, tamper-proof record of every significant action taken on the platform. |
| **Subscription Plan** | A pricing tier that determines which features and resource limits are available to a Platform User. |
