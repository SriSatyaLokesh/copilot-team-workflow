# docs/ — Project Documentation Index

This directory contains the physical documentation for the project. Every architectural decision, user flow, and work item is tracked here.

## The Golden Rule
**One thing = One file.** Every API endpoint, user flow, or work item has exactly one doc file. This ensures clear context for both developers and AI agents.

## Folder Map
```
docs/
├── README.md                     ← This index
├── codebase/                     ← Standardized codebase docs (Architecture, Stack, etc.)
├── templates/                    ← Copy-paste starters for every doc type
├── external-apis/                ← One folder per external API called
├── issues/                       ← One doc per work item / feature / bug fix
├── apis/                         ← One doc per internal API endpoint
├── flows/                        ← One doc per user journey / business flow
├── decisions/                    ← Architecture Decision Records (ADRs)
└── team-notes/                   ← Personal scratch space per developer
```

## Project Overview
[Summarize the project's purpose, core mission, and primary users here. This serves as the high-level functional context for the entire codebase.]

## System Architecture
[Document the conceptual organization of the system. Describe the relationship between major layers (e.g., Controllers, Services, Wrappers) and how data flows through them.]

---
*For a detailed developer onboarding experience, see [DEVELOPER-STANDARD-GUIDE.md](DEVELOPER-STANDARD-GUIDE.md).*
