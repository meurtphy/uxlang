Prometta – MVP Technical Documentation

Version 0.9 – 30 May 2025

Table of Contents

User Stories & Mockups

System Architecture

Components, Classes & Database Design

Key Sequence Diagrams

API Specifications

SCM & QA Strategies

Technical Justifications & Road‑map

1 · User Stories & Mockups

1.1 Prioritised User Stories (MoSCoW)

Priority

User Story

M

En tant que client connecté, je veux coller l’URL de mon site et lancer un audit afin de recevoir rapidement une évaluation UX/UI.

M

En tant que système, je veux analyser automatiquement l’URL via l’API ChatGPT afin de produire un rapport UX structuré et stocké.

M

En tant que client, je veux consulter le rapport sur une page interactive avec des recommandations priorisées.

M

En tant que client, je veux créer un compte et me connecter de manière sécurisée.

M

En tant que admin (Brutux), je veux accéder à une interface d’administration.

S

En tant que client, je veux retrouver l’historique de tous mes audits.

S

En tant que client, je veux voir un score global et un graphique d’évolution.

S

En tant que client, je veux que chaque recommandation affiche un niveau de sévérité.

S

En tant que admin, je veux rechercher / filtrer les audits.

C

En tant que client, je veux exporter le rapport en PDF.

C

En tant que client, je veux une comparaison avant / après.

C

En tant que client, je veux personnaliser certains paramètres d’audit.

(Source : "Prometta_MVP_Dossier" doc) fileciteturn0file0

1.2 Mockups / Wireframes

* Dashboard (logged‑in)* New Audit Form* Audit Report Page* History & Stats* Admin Console

Note : Low‑fidelity wireframes will be produced in Figma during the next design sprint. A placeholder link will be added once available.

2 · System Architecture



Components

Front‑End (React + Vite) — Browser / mobile PWA

Back‑End (Node.js + Express / NestJS)

Auth Service

Audit Controller

ChatGPT Service (orchestrator)

Report Service / Internal API

Databases

PostgreSQL (Users & access control)

MongoDB (Audit reports & screenshots)

External Services

OpenAI ChatGPT API — UX audit generation

(Opt.) SendGrid — email notifications

Infrastructure & Security

Deployed on Heroku / Railway (staging) then AWS ECS (prod)

All traffic over HTTPS; JWT‑based auth; environment secrets in Vault

Data Flow (see arrows on diagram): UI ➡️ Audit request ➡️ Back‑End ➡️ ChatGPT API ➡️ Store results ➡️ Serve report.

3 · Components, Classes & Database Design

3.1 Back‑End Classes / Services (TypeScript)

Class / Service

Key Attributes

Core Methods

User

id (UUID), email, passwordHash, role, createdAt

‑

Audit

id, userId, url, triggeredAt, completedAt, scoreOverall, rawResult(JSON)

calculateScore(), addRecommendation()

Recommendation

id, auditId, severity, title, description, guidelineRef

‑

AuthService

userRepo, tokenSecret

register(), login(), validateToken()

ChatGPTService

openaiClient, promptTemplates

buildPrompt(url), runAuditPrompt(url)

AuditController

auditRepo, chatGPTService

POST / audits, GET / audits/:id

AdminController

userRepo, auditRepo

listUsers(), listAudits(), searchAudits()

3.2 Database Schema (ERD)

See ERD image provided.  fileciteturn0file0turn0file2

Table : users

Field

Type

Notes

id

UUID (PK)



email

VARCHAR(255) (UNQ)



password_hash

TEXT

Argon2 salted

role

ENUM('client','admin')



created_at

TIMESTAMP

default NOW()

Table : audits

Field

Type

Notes

id

UUID (PK)



user_id

UUID (FK→users.id)

ON DELETE CASCADE

url

TEXT

validated URL

triggered_at

TIMESTAMP



completed_at

TIMESTAMP

nullable until finished

score_overall

INT

0‑100

raw_result

JSONB

full ChatGPT JSON

Table : recommendations

Field

Type

Notes

id

UUID (PK)



audit_id

UUID (FK→audits.id)



severity

ENUM('low','medium','high')



title

TEXT



description

TEXT



guideline_ref

TEXT

WCAG / NNG ref

4 · Key Sequence Diagrams

Launch Audit — see prometta_sequence_newaudit_clean.png

User Login & JWT Refresh — to be drawn

Admin View Audits — to be drawn

Each diagram highlights the interactions between Front‑End, AuditController, ChatGPTService, OpenAI API & Database.

5 · API Specifications

5.1 External APIs

Service

Reason for Choice

Usage

OpenAI ChatGPT v 4o

State‑of‑the‑art LLM, JSON mode

UX prompt → structured result

SendGrid (v3)

Reliable transactional email

Account verification, report ready notifications

5.2 Internal REST API (JSON over HTTPS)

Method

Path

Request Body

Success Response

POST

/auth/register

{email, password}

201 Created {token, user}

POST

/auth/login

{email, password}

200 OK {token, user}

POST

/audits

{url}

202 Accepted {auditId}

GET

/audits

–

200 OK [AuditSummary]

GET

/audits/:id

–

200 OK AuditDetails

GET

/audits/:id/pdf

–

200 OK PDF stream

GET

/stats/score-history

–

200 OK [{date, score}]

GET

/admin/users

?page, ?q

200 OK [User]

GET

/admin/audits

?status, ?q

200 OK [Audit]

Standard error envelope: { "error": { "message": string, "code": int } }

6 · SCM & QA Strategies

SCM – Git Hub

main → production ; protected branch

develop → integration

feature/…, bugfix/…, hotfix/…

Mandatory PR review (> 1 approval) & Status‑checks

CI / CD

GitHub Actions: lint → test → build → deploy‑staging

Manual approval gate for production release

QA

Unit: Jest (services, utilities)

Integration: SuperTest (REST endpoints)

E2E: Playwright (critical user journeys)

API contract: Postman collection in CI

Code Quality: ESLint + Prettier, SonarCloud

7 · Technical Justifications & Road‑map

Decision

Rationale

Node.js + TypeScript

High dev velocity; NPM ecosystem; async I/O suits API calls

React

Component re‑use; strong community; PWA ready

PostgreSQL + MongoDB

Relational integrity for auth; flexible JSON storage for audit blobs

OpenAI GPT‑4o

Best accuracy for UX heuristics in natural language

Heroku → AWS ECS

Simple PaaS for MVP, scalable path to production