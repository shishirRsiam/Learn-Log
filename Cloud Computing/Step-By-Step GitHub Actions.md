Let’s dive into **GitHub Actions**, step-by-step, from beginner to advanced concepts.

---

## 🔰 What is GitHub Actions?

**GitHub Actions** is a **CI/CD** (Continuous Integration / Continuous Deployment) tool built into GitHub. It allows you to **automate workflows** like:

- Running tests when code is pushed
- Deploying to a server or cloud
- Sending notifications (e.g., to Slack)
- Running cron jobs (scheduled tasks)

---

## 📦 Core Concepts

### 1. **Workflow**
A YAML file that defines automation steps. Located inside:  
`/.github/workflows/your-workflow-name.yml`

### 2. **Event**
What triggers the workflow? Examples:
- `push` (to any branch or a specific one)
- `pull_request`
- `schedule` (cron jobs)
- `workflow_dispatch` (manual trigger)

### 3. **Job**
Each workflow contains one or more **jobs**. A job is a set of steps that run on the same machine (VM).

### 4. **Step**
A single command or action within a job.

### 5. **Action**
Reusable pieces of code (like plugins). There are official and community-made actions on [GitHub Marketplace](https://github.com/marketplace?type=actions).

---

## 🛠 Basic Example: Run Tests on Push

```yaml
# .github/workflows/test.yml
name: Run Tests

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: 3.10

    - name: Install dependencies
      run: pip install -r requirements.txt

    - name: Run tests
      run: pytest
```

### What it does:
1. Runs when someone pushes code.
2. Sets up Python.
3. Installs dependencies.
4. Runs tests.

---

## 📚 Common `on` Triggers

```yaml
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
  schedule:
    - cron: '0 0 * * *'  # every day at midnight UTC
  workflow_dispatch:     # manual trigger
```

---

## 🔁 Reusable Actions (Marketplace Example)

Let’s say you want to send a Slack notification.

```yaml
- uses: rtCamp/action-slack-notify@v2
  with:
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}
    message: "Deployment complete ✅"
```

---

## 🔐 Secrets

Use `secrets` to store passwords, tokens, API keys.

Access them in the workflow:

```yaml
env:
  API_KEY: ${{ secrets.API_KEY }}
```

Set secrets in:  
GitHub repo → **Settings** → **Secrets and variables**

---

## 🧪 Matrix Builds

Test against multiple versions of Python or Node.js:

```yaml
strategy:
  matrix:
    python-version: [3.8, 3.9, 3.10]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.8, 3.9, 3.10]

    steps:
    - uses: actions/setup-python@v4
      with:
        python-version: ${{ matrix.python-version }}
```

---

## 🚀 Deployment Example

Here’s a simple example of deploying a static site to GitHub Pages:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Build
      run: |
        npm install
        npm run build
    - name: Deploy
      uses: peaceiris/actions-gh-pages@v3
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./build
```

---

## 🧠 Best Practices

- Use secrets for anything sensitive
- Separate testing and deployment workflows
- Use job names that describe what’s happening
- Leverage caching to speed up builds (e.g., `actions/cache`)
- Keep workflows in `.github/workflows/` directory

---

## 🤔 Want Hands-On Practice?

Here are a few ideas:
- ✅ Run tests when pushing to a feature branch
- 🚀 Auto-deploy a static site (e.g., React or Jekyll)
- 🧹 Auto-format code using `black`, `prettier`, or `eslint`
- 🧪 Setup CI for a Django or Flask app

