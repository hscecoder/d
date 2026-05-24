
# DevOps Workflow for Secure Infrastructure Validation & Deployment

## 1. Project Structure

```bash
secure-devops-workflow/
│
├── app.js
├── package.json
├── deployment.json
├── Dockerfile
├── .snyk
│
├── policy/
│   └── deployment.rego
│
├── .husky/
│   └── pre-commit
│
├── .github/
│   └── workflows/
│       └── ci.yml
│
└── README.md
```

---

# 2. Create Simple Express.js Web Service

## app.js

```javascript
const express = require("express");

const app = express();
const PORT = process.env.PORT || 3000;

app.get("/health", (req, res) => {
  res.send("Application Healthy");
});

app.get("/status", (req, res) => {
  res.json({
    service: "inventory-service",
    status: "running",
    uptime: process.uptime(),
    timestamp: new Date()
  });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

---

## package.json

```json
{
  "name": "secure-devops-workflow",
  "version": "1.0.0",
  "description": "DevOps Security Workflow",
  "main": "app.js",
  "scripts": {
    "start": "node app.js",
    "test": "echo \"No tests configured\"",
    "snyk-test": "snyk test",
    "docker-build": "docker build -t secure-node-app ."
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "husky": "^9.0.11"
  }
}
```

---

# 3. Infrastructure Policy Validation using OPA

## deployment.json

```json
{
  "app": "inventory-service",
  "cpu": "500m",
  "memory": "512Mi",
  "environment": "production"
}
```

---

## policy/deployment.rego

```rego
package deployment

deny[msg] {
    input.environment == ""
    msg := "Environment cannot be empty"
}

deny[msg] {
    not input.cpu
    msg := "CPU limit must be defined"
}

deny[msg] {
    input.environment == "production"
    memory := to_number(trim_suffix(input.memory, "Mi"))
    memory < 512
    msg := "Production deployments must have at least 512Mi memory"
}
```

---

## Run OPA Validation

Install OPA:

```bash
brew install opa
```

OR

```bash
wget https://openpolicyagent.org/downloads/latest/opa_linux_amd64_static
chmod +x opa_linux_amd64_static
sudo mv opa_linux_amd64_static /usr/local/bin/opa
```

Validate:

```bash
opa eval --input deployment.json --data policy/ 'data.deployment.deny'
```

Expected Output:

```json
{
  "result": [
    {
      "expressions": [
        {
          "value": [],
          "text": "data.deployment.deny"
        }
      ]
    }
  ]
}
```

---

# 4. Git & GitHub Workflow

## Initialize Repository

```bash
git init
git add .
git commit -m "Initial secure DevOps workflow setup"
```

Create GitHub repository and push:

```bash
git remote add origin https://github.com/your-username/secure-devops-workflow.git
git branch -M main
git push -u origin main
```

Use meaningful commits like:

```bash
git commit -m "Added OPA deployment policy validation"
git commit -m "Configured GitHub Actions CI pipeline"
git commit -m "Integrated Snyk security scanning"
```

---

# 5. Dockerization

## Dockerfile

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

---

## Build Docker Image

```bash
docker build -t secure-node-app .
```

---

## Run Docker Container

```bash
docker run -p 3000:3000 secure-node-app
```

Test routes:

```bash
http://localhost:3000/health
```

```bash
http://localhost:3000/status
```

---

# 6. Snyk Security Checks

## Install Snyk

```bash
npm install -g snyk
```

Authenticate:

```bash
snyk auth
```

---

## Scan Dependencies

```bash
snyk test
```

---

## Scan Docker Image

```bash
docker build -t secure-node-app .

snyk container test secure-node-app
```

---

# 7. Husky Git Hook

## Install Husky

```bash
npm install husky --save-dev
npx husky init
```

---

## .husky/pre-commit

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

echo "Running Snyk Security Scan..."

npm install

snyk test

if [ $? -ne 0 ]; then
  echo "Snyk vulnerabilities detected. Commit blocked."
  exit 1
fi
```

Make executable:

```bash
chmod +x .husky/pre-commit
```

Now insecure commits are blocked automatically.

---

# 8. GitHub Actions CI/CD Pipeline

## .github/workflows/ci.yml

```yaml
name: Secure DevOps Pipeline

on:
  push:
    branches:
      - main

jobs:
  security-validation:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18

      - name: Install Dependencies
        run: npm install

      - name: Install OPA
        run: |
          curl -L -o opa https://openpolicyagent.org/downloads/latest/opa_linux_amd64_static
          chmod +x opa
          sudo mv opa /usr/local/bin/

      - name: Validate Infrastructure Policies
        run: |
          opa eval --input deployment.json \
          --data policy/ \
          'data.deployment.deny'

      - name: Install Snyk
        run: npm install -g snyk

      - name: Authenticate Snyk
        run: snyk auth ${{ secrets.SNYK_TOKEN }}

      - name: Run Dependency Scan
        run: snyk test

      - name: Build Docker Image
        run: docker build -t secure-node-app .

      - name: Run Container Scan
        run: snyk container test secure-node-app
```

---

# 9. Add GitHub Secret for Snyk

Go to:

GitHub Repository → Settings → Secrets and Variables → Actions

Add:

```text
SNYK_TOKEN=your_snyk_token
```

Get token from:

[Snyk Account Settings](https://app.snyk.io/account?utm_source=chatgpt.com)

---

# 10. Expected Screenshots for Submission

Students should capture:

### OPA Validation

* Successful OPA policy check terminal output

### Docker Execution

* Running container
* `/health` route response
* `/status` route JSON response

### Snyk Scan Results

* Dependency vulnerability scan
* Docker image scan

### GitHub Actions Pipeline

* Green successful workflow execution

