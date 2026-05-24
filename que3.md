# CI/CD Pipeline Implementation using GitHub [GitHub Actions](https://github.com/features/actions?utm_source=chatgpt.com)

This project creates a complete automated CI/CD pipeline for application deployment using GitHub Actions.

---

# Objective

Implement a CI/CD pipeline that:

* Automatically triggers on code push
* Builds the application
* Runs automated tests
* Deploys the application
* Provides logs and monitoring

---

# Project Structure

Example Node.js application structure:

```plaintext
project/
│── .github/
│   └── workflows/
│       └── ci-cd.yml
│── src/
│── tests/
│── package.json
│── Dockerfile
│── README.md
```

---

# Prerequisites

Install:

* Git
* Node.js
* GitHub account

Useful links:

* [Node.js Official Website](https://nodejs.org?utm_source=chatgpt.com)
* [Git Official Website](https://git-scm.com?utm_source=chatgpt.com)
* [GitHub](https://github.com?utm_source=chatgpt.com)

---

# Step 1: Create GitHub Repository

Create a new repository on GitHub.

Clone it locally:

```bash
git clone https://github.com/username/project.git
cd project
```

---

# Step 2: Sample Node.js Application

Initialize project:

```bash
npm init -y
```

Install dependencies:

```bash
npm install express
npm install --save-dev jest
```

Example `app.js`:

```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
    res.send('CI/CD Pipeline Running');
});

module.exports = app;
```

Example `server.js`:

```javascript
const app = require('./app');

app.listen(3000, () => {
    console.log('Server running on port 3000');
});
```

---

# Step 3: Add Test Cases

Create `tests/app.test.js`

```javascript
const request = require('supertest');
const app = require('../app');

test('GET /', async () => {
    const response = await request(app).get('/');
    expect(response.statusCode).toBe(200);
});
```

Install testing package:

```bash
npm install --save-dev supertest
```

Update `package.json`:

```json
"scripts": {
  "test": "jest"
}
```

---

# Step 4: Create GitHub Actions Workflow

Create file:

```plaintext
.github/workflows/ci-cd.yml
```

Add pipeline configuration:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:

  build-and-test:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Code
      uses: actions/checkout@v4

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: 18

    - name: Install Dependencies
      run: npm install

    - name: Run Tests
      run: npm test

  deploy:
    needs: build-and-test
    runs-on: ubuntu-latest

    steps:
    - name: Deploy Application
      run: echo "Deploying application..."
```

---

# Step 5: Push Code to GitHub

```bash
git add .
git commit -m "Added CI/CD pipeline"
git push origin main
```

Pipeline automatically starts after push.

---

# Step 6: Deployment Automation

Example deployment using Docker:

## Dockerfile

```dockerfile
FROM node:18

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["node", "server.js"]
```

Update deploy stage:

```yaml
deploy:
  needs: build-and-test
  runs-on: ubuntu-latest

  steps:
  - name: Checkout Code
    uses: actions/checkout@v4

  - name: Build Docker Image
    run: docker build -t myapp .

  - name: Run Container
    run: docker run -d -p 3000:3000 myapp
```

---

# Step 7: Pipeline Monitoring & Logs

GitHub Actions provides:

* Build logs
* Test reports
* Deployment logs
* Workflow status
* Failure notifications

View pipeline:

1. Open GitHub repository
2. Go to **Actions** tab
3. Select workflow run

Documentation:

* [GitHub Actions Documentation](https://docs.github.com/actions?utm_source=chatgpt.com)

---

# CI/CD Workflow Diagram

```plaintext
Developer Pushes Code
          │
          ▼
GitHub Actions Triggered
          │
          ▼
Install Dependencies
          │
          ▼
Run Tests
          │
          ▼
Build Application
          │
          ▼
Deploy Application
          │
          ▼
Monitoring & Logs
```

---
