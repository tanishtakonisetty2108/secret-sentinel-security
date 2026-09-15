# Secret Sentinel

> **From Secret Detection to Exposure Intelligence**

Secret Sentinel is a developer-focused security platform that detects potentially exposed secrets in source code **before they are committed**, explains why a finding is risky, blocks high-risk commits through a Git pre-commit hook, and provides a web dashboard for tracking exposure, remediation, and repository security posture.

It is designed around one principle:

> **A secret is not merely a suspicious string. It is an exposure event with identity, confidence, risk, location, impact, lifecycle, and remediation status.**

---

## Table of Contents

- [Problem Statement](#problem-statement)
- [Solution](#solution)
- [Why Secret Sentinel](#why-secret-sentinel)
- [Key Features](#key-features)
- [Detection Methodology](#detection-methodology)
- [Risk and Confidence Model](#risk-and-confidence-model)
- [Secret DNA](#secret-dna)
- [Exposure Timeline](#exposure-timeline)
- [Exposure Impact and Blast Radius](#exposure-impact-and-blast-radius)
- [Intent Classification](#intent-classification)
- [Pre-Commit Protection](#pre-commit-protection)
- [Git History Scanning](#git-history-scanning)
- [Dashboard](#dashboard)
- [Compliance Metrics](#compliance-metrics)
- [Adaptive Organization Rules](#adaptive-organization-rules)
- [Architecture](#architecture)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Supported Files](#supported-files)
- [Installation](#installation)
- [Running the Scanner](#running-the-scanner)
- [Installing the Git Hook](#installing-the-git-hook)
- [Running the Dashboard](#running-the-dashboard)
- [Testing](#testing)
- [Hackathon Demo](#hackathon-demo)
- [Security and Privacy](#security-and-privacy)
- [Limitations](#limitations)
- [Future Scope](#future-scope)
- [Team](#team)
- [License](#license)

---

## Problem Statement

Developers can accidentally commit sensitive information into source-code repositories, including:

- API keys
- Database passwords
- Access tokens
- Authentication tokens
- Cloud credentials
- Private keys
- Other credential-like values

A secret that reaches a repository can create unnecessary security exposure and may remain discoverable in repository history even after it is removed from the latest version of the code.

The official challenge requires a solution with two components:

1. **Developer-side scanner**
2. **Web security dashboard**

The scanner must detect potential secrets before commit using pattern matching and entropy analysis, classify risk, warn developers, and block sufficiently high-risk commits.

The dashboard must provide security visibility through scan history, findings, exposure information, and compliance metrics.

---

## Solution

Secret Sentinel combines multiple detection signals instead of treating every occurrence of words such as `PASSWORD`, `TOKEN`, or `SECRET` as a confirmed credential.

### Core pipeline

```text
SOURCE CODE
     │
     ▼
FILE SCANNER
     │
     ├── Pattern / Regex Detection
     ├── Shannon Entropy
     └── Context Analysis
             │
             ▼
       RISK ENGINE
             │
             ├── Confidence
             ├── Severity
             └── Explanation
             │
             ▼
       SECRET DNA
             │
             ├── Exposure Identity
             └── Cross-location Correlation
             │
             ▼
     EXPOSURE ANALYSIS
             │
             ├── Timeline
             └── Static Impact Estimation
             │
             ▼
       ALLOW / BLOCK
             │
             ▼
       BACKEND + DATABASE
             │
             ▼
        WEB DASHBOARD
```

The product flow is:

**DETECT → ANALYZE → IDENTIFY → PRIORITIZE → PREVENT → REMEDIATE → MONITOR**

---

# Why Secret Sentinel

The official requirements establish the baseline for secret detection. Secret Sentinel extends that baseline by treating findings as security events rather than isolated strings.

### Differentiators

| Capability | Purpose |
|---|---|
| **Secret DNA / Fingerprinting** | Correlates the same suspicious value across files and commits without storing the raw secret |
| **Exposure Timeline** | Shows the lifecycle of a finding from introduction through detection and remediation |
| **Detection Confidence vs Exposure Impact** | Separates "How likely is this a secret?" from "How much could it matter?" |
| **Static Blast-Radius Estimation** | Estimates potentially affected files/modules using code relationships and context |
| **Context-Aware Intent Classification** | Distinguishes likely live credentials from placeholders, test data, and environment references |
| **Explainable Evidence** | Shows the measurable signals responsible for a finding |
| **Current vs Historical Exposure** | Separates secrets present now from secrets that remain exposed through Git history |
| **Adaptive Organization Rules** | Allows security teams to create organization-specific rules from recurring patterns |

These are implemented as functional security capabilities rather than decorative dashboard labels.

---

# Key Features

## 1. Multi-Signal Secret Detection

Secret Sentinel combines:

- Configurable regex/pattern detection
- Shannon entropy analysis
- Source-code context analysis
- False-positive reduction
- Explainable risk scoring

This allows the system to distinguish cases such as:

```python
API_KEY="random-looking-value"
```

from:

```python
API_KEY=os.getenv("API_KEY")
```

and:

```python
API_KEY="YOUR_API_KEY_HERE"
```

---

## 2. Entropy-Based Detection

Pattern matching cannot cover every possible secret format.

Secret Sentinel calculates **Shannon entropy** for suspicious values and uses randomness as one signal in the risk model.

Entropy is **not used alone** to declare a value a secret.

Instead:

```text
Pattern Strength
       +
Entropy
       +
Context
       +
File Sensitivity
       +
Other Signals
       ↓
Risk Assessment
```

---

## 3. Context-Aware False-Positive Reduction

The scanner considers:

- Variable/key names
- Surrounding source lines
- File extension
- Filename
- Configuration context
- Comments
- Test/example indicators
- Placeholder indicators
- Environment-variable references
- Allowlisted values

Recognized low-risk cases include:

- Placeholders
- Documentation examples
- Test fixtures
- Mock credentials
- Environment-variable references
- Allowlisted values

Developers can suppress/allowlist findings with a recorded reason.

---

# Risk and Confidence Model

Secret Sentinel does not randomly generate risk scores.

Risk is derived from measurable signals such as:

- Pattern match strength
- Entropy
- Credential-like variable name
- Value length
- File sensitivity
- Production/configuration context
- Test/example indicators
- Git exposure
- Number of occurrences
- Reference/dependency impact

A finding produces:

- **Risk score**
- **Confidence percentage**
- **Severity**
- **Evidence**
- **Explanation**
- **Exposure impact**

Example:

```text
CRITICAL
Risk: 94/100
Confidence: 97%

Evidence:
✓ Credential pattern matched
✓ High entropy
✓ Credential-like variable name
✓ Configuration file
✓ No placeholder indicators
```

### Important distinction

**Detection Confidence** answers:

> How likely is this value to be a real secret?

**Exposure Impact** answers:

> If it is a secret, how significant could its exposure be?

This prevents a high-confidence finding from automatically being treated as high-impact without contextual analysis.

---

# Secret DNA

## Privacy-Preserving Secret Fingerprinting

Secret Sentinel never needs to display the complete detected credential.

Instead, it creates a privacy-preserving fingerprint and internal Secret ID.

Example:

```text
Secret ID: SEC-A81F2C
Fingerprint: 7F92...
```

The fingerprint allows the platform to recognize when the same suspicious value appears in multiple locations without storing the raw secret.

For example:

```text
1 credential
   ├── config/database.env
   ├── backend/config.py
   └── historical commit
```

can be correlated as one exposure rather than incorrectly represented as three unrelated secrets.

**The raw credential is never shown in the dashboard or terminal warning.**

---

# Exposure Timeline

Secret Sentinel goes beyond a simple scan-history table.

A finding can have a lifecycle such as:

```text
Secret Introduced
       ↓
Commit Attempted
       ↓
Scanner Detected
       ↓
Commit Blocked
       ↓
Developer Remediated
       ↓
Clean Scan
       ↓
Commit Allowed
```

Where Git history is available, the system can associate findings with:

- First detected commit
- Affected files
- Number of commits
- Detection time
- Resolution time
- Current status

No Git events or timestamps are fabricated.

---

# Exposure Impact and Blast Radius

Secret Sentinel performs **static analysis** to estimate potential exposure impact.

It does **not** attempt to use, validate, authenticate with, or connect to detected credentials.

The analysis can consider:

- Files referencing the finding
- Related modules
- Configuration context
- Production-related paths
- Number of affected files/modules

### Exposure Impact

```text
LOW
MEDIUM
HIGH
CRITICAL
```

The product clearly labels this as an **estimate based on static analysis**, not proof of actual compromise.

Example:

```text
Detection Confidence: 96%
Exposure Impact: HIGH
Overall Priority: CRITICAL
```

---

# Intent Classification

The prototype classifies findings into useful context-based categories:

- `LIKELY LIVE CREDENTIAL`
- `SUSPICIOUS`
- `TEST DATA`
- `PLACEHOLDER`
- `SECURE ENVIRONMENT REFERENCE`
- `ALLOWLISTED`

The prototype uses rule- and context-based reasoning.

It does **not** claim AI/ML functionality unless an actual AI/ML component is implemented.

Every finding includes a **"Why was this flagged?"** explanation.

---

# Pre-Commit Protection

The developer-side scanner integrates with Git through a pre-commit hook.

### Workflow

```text
git add .
     ↓
git commit
     ↓
Pre-Commit Hook
     ↓
Secret Sentinel Scanner
     ↓
Detection Engine
     ↓
Risk Assessment
     ↓
HIGH / CRITICAL Finding?
   ↙           ↘
 YES            NO
 ↓              ↓
BLOCK           ALLOW
```

### Example terminal output

```text
==================================================
SECRET SENTINEL SECURITY SCAN
==================================================

Scanning staged files...

3 potential secrets detected

[CRITICAL] Possible API Credential
File: config/database.env
Line: 12
Confidence: 97%
Risk: 94/100

Evidence:
- Credential pattern matched
- High entropy
- Credential-like variable name

Commit BLOCKED.

Remove or securely reference the credential
before committing.

==================================================
```

The complete secret value is never printed.

A clean scan follows the normal Git commit path.

---

# Git History Scanning

Secret Sentinel can optionally scan repository history for credentials that may have been removed from the current working tree but remain present in earlier commits.

Historical findings can show:

- First exposure
- Commit identifier
- Affected file
- Number of historical occurrences
- Current status

The dashboard distinguishes:

```text
CURRENT EXPOSURE
```

from:

```text
HISTORICAL EXPOSURE
```

Removing a secret from the latest version of a file does not automatically make the historical exposure safe.

Recommended remediation includes credential rotation/revocation and appropriate repository-history remediation procedures.

Secret Sentinel does not automatically execute destructive history-rewriting operations.

---

# Remediation Guidance

Each finding provides practical next steps.

Typical guidance includes:

1. Remove the credential from source code.
2. Rotate or revoke an exposed credential.
3. Move secrets into environment variables or a secrets manager.
4. Add appropriate sensitive files to `.gitignore`.
5. If exposed in Git history, follow appropriate repository-history remediation procedures.
6. Review potentially affected systems.

The platform does not expose the secret itself.

---

# Dashboard

The web dashboard provides security visibility backed by application data.

## Overview

Metrics include:

- Total scans
- Potential secrets detected
- Critical findings
- High findings
- Blocked commits
- Resolved findings
- Current open findings
- Compliance score

No dashboard number is hardcoded as a fake statistic.

---

## Secret Intelligence

The dashboard can provide:

- Secret type distribution
- Severity distribution
- Current vs historical exposure
- Top affected repositories/files
- Exposure trends
- Correlated Secret DNA findings

---

## Findings

Each finding can contain:

| Field | Description |
|---|---|
| Secret ID | Privacy-preserving finding identity |
| Type | Credential category |
| Severity | Risk classification |
| Risk Score | Calculated risk |
| Confidence | Detection confidence |
| Repository/Project | Source context |
| File | Affected file |
| Line | Affected line |
| Detection Time | Stored event timestamp |
| Status | Open/resolved/etc. |
| Exposure Impact | Static impact estimate |
| Fingerprint | Privacy-preserving correlation value |
| Explanation | Detection evidence |
| Remediation | Recommended next steps |

---

# Compliance Metrics

Secret Sentinel provides basic repository security posture metrics such as:

- Repositories scanned
- Repositories with unresolved findings
- Blocked commits
- Resolved findings
- Calculated compliance percentage

Metrics are derived from stored scan and event data.

---

# Security Event Timeline

The dashboard includes an event timeline based on real stored application events.

Example:

```text
09:42  Secret detected
09:42  Commit blocked
09:45  Finding remediated
09:46  Clean scan
09:46  Commit allowed
```

This connects developer activity with the lifecycle of a security finding.

---

# Security Exposure Heatmap

The dashboard can visualize exposure by repository/file or application area.

Example categories include:

- Frontend
- Backend
- Payments
- Authentication
- Analytics
- Infrastructure

The visualization is derived from scan results rather than invented organizational data.

The included demo environment is explicitly labelled as **synthetic demo data**.

---

# Adaptive Organization Rules

Organizations often have internal credential formats that generic scanners may not recognize.

Secret Sentinel provides a prototype mechanism for organization-specific rules.

Example:

```text
Observed recurring pattern:
COMPANY_<random-value>

        ↓

Recurring suspicious pattern detected

        ↓

[ Create Rule ]
```

An approved rule is persisted and used in future scans.

This feature is intentionally described as **rule-based adaptation**, not machine learning.

---

# Supported Files

The scanner examines relevant source/configuration files including:

```text
.py
.js
.ts
.java
.env
.yml
.yaml
.json
.xml
.conf
```

It avoids unnecessary content such as:

- Binary files
- Dependency directories
- `.git`
- Generated files
- `node_modules`
- `build/`
- `dist/`
- Virtual environments
- Cache directories

The exclusion system is configurable.

---

# Architecture

```text
                    ┌─────────────────────┐
                    │     Developer       │
                    └──────────┬──────────┘
                               │
                         git commit
                               │
                               ▼
                    ┌─────────────────────┐
                    │   Git Pre-Commit    │
                    │       Hook          │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │    File Scanner     │
                    └──────────┬──────────┘
                               │
                ┌──────────────┼──────────────┐
                ▼              ▼              ▼
          ┌──────────┐  ┌──────────┐  ┌────────────┐
          │ Pattern  │  │ Entropy  │  │  Context   │
          │ Matching │  │ Analysis │  │  Analysis  │
          └────┬─────┘  └────┬─────┘  └─────┬──────┘
               └─────────────┼───────────────┘
                             ▼
                    ┌─────────────────────┐
                    │     Risk Engine     │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │     Secret DNA      │
                    │    Fingerprinting    │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Exposure Intelligence│
                    │ Timeline + Impact    │
                    └──────────┬──────────┘
                               │
                         ┌─────┴─────┐
                         ▼           ▼
                       BLOCK       ALLOW
                         │           │
                         └─────┬─────┘
                               ▼
                    ┌─────────────────────┐
                    │    Backend / API    │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Persistent Database  │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   Web Dashboard     │
                    └─────────────────────┘
```

---

# Technology Stack

The implementation is designed around a lightweight, understandable stack suitable for reliable hackathon deployment.

### Frontend

- React / Next.js
- TypeScript
- Tailwind CSS

### Backend

- Python
- FastAPI

### Scanner

- Python
- Git hooks / subprocess integration

### Database

- SQLite or another lightweight persistent database

The exact stack may be adapted to the implementation while preserving the architecture and functional requirements.

---

# Project Structure

```text
secret-sentinel/
│
├── scanner/              # Secret scanning and detection engine
├── rules/                # Configurable detection rules
├── cli/                  # Developer-facing CLI
├── hooks/                # Git pre-commit integration
├── backend/              # API and application services
├── frontend/             # Web dashboard
├── database/             # Database models/schema
├── tests/                # Automated tests
├── demo/                 # Safe synthetic demonstration repository
├── docs/                 # Architecture and supporting documentation
│
├── README.md
├── .gitignore
├── LICENSE
└── package/requirements files
```

The final structure should reflect the actual implementation.

---

# Installation

> The commands below should be updated to match the final project stack.

## Clone

```bash
git clone <REPOSITORY_URL>
cd secret-sentinel
```

## Backend / Scanner

Create the required environment and install dependencies:

```bash
pip install -r requirements.txt
```

If the project uses a frontend package manager:

```bash
npm install
```

---

# Running the Scanner

Example:

```bash
python -m scanner
```

or the project's final CLI command.

The scanner should report:

- Files scanned
- Findings
- Severity
- Confidence
- Risk
- Evidence
- File and line
- Final scan decision

Complete secret values must never be printed.

---

# Installing the Git Hook

Install Secret Sentinel as the repository's pre-commit hook using the project-provided installation command.

The expected workflow is:

```bash
git add .
git commit -m "test commit"
```

For a sufficiently high-risk finding:

```text
SECRET SENTINEL
      ↓
Finding detected
      ↓
Commit BLOCKED
```

After remediation:

```text
SECRET SENTINEL
      ↓
Clean scan
      ↓
Commit ALLOWED
```

---

# Running the Dashboard

Start the backend and frontend using the project's configured development commands.

The dashboard should read scan results and security events from persistent application storage rather than hardcoded numbers.

Example development flow:

```bash
# Start backend
<backend-start-command>

# Start frontend
<frontend-start-command>
```

---

# Testing

The most security-critical components should have automated tests covering:

- Regex/pattern detection
- Shannon entropy calculation
- Context analysis
- False-positive handling
- Risk scoring
- Fingerprint generation
- Supported file types
- Excluded directories
- Pre-commit blocking
- Clean commit path

Run the project's test suite with the configured test command, for example:

```bash
pytest
```

---

# Hackathon Demo

The intended end-to-end demonstration is:

### 1. Open the demo repository

The demo uses **synthetic credentials only**.

### 2. Introduce a synthetic credential

Place a deliberately synthetic credential-like value in a source/configuration file.

### 3. Run Secret Sentinel

The scanner analyzes the staged files.

### 4. Show the detection reasoning

Demonstrate:

```text
Pattern
   +
Entropy
   +
Context
   ↓
Risk
   ↓
Decision
```

### 5. Show the finding

Example:

```text
Type: Possible API Credential
Confidence: 97%
Risk: 94/100
Exposure Impact: HIGH
```

### 6. Attempt a Git commit

```bash
git commit -m "test commit"
```

### 7. Show the pre-commit protection

The high-risk commit is blocked.

### 8. Open the dashboard

Show the finding and its associated security event.

### 9. Open finding details

Demonstrate:

- Secret ID
- Fingerprint
- Risk
- Confidence
- Detection evidence
- Exposure timeline
- Static impact estimate
- Remediation guidance

### 10. Remediate

Replace the synthetic credential with a secure environment-variable reference.

### 11. Rescan

The scanner performs a clean scan.

### 12. Commit again

The clean commit succeeds.

### 13. Show the updated dashboard

The latest security event is reflected in the application data.

---

# Demo Data

The included demonstration environment should contain only **synthetic** credential-like values.

It should demonstrate:

1. Obvious credential-like value
2. High-entropy suspicious value
3. Placeholder
4. Environment-variable reference
5. Test/example value
6. Multiple occurrences of the same synthetic value

This makes it possible to demonstrate:

- True-positive detection
- Entropy-based detection
- False-positive reduction
- Secret fingerprinting
- Multiple exposure locations
- Risk scoring
- Commit blocking
- Remediation
- Clean rescanning

---

# Security and Privacy

Secret Sentinel follows a security-first design.

### The system does not:

- Display complete detected secrets
- Include real credentials in the demo
- Validate credentials against external services
- Attempt to authenticate using detected credentials
- Automatically access systems associated with a credential
- Automatically execute destructive Git-history operations
- Claim that static analysis proves a breach

### Secret handling principle

```text
SOURCE VALUE
     ↓
DETECTION
     ↓
SECURE FINGERPRINT
     ↓
METADATA + RISK
     ↓
DASHBOARD

RAW SECRET
     ✕
NOT EXPOSED
```

**Security notice:** This project uses synthetic credentials for demonstration and testing. **Never commit real secrets to this repository.**

---

# Limitations

Secret Sentinel is a hackathon prototype and therefore has practical limitations.

- Detection is probabilistic and may produce false positives or false negatives.
- Entropy is a supporting signal, not proof that a value is a credential.
- Static blast-radius analysis is an estimate, not a guarantee of actual system impact.
- Provider-specific credential formats require appropriate rules.
- The prototype does not validate whether a detected credential is currently active.
- Historical exposure analysis depends on available Git history.
- Organization-specific rules require deliberate configuration/approval.
- Security posture metrics are limited to repositories and events available to the application.

These limitations are intentionally documented rather than hidden.

---

# Future Scope

Potential extensions include:

- Broader provider-specific secret coverage
- More advanced code/data-flow analysis
- Enterprise secret-manager integrations
- Credential rotation workflows through approved integrations
- Deeper repository and dependency graphs
- Organization-wide policy management
- Pull-request protection
- CI/CD pipeline integrations
- Distributed scanning
- Advanced statistical/ML-assisted contextual analysis
- Expanded compliance reporting
- Security team notifications and workflow integrations

---

# What We Do Not Claim

Secret Sentinel does **not** claim:

- 100% secret detection
- Zero false positives
- Complete breach prevention
- Military-grade security
- Enterprise-grade security without appropriate validation
- AI-powered detection unless an actual AI component is present

Instead, the product uses precise security language:

- **Potential secret**
- **Detection confidence**
- **Estimated exposure impact**
- **Static analysis**
- **Prototype**

---

# Quality Bar

Before release, the prototype should satisfy the following end-to-end checks:

- [ ] Scanner scans supported files
- [ ] Unnecessary directories are excluded
- [ ] Pattern detection works
- [ ] Entropy calculation works
- [ ] Context analysis works
- [ ] Risk score is calculated from real signals
- [ ] Complete secrets are never exposed
- [ ] Fingerprinting works
- [ ] Pre-commit hook works
- [ ] High-risk commits are blocked
- [ ] Clean commits succeed
- [ ] Dashboard reads backend data
- [ ] Scan history persists
- [ ] Compliance metrics are calculated
- [ ] Finding details work
- [ ] Exposure timeline works
- [ ] Impact estimation works
- [ ] Synthetic demo data works
- [ ] Light mode works
- [ ] Dark mode works
- [ ] Responsive layout works
- [ ] No fake statistics
- [ ] No fake reviews/testimonials/users
- [ ] No non-functional major buttons
- [ ] Tests pass
- [ ] Deployment works
- [ ] No real credentials are present

---

# Product Story

Secret Sentinel turns a source-code security problem into a complete exposure-management workflow:

```text
CODE
  ↓
SCAN
  ↓
DETECT
  ↓
EXPLAIN
  ↓
RISK
  ↓
BLOCK
  ↓
TRACK
  ↓
REMEDIATE
  ↓
RESCAN
  ↓
ALLOW
```

Instead of asking only:

> **"Does this line look like a secret?"**

Secret Sentinel asks:

> **"What is this finding, how confident are we, how exposed is it, where else does it appear, what is its potential impact, and has it been remediated?"**

That is the foundation of **Secret Exposure Intelligence**.

---

# Team

**Team:** `<TEAM_NAME>`

**Members:**
- `<MEMBER_1>`
- `<MEMBER_2>`
- `<MEMBER_3>`
- `<MEMBER_4>`
- `<MEMBER_5>`

---

# License

Add the project's selected open-source license here.

---

## Security Notice

**This project uses synthetic credentials for demonstration and testing. Never commit real secrets to this repository.**
