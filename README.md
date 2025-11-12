# WebAppEval: A Benchmark for Autonomous Web Agents and Agentic Test Automation

## Introduction

**WebAppEval** is a benchmark designed to evaluate **autonomous web agents** and **LLM-driven test automation systems**.  
It provides realistic, containerized web applications—such as **Magento**, **Odoo**, **Jorani**, and **Redmine**—and a standardized evaluation protocol that measures task performance in a reproducible and automated way.

While existing benchmarks like *MiniWoB++*, *Online-Mind2Web*, and *WebArena* have driven progress in this field, they often lack realism or reproducibility.  
**WebAppEval** fills this gap by combining:

- **Realistic enterprise-scale web apps**
- **JSON-based task schemas**
- **Automatic matchers for evaluation**
- **Unified scoring metrics (TSR, SCR)**
  **Reproducible Dockerized setup**

## 🚀 Key Features

- **Multi-domain coverage:** Magento (e-commerce), Odoo (ERP), Jorani (HRM), Redmine (project management).
- **Declarative JSON task schema** defining start state, action goals, and evaluation criteria.
- **Automatic evaluation** with multiple matchers:
  - `string_match`, `url_match`, `dom_match`, and `semantic_match`.
- **Supports multiple agent frameworks:** HxAgent, SeeAct, Li et al., and custom LLM agents.
- **Fully Dockerized:** one command to spin up all web apps and evaluation modules.

## ⚙️ Setup

### 1️. Requirements

- [Docker](https://www.docker.com/) and [Docker Compose](https://docs.docker.com/compose/)
- Python ≥ **3.10**
- `pip` for installing dependencies

### 2️. Clone the repository

```bash
git clone https://github.com/nguyen0710/WebAppEval.git
cd WebAppEval
```

### 3️. Launch the environments

You can run each web application individually by navigating into its environment folder.
#### Example: Magento
```bash
cd environments/magento2
docker compose up -d
```
After a few minutes, the application will be available in your browser. 
The URL and credentials for each environment are defined in **env_config.json**.

## Usage
### 1️. Example task definition
Each task in WebAppEval is defined using a unified JSON schema and stored in the `datasets/tasks.json`.

```json
{
  "task_id": "7",
  "task_description": "Subscribe to the website via the subscription field with email 'account.test.4@mynes.com'.",
  "task_type": "operation",
  "start_url": "__SHOPPING__",
  "require_login": false,
  "eval": {
    "eval_type": ["dom_match"],
    "dom_match": {
      "url": "last",
      "dom_extractor": "document.querySelector('.message-success.success.message > div').innerText",
      "match_type": "contains",
      "match_value": "Thank you for your subscription.",
      "description": "Verify that the success message is displayed after subscribing."
    }
  }
}
```
**Explanation:**
- `task_id`: Unique ID of the task.
- `task_description`: The goal the agent must perform.
- `task_type`: The nature of the action (e.g., lookup, operation).
- `start_url`: Initial page (mapped from env_config.json).
- `eval`: Defines the evaluation method. Here, the `dom_match` matcher extracts the success message from the page
and checks whether it contains the text "Thank you for your subscription.".

### 2. Evaluate a task
Each task includes an **`eval`** field that defines how success is determined after the agent completes the action.  
The evaluator uses one or more *matchers* — rules that verify different aspects of the final web page or state,  
such as **`string_match`**, **`dom_match`**, **`url_match`**, or **`semantic_match`**.

For example:

```json
"eval": {
  "eval_type": ["dom_match", "url_match"],
  "dom_match": {
    "url": "last",
    "dom_extractor": "document.querySelector('.message-success.success.message > div').innerText",
    "match_type": "contains",
    "match_value": "Thank you for your subscription.",
    "description": "Verify that the success message is displayed after subscribing."
  },
  "url_match": {
    "match_type": "contains",
    "match_value": "newsletter/subscriber/success",
    "description": "Ensure the page navigates to the subscription success URL."
  }
}
```
**Explanation:**

- `eval_type`: A list of matchers to apply (e.g., `["string_match"]` or multiple like `["dom_match", "url_match"]`).
- `match_type`: The comparison rule (`exact`, `contains`).
- `match_value`: The expected value used for comparison.
- `description`: A human-readable explanation of the check’s purpose.

In this example, the evaluator will perform **two checks**:
1. Extract the success message and ensure it contains the correct text.  
2. Verify that the browser navigated to a URL containing `newsletter/subscriber/success`.

✅ Both conditions must pass for the task to be considered **successful**.

**Example:**
The evaluation logic is provided in the **`evaluate/`** directory as a reference for agents using Selenium or Playwright.

### 3️. Supported Matchers

| Matcher | Description | Typical Use Case |
|----------|--------------|------------------|
| **`string_match`** | Compares textual outputs (e.g., success messages, labels). | Checking if a confirmation message appears. |
| **`dom_match`** | Extracts and validates elements or attributes from the DOM. | Verifying dynamic content on a web page. |
| **`url_match`** | Ensures the agent navigated to the correct URL. | Checking redirection or flow correctness. |
| **`semantic_match`** | Uses LLM-based similarity for flexible text comparison. | Matching paraphrased or non-exact outputs. |

