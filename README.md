#  React App Deployment to EC2 using GitHub Actions (pnpm + PM2)

This project demonstrates **automated CI/CD deployment of a React production build** to an AWS EC2 server using:

* GitHub Actions
* AWS EC2
* pnpm package manager
* PM2 process manager
* Secure file transfer (SCP)

Whenever code is pushed to the `master` branch, GitHub automatically:

✔ Installs dependencies
✔ Builds the React app
✔ Packages the production build
✔ Uploads it to EC2
✔ Serves the app using PM2

---

##  Deployment Workflow Overview

###  Trigger

Deployment runs automatically on:

```
push → master branch
```

---

##  Automated Deployment Steps

1. Checkout latest repository code.
2. Install pnpm package manager.
3. Setup Node.js environment.
4. Install project dependencies.
5. Build production React app.
6. Create compressed deployment package (`dist`).
7. Upload package to EC2 server using SCP.
8. Extract files on EC2.
9. Serve React build using PM2.

---

##  GitHub Actions Workflow

```yaml
name: Deploy to EC2 on Push to Main

on:
  push:
    branches:
      - master

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 7

      - name: Setup node
        uses: actions/setup-node@v3
        with:
          node-version: "20.x"
          cache: pnpm

      - name: Install Dependencies
        run: pnpm install

      - name: Build production
        run: pnpm run build

      - name: create deployment package
        run: |
          tar -czf deployment-package.tar.gz dist

      - name: Copy files to EC2
        uses: appleboy/scp-action@v0.1.4
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_KEY }}
          source: "deployment-package.tar.gz"
          target: "~/react-app/"

      - name: Serve Dist and Cleanup
        uses: appleboy/ssh-action@v0.1.6
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_KEY }}
          script: |
            cd ~/react-app
            tar -xzf deployment-package.tar.gz
            rm -rf deployment-package.tar.gz
            export PATH=/home/ubuntu/.nvm/versions/node/v22.17.1/bin/pm2:$PATH
            pm2 serve dist 3000 --name "react-app" --spa --no-daemon
```

---

##  Required GitHub Secrets

Go to:

```
Repository → Settings → Secrets and variables → Actions
```

Add the following secrets:

| Secret   | Description                    |
| -------- | ------------------------------ |
| EC2_HOST | Public IP of EC2 instance      |
| EC2_USER | SSH username (example: ubuntu) |
| EC2_KEY  | Private SSH key                |

---

##  EC2 Server Requirements

Make sure your EC2 instance has:

* Node.js installed via NVM
* pnpm installed
* PM2 installed globally
* Git installed
* Target directory created

Create deployment directory:

```bash
mkdir ~/react-app
```

Install PM2:

```bash
npm install -g pm2
```

---

##  How the App Runs in Production

The React production build is served using:

```bash
pm2 serve dist 3000 --name "react-app" --spa
```

This means:

* App runs on port **3000**
* Supports SPA routing
* Managed by PM2 for reliability

---

##  Tech Stack

* React
* pnpm
* GitHub Actions
* AWS EC2
* PM2
* SCP file transfer

---
