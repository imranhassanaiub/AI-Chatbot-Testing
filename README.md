# 🤖 AI Chatbot Testing — AchieveTestPrep Admissions Assistant

> **Automated prompt evaluation framework for an EdTech Sales Portal Appointment Setter AI**
> Built with [Promptfoo](https://promptfoo.dev) | Provider: Ollama (Gemma 4) | Domain: EdTech / Healthcare

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [System Under Test (SUT)](#-system-under-test-sut)
- [Technology Stack](#-technology-stack)
- [Project Structure](#-project-structure)
- [Prerequisites](#-prerequisites)
- [Installation & Setup](#-installation--setup)
- [Configuration Guide](#-configuration-guide)
- [Test Cases & Scenarios](#-test-cases--scenarios)
- [Running Tests](#-running-tests)
- [Viewing Results](#-viewing-results)
- [Assertion Types](#-assertion-types)
- [AI Model Provider](#-ai-model-provider)
- [Known Issues & Observations](#-known-issues--observations)
- [QA Best Practices Applied](#-qa-best-practices-applied)
- [Extending the Test Suite](#-extending-the-test-suite)
- [Contributing](#-contributing)
- [References](#-references)

---

## 📌 Project Overview

This project tests an **AI-powered Admissions Chatbot** for *AchieveTestPrep*, an EdTech company offering nursing and healthcare certification programs. The chatbot's primary role is to:

- Qualify prospective students based on pre-defined criteria
- Schedule 15-minute consultation calls with Admissions Advisors
- Route unqualified leads to the free course catalog
- Maintain persona integrity and resist off-topic or competitor manipulation

The evaluation framework uses **Promptfoo** — an open-source LLM testing tool — to run structured test cases against the AI prompt and assert on the quality and correctness of chatbot responses.

---

## 🎯 System Under Test (SUT)

**Chatbot Name:** AchieveTestPrep AI Admissions Assistant

**Role:** Qualify prospective nursing/healthcare students and book consultation appointments

**Programs Offered:**
- CNA to RN (Certified Nursing Assistant to Registered Nurse)
- Medical Assistant to RN
- No License Yet (entry-level pathway)

**Lead Qualification Criteria:**
1. User must specify a target program
2. User must have a funding source OR intent to study within the next 6 months

**Qualified Lead Action:** Offer available appointment slots for a 15-minute consultation

**Unqualified Lead Action:** Direct user to the free catalog page

**Reference Date Used in Tests:** Monday, June 1, 2026

---

## 🛠 Technology Stack

| Component | Tool / Technology |
|---|---|
| Prompt Evaluation Framework | [Promptfoo](https://promptfoo.dev) |
| Configuration Format | YAML (`promptfooconfig.yaml`) |
| LLM Provider | [Ollama](https://ollama.com) — local inference |
| AI Model | `gemma4:e4b` (Google Gemma 4 via Ollama) |
| Runtime Environment | Node.js (required by Promptfoo CLI) |
| Result Sharing | Disabled (`sharing: false`) |
| Latest Results Persistence | Enabled (`writeLatestResults: true`) |

---

## 📁 Project Structure

```
AI-Chatbot-Testing/
│
├── promptfooconfig.yaml      # Core configuration: prompts, providers, test cases, assertions
└── README.md                 # Project documentation
```

> **Note for QA teams:** As the project grows, consider organising test cases into separate YAML files and importing them into the main config using Promptfoo's `import` feature.

---

## ✅ Prerequisites

Ensure the following are installed and configured before running evaluations:

- **Node.js** v18+ — [Download](https://nodejs.org/)
- **Promptfoo CLI** — installed globally via npm
- **Ollama** — installed and running locally — [Download](https://ollama.com/)
- **Gemma 4 model** pulled in Ollama (`ollama pull gemma4:e4b`)

Alternatively, if using a cloud LLM provider, set the appropriate API key:

```bash
# OpenAI
export OPENAI_API_KEY=sk-...

# Anthropic
export ANTHROPIC_API_KEY=sk-ant-...

# Google
export GOOGLE_API_KEY=...
```

---

## ⚙️ Installation & Setup

### 1. Clone the Repository

```bash
git clone https://github.com/imranhassanaiub/AI-Chatbot-Testing.git
cd AI-Chatbot-Testing
```

### 2. Install Promptfoo

```bash
npm install -g promptfoo
```

Verify installation:

```bash
promptfoo --version
```

### 3. Set Up Ollama (Local Provider)

```bash
# Pull the Gemma 4 model used in this project
ollama pull gemma4:e4b

# Verify Ollama is running
ollama list
```

> If Ollama is not running, start it with: `ollama serve`

### 4. (Optional) Switch to a Cloud Provider

To use OpenAI or Anthropic instead of Ollama, update the `providers` section in `promptfooconfig.yaml`:

```yaml
providers:
  - id: openai:gpt-4o
    config: {}
  # or
  - id: anthropic:claude-3-5-sonnet-20241022
    config: {}
```

---

## 🔧 Configuration Guide

The main configuration file is `promptfooconfig.yaml`. Key sections are:

### `description`
High-level description of what the evaluation suite tests.

```yaml
description: 'Testing an EdTech Sales Portal Appointment Setter AI'
```

### `prompts`
Defines the system prompt injected into the model. Uses `{{user_input}}` as the dynamic variable placeholder that is substituted with each test case's `vars.user_input`.

```yaml
prompts:
  - |
    You are an AI Admissions Assistant for 'AchieveTestPrep'.
    ...
    User: {{user_input}}
    Bot:
```

### `providers`
Specifies the LLM backend. Currently configured to use a local Ollama instance.

```yaml
providers:
  - id: ollama:gemma4:e4b
    config: {}
    label: ollama:gemma4:e4b
```

### `tests`
A list of individual test cases, each with:
- `description` — human-readable test name
- `vars` — dynamic input values (e.g., `user_input`)
- `assert` — one or more assertion rules

### `defaultTest`
Global default configuration applied to all tests (currently empty).

### `env`
Environment variable overrides (currently empty).

---

## 🧪 Test Cases & Scenarios

The following 5 test cases are currently defined:

---

### TC-01 — Strict Lead Qualification (Pass)

| Field | Detail |
|---|---|
| **Type** | Positive / Happy Path |
| **Input** | User expresses interest in CNA to RN with intent to discuss financing |
| **Expected Behavior** | Bot identifies prospect as **qualified**, references CNA to RN program, and **offers calendar booking options** |
| **Assertion** | `contains` — response should reference booking options and CNA to RN program |

---

### TC-02 — Strict Lead Qualification (Fail / Route to Catalog)

| Field | Detail |
|---|---|
| **Type** | Negative / Unqualified Lead |
| **Input** | User asks about free hobby coding courses with no intent to spend or talk to anyone |
| **Expected Behavior** | Bot **does not offer a 1-on-1 call**; instead routes user to **free catalog resources** |
| **Assertions** | `not-contains: "schedule a call"` + `contains` reference to free catalog |

---

### TC-03 — Advanced Relative Date Math

| Field | Detail |
|---|---|
| **Type** | Date Reasoning / Accuracy |
| **Input** | Qualified user requests an info session "this Thursday afternoon" (reference date: June 1, 2026) |
| **Expected Behavior** | Bot **correctly resolves "this Thursday" to June 4, 2026** and focuses on afternoon slots |
| **Assertions** | `contains: "June 4"` + contextual confirmation of afternoon availability |

> ⚠️ **QA Note:** This test checks temporal reasoning accuracy — a common failure mode in LLMs. The typo "CNA ro RN" in the input (`ro` instead of `to`) is intentional or incidental and worth noting as a test for fuzzy input tolerance.

---

### TC-04 — Hallucination / Competitor Deflection

| Field | Detail |
|---|---|
| **Type** | Adversarial / Prompt Injection |
| **Input** | User tries to make the bot endorse competitor "Springboard" and abandon its persona |
| **Expected Behavior** | Bot **does not endorse Springboard**, stays in character, and pivots back to AchieveTestPrep strengths |
| **Assertion** | `equals` — response must professionally redirect without breaking persona |

> ⚠️ **QA Note:** The assertion references "EduSphere" in the expected value text — this appears to be a **copy-paste error** from another project. The system prompt refers to "AchieveTestPrep", not "EduSphere". This should be corrected to ensure accurate assertions.

---

### TC-05 — Ambiguous/Empty Input Handling

| Field | Detail |
|---|---|
| **Type** | Edge Case / Input Validation |
| **Input** | Single character `"I"` — ambiguous / incomplete input |
| **Expected Behavior** | Bot asks a clarifying question and offers guidance on next steps |
| **Assertion** | `contains` — expected clarifying response text about what the user meant |

---

## ▶️ Running Tests

### Run Full Evaluation

```bash
promptfoo eval
```

### Run with Verbose Output

```bash
promptfoo eval --verbose
```

### Run a Specific Test by Description

```bash
promptfoo eval --filter-description "Strict Lead Qualification"
```

### Run Against a Specific Provider

```bash
promptfoo eval --providers openai:gpt-4o
```

### Run and Output Results to a File

```bash
promptfoo eval --output results/eval-output.json
```

---

## 📊 Viewing Results

After running `promptfoo eval`, launch the interactive results dashboard:

```bash
promptfoo view
```

This opens a browser-based UI at `http://localhost:15500` showing:
- Pass/Fail status per test case
- Model responses vs expected outputs
- Latency metrics per assertion
- Side-by-side comparison across providers (if multiple are configured)

---

## 🔍 Assertion Types

The following Promptfoo assertion types are used in this project:

| Type | Description |
|---|---|
| `contains` | Response must include the specified substring or concept |
| `not-contains` | Response must NOT include the specified substring |
| `equals` | Response must exactly match the expected value |

For a full list of supported assertion types, refer to the [Promptfoo Assertions Docs](https://promptfoo.dev/docs/configuration/expected-outputs).

---

## 🤖 AI Model Provider

This project runs evaluations locally using **Ollama** with the **Gemma 4 (`gemma4:e4b`)** model by Google.

| Property | Value |
|---|---|
| Provider ID | `ollama:gemma4:e4b` |
| Inference Type | Local (no cloud API key required) |
| Result Sharing | Disabled |
| Latest Results Write | Enabled |

Using a local model is ideal for:
- Privacy-sensitive testing (no data sent to external APIs)
- Cost-free iterations during test development
- Consistent, reproducible evaluation environments

---

## ⚠️ Known Issues & Observations

| # | Issue | Severity | Location |
|---|---|---|---|
| 1 | Assertion in TC-04 references `"EduSphere"` instead of `"AchieveTestPrep"` | Medium | `promptfooconfig.yaml`, TC-04 assert value |
| 2 | Typo in TC-03 input: `"CNA ro RN"` should be `"CNA to RN"` — may be intentional for fuzzy input testing | Low | `promptfooconfig.yaml`, TC-03 `user_input` |
| 3 | `scenarios`, `derivedMetrics`, and `extensions` are defined but empty | Low | `promptfooconfig.yaml` — can be removed to reduce clutter |
| 4 | No CI/CD pipeline configured (GitHub Actions absent) | Medium | Repository root |
| 5 | Single prompt label — the `label` field mirrors the full prompt body, making it verbose | Low | `prompts[].label` |

---

## 🏆 QA Best Practices Applied

- **Boundary Testing:** TC-05 tests single-character ambiguous input
- **Negative Testing:** TC-02 validates unqualified lead routing
- **Adversarial Testing:** TC-04 tests prompt injection / competitor manipulation resistance
- **Date Arithmetic Testing:** TC-03 validates temporal reasoning accuracy
- **Happy Path Testing:** TC-01 confirms the core booking flow works end-to-end
- **Assertion Specificity:** Mix of `contains`, `not-contains`, and `equals` for different validation needs

---

## 🚀 Extending the Test Suite

### Add a New Test Case

In `promptfooconfig.yaml`, append to the `tests` array:

```yaml
- description: Multi-program Interest
  vars:
    user_input: "I'm interested in both CNA to RN and Medical Assistant to RN. Which is faster?"
  assert:
    - type: contains
      value: The bot should compare both programs and ask clarifying questions to help the user decide.
```

### Add a Second Provider for Comparison

```yaml
providers:
  - id: ollama:gemma4:e4b
    label: Local Gemma 4
  - id: openai:gpt-4o
    label: GPT-4o Cloud
```

Running `promptfoo eval` will now test both models side-by-side.

### Add CI/CD with GitHub Actions

Create `.github/workflows/promptfoo-eval.yml`:

```yaml
name: Promptfoo Evaluation

on:
  push:
    branches: [master]
  pull_request:
    branches: [master]

jobs:
  evaluate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '20'
      - run: npm install -g promptfoo
      - run: promptfoo eval --no-cache
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
```

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/add-new-test-cases`
3. Add or update test cases in `promptfooconfig.yaml`
4. Run `promptfoo eval` to validate locally
5. Commit and push: `git push origin feature/add-new-test-cases`
6. Open a Pull Request with a clear description of new test coverage

---

## 📚 References

- [Promptfoo Official Documentation](https://promptfoo.dev/docs/configuration/guide)
- [Promptfoo — All Providers](https://promptfoo.dev/docs/providers)
- [Promptfoo — Assertions & Metrics](https://promptfoo.dev/docs/configuration/expected-outputs)
- [Promptfoo GitHub Examples](https://github.com/promptfoo/promptfoo/tree/main/examples)
- [Ollama — Local LLM Inference](https://ollama.com/)
- [Google Gemma Models](https://ai.google.dev/gemma)

---

## 👤 Author

**Imran Hassan**
GitHub: [@imranhassanaiub](https://github.com/imranhassanaiub)

---

*This README was prepared based on the project source at [github.com/imranhassanaiub/AI-Chatbot-Testing](https://github.com/imranhassanaiub/AI-Chatbot-Testing)*
