# Reusable GitHub Actions Workflows

This repository contains reusable GitHub Actions workflows to automate development workflows and team communication across **glocurrency** repositories.

---

## Table of Contents

- [1. Assign Reviewer (`assign-reviewer.yml`)](#1-assign-reviewer-assign-revieweryml)
- [2. Alert QA (`alert-qa.yml`)](#2-alert-qa-alert-qayml)

---

## Reusable Workflows

### 1. Assign Reviewer (`assign-reviewer.yml`)

Automatically assigns reviewers to a Pull Request (PR) and notifies them on Telegram.

#### Features
* Skips execution if the branch starts with `release-`.
* Only runs if no reviewers have been assigned/reviewed yet.
* Translates GitHub usernames to Telegram handles dynamically using a JSON-formatted secret mapping.

#### Inputs

| Name | Type | Required | Default | Description |
| :--- | :--- | :---: | :--- | :--- |
| `config-path` | `string` | No | `'.github/assign-config.yml'` | Path to the reviewer configuration file. |

#### Secrets

| Name | Required | Description |
| :--- | :---: | :--- |
| `telegram-token` | **Yes** | Telegram Bot token used to send notifications. |
| `telegram-chat-id` | **Yes** | Telegram Chat ID where notifications are sent. |
| `telegram-usernames` | No | JSON string mapping GitHub to Telegram handles (e.g. `{"github_user": "@telegram_user"}`). |

#### Setup & Prerequisites

You must create a configuration file (e.g., `.github/assign-config.yml`) in your repository for `auto-assign-action`:

```yaml
# .github/assign-config.yml
addReviewers: true
addAssignees: false
reviewers:
  - github_user1
  - github_user2
numberOfReviewers: 1
```

#### Usage Example

```yaml
name: Auto Assign Reviewers

on:
  pull_request:
    types: [opened, ready_for_review]

jobs:
  assign-and-notify:
    uses: glocurrency/reusable-actions/.github/workflows/assign-reviewer.yml@main
    with:
      config-path: '.github/assign-config.yml'
    secrets:
      telegram-token: ${{ secrets.TELEGRAM_TOKEN }}
      telegram-chat-id: ${{ secrets.TELEGRAM_CHAT_ID }}
      telegram-usernames: '{"github_user1": "@telegram_handle1", "github_user2": "@telegram_handle2"}'
```

---

### 2. Alert QA (`alert-qa.yml`)

Sends structured build/test alerts to Telegram and randomly tags a QA engineer from a configured team list.

#### Features
* Randomly tags a QA engineer from a provided JSON array.
* Sets a green prefix (`✅ OK`) for `'success'` status and red prefix (`🚨 Alert`) for `'failure'`.
* Uses HTML formatting in Telegram messages.

#### Inputs

| Name | Type | Required | Default | Description |
| :--- | :--- | :---: | :--- | :--- |
| `message` | `string` | **Yes** | — | Message body text to send. |
| `status` | `string` | No | `'failure'` | Job status (`'success'` or `'failure'`) to format the alert prefix. |

#### Secrets

| Name | Required | Description |
| :--- | :---: | :--- |
| `telegram-token` | **Yes** | Telegram Bot token used to send notifications. |
| `telegram-chat-id` | **Yes** | Telegram Chat ID where notifications are sent. |
| `telegram-qa-usernames` | No | JSON array of QA Telegram handles (e.g., `["@qa_alice", "@qa_bob"]`). Falls back to notifying `"QA Team"` if not provided. |

#### Usage Example

```yaml
name: Continuous Integration

on:
  push:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4
      - name: Run Tests
        run: npm test

  notify-qa-success:
    needs: build-and-test
    if: success()
    uses: glocurrency/reusable-actions/.github/workflows/alert-qa.yml@main
    with:
      status: 'success'
      message: 'Build and unit tests passed on main!'
    secrets:
      telegram-token: ${{ secrets.TELEGRAM_TOKEN }}
      telegram-chat-id: ${{ secrets.TELEGRAM_CHAT_ID }}
      telegram-qa-usernames: '["@qa_alice", "@qa_bob"]'

  notify-qa-failure:
    needs: build-and-test
    if: failure()
    uses: glocurrency/reusable-actions/.github/workflows/alert-qa.yml@main
    with:
      status: 'failure'
      message: 'Build or tests failed on main.'
    secrets:
      telegram-token: ${{ secrets.TELEGRAM_TOKEN }}
      telegram-chat-id: ${{ secrets.TELEGRAM_CHAT_ID }}
      telegram-qa-usernames: '["@qa_alice", "@qa_bob"]'
```
