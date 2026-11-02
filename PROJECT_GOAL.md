# IdeaBin.ai - Project Goal

## Goal
AI-powered idea operating system that captures, structures, connects, evaluates, prioritizes, and converts ideas into executable projects.

## Original objective
Source: repo README (main), 'V1 objective': prove the system can capture at least 100 real ideas, convert them into consistent structured records, identify useful relationships and shared capabilities, rank them better than an unstructured notes workflow, convert selected ideas into executable plans, and learn from predicted vs actual outcomes.

## In scope
Next.js + Supabase (Postgres, Auth, Storage, pgvector) architecture, OpenAI primary models, n8n background ingestion, Vercel deployment, Connections UI.

## Out of scope
Autonomous implementation work of any kind while the owner pause stands (owner paused IdeaBin 2026-09-29 pending a product-overlap reassessment). ChatGPT Sites prototypes are not canonical production.

## Done definition (current)
Registry benchmark IDEA-B2: Mission 6 produces useful owner-scoped related, duplicate, merge, and shared-capability suggestions using the existing Postgres/pgvector architecture, with validated provenance, bounded candidate retrieval, explainable relationship rows, and Connections UI.

## Registry link
Mission Control registry entry `ideabin` in [control/PROJECTS.yaml](https://github.com/TheRealSynth/agent-mission-control/blob/main/control/PROJECTS.yaml) (TheRealSynth/agent-mission-control). Status: PAUSED by owner 2026-09-29; no dispatch until the owner explicitly resumes.
