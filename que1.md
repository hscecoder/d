Here’s a complete secure backend project design that satisfies all the assignment requirements using:

* Node.js + Express
* Open Policy Agent (OPA)
* dotenv for secrets
* Git + GitHub
* Husky + Snyk
* GitHub Actions CI

---

# Project Structure

```txt
secure-backend/
│
├── .github/
│   └── workflows/
│       └── ci.yml
│
├── opa/
│   └── policy.rego
│
├── src/
│   ├── middleware/
│   │   └── auth.js
│   ├── routes/
│   │   └── secure.js
│   └── server.js
│
├── .env
├── .gitignore
├── package.json
├── README.md
└── snyk-test.sh
```

---

# 1. Initialize Project

```bash
mkdir secure-backend
cd secure-backend

npm init -y
```

Install dependencies:

```bash
npm install express axios dotenv
npm install --save-dev husky snyk
```

---

# 2. Environment Variables (.env)

Create `.env`

```env
PORT=3000
OPA_URL=http://localhost:8181/v1/data/httpapi/authz
```

---

# 3. Git Ignore

Create `.gitignore`

```gitignore
node_modules/
.env
.snyk
```

This ensures secrets are not committed.

---

# 4. Express Server

## `src/server.js`

```javascript
require('dotenv').config();

const express = require('express');
const secureRoute = require('./routes/secure');

const app = express();

app.use(express.json());

app.use('/secure', secureRoute);

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});
```

---

# 5. Protected Route

## `src/routes/secure.js`

```javascript
const express = require('express');
const authMiddleware = require('../middleware/auth');

const router = express.Router();

router.get('/', authMiddleware, (req, res) => {
    res.status(200).json({
        message: 'Access granted to secure route'
    });
});

module.exports = router;
```

---

# 6. OPA Authorization Middleware

## `src/middleware/auth.js`

```javascript
const axios = require('axios');

module.exports = async (req, res, next) => {
    try {
        const role = req.headers.role;

        const opaResponse = await axios.post(process.env.OPA_URL, {
            input: {
                role: role
            }
        });

        const allowed = opaResponse.data.result.allow;

        if (!allowed) {
            return res.status(403).json({
                message: 'Access denied'
            });
        }

        next();

    } catch (error) {
        console.error(error.message);

        res.status(500).json({
            message: 'Authorization error'
        });
    }
};
```

---

# 7. OPA Policy

Create folder:

```bash
mkdir opa
```

## `opa/policy.rego`

```rego
package httpapi.authz

default allow = false

allow {
    input.role == "admin"
}
```

---

# 8. Run OPA

Install OPA:

[Open Policy Agent Official Site](https://www.openpolicyagent.org/?utm_source=chatgpt.com)

Run OPA server:

```bash
opa run --server opa/policy.rego
```

---

# 9. Test the API

Start server:

```bash
node src/server.js
```

## Authorized Request

```bash
curl -H "role: admin" http://localhost:3000/secure
```

Response:

```json
{
  "message": "Access granted to secure route"
}
```

## Unauthorized Request

```bash
curl -H "role: user" http://localhost:3000/secure
```

Response:

```json
{
  "message": "Access denied"
}
```

---

# 10. Husky Setup

Initialize Husky:

```bash
npx husky-init
npm install
```

---

# 11. Snyk Authentication

Create account and authenticate:

[Snyk Official Website](https://snyk.io/?utm_source=chatgpt.com)

```bash
npx snyk auth
```

---

# 12. Snyk Test Script

## `snyk-test.sh`

```bash
#!/bin/sh

npx snyk test

if [ $? -ne 0 ]; then
  echo "Snyk found vulnerabilities. Commit blocked."
  exit 1
fi
```

Make executable:

```bash
chmod +x snyk-test.sh
```

---

# 13. Configure Pre-Commit Hook

Edit:

## `.husky/pre-commit`

```bash
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

./snyk-test.sh
```

Now commits fail if vulnerabilities are detected.

---

# 14. GitHub Actions CI/CD

Create:

## `.github/workflows/ci.yml`

```yaml
name: Security CI Pipeline

on:
  push:
    branches:
      - main
      - master

jobs:
  security-check:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install Dependencies
        run: npm install

      - name: Install Snyk
        run: npm install -g snyk

      - name: Run Snyk Security Scan
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        run: snyk test
```

---

# 15. Add GitHub Secret

In GitHub repository:

```txt
Settings → Secrets and Variables → Actions
```

Add:

```txt
SNYK_TOKEN=your_snyk_token
```

Get token from:

[Snyk Account Settings](https://app.snyk.io/account?utm_source=chatgpt.com)

---

# 16. Git Commands

Initialize repository:

```bash
git init
git add .
git commit -m "Initial secure backend setup"
```

Create GitHub repo and push:

```bash
git remote add origin <your-repo-url>

git branch -M main

git push -u origin main
```

---

# 17. Recommended Commit History

Example:

```txt
feat: initialize express backend
feat: add OPA authorization
feat: add secure route
chore: configure dotenv secrets
feat: add husky pre-commit hook
feat: integrate snyk security scan
ci: add github actions pipeline
docs: update README
```

---

# 18. README Content

Your README should include:

* Project setup
* OPA setup
* Running the API
* Running security checks
* CI pipeline explanation
* Screenshots

---

# 19. Required Screenshots

Take screenshots of:

### API Authorization

* Admin access success
* User access denied

### Husky + Snyk

* Failed commit because vulnerabilities detected

### GitHub Actions

* Successful CI workflow execution

---

