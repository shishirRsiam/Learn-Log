## 📧 Goal: Send an email on push using GitHub Actions

---

### ✅ Step 1: Create an App Password (Gmail)

> **Skip this if you're using another provider like Mailgun or SendGrid.**

1. Turn on **2-step verification** for your Gmail account.
2. Go to: [https://myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords)
3. Generate an **App Password** for "Mail" + "Other".
4. Copy that 16-digit app password.

---

### 🔐 Step 2: Add GitHub Secrets

Go to your **repo → Settings → Secrets and variables → Actions → New repository secret** and add:

| Name              | Value                      |
|-------------------|----------------------------|
| `EMAIL_USERNAME`  | your Gmail (e.g. `you@gmail.com`) |
| `EMAIL_PASSWORD`  | your **app password**      |
| `EMAIL_TO`        | where the email should go  |

---

### 📄 Step 3: Create Workflow File

Create or update `.github/workflows/email-on-push.yml`:

```yaml
name: Send Email on Push

on:
  push:
    branches: [main] # or any branch you prefer

jobs:
  notify:
    runs-on: ubuntu-latest

    steps:
    - name: Send Email
      uses: dawidd6/action-send-mail@v3
      with:
        server_address: smtp.gmail.com
        server_port: 465
        username: ${{ secrets.EMAIL_USERNAME }}
        password: ${{ secrets.EMAIL_PASSWORD }}
        subject: "🚀 New push to main branch"
        to: ${{ secrets.EMAIL_TO }}
        from: GitHub Actions <${{ secrets.EMAIL_USERNAME }}>
        content_type: text/plain
        body: |
          A new push was made to the main branch.

          Repo: ${{ github.repository }}
          Commit: ${{ github.sha }}
          By: ${{ github.actor }}
          Message: ${{ github.event.head_commit.message }}
```

---

### ✨ Result

Every time someone pushes to `main`, GitHub will send an email notification with commit info.

---

## 📌 Notes

- You can customize the subject, body, and when it triggers (`push`, `pull_request`, etc).
- You can even add **HTML content** if you change `content_type` to `text/html`.

