Suggested Folder Structure
-------------------------------------------

``` billing-app/
├── client/
│   ├── public/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── services/
│   │   ├── hooks/
│   │   ├── contexts/
│   │   └── App.tsx
│   └── .env
├── server/
│   ├── src/
│   │   ├── controllers/
│   │   ├── routes/
│   │   ├── models/
│   │   ├── middlewares/
│   │   ├── services/
│   │   ├── utils/
│   │   └── index.ts
│   └── .env
├── infra/
│   ├── aws/
│   │   ├── lambdas/
│   │   ├── terraform/
│   │   └── s3/
│   └── docker/
│       ├── Dockerfile.client
│       └── Dockerfile.server
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── cd.yml
├── README.md
└── package.json ```

########### ISSUES ############

| #  | Title                      | Labels                   | Description                                      |
| -- | -------------------------- | ------------------------ | ------------------------------------------------ |
| 1  | Setup MERN boilerplate     | backend,frontend         | Scaffold React app, Express server, basic Docker |
| 2  | Add JWT authentication     | backend,security         | Role-based login, signup                         |
| 3  | Product CRUD UI & API      | backend,frontend         | Endpoints + admin interface                      |
| 4  | Order+Invoice workflow     | backend,frontend,billing | Create orders, compute total, generate invoice   |
| 5  | Generate PDF invoice       | backend,billing          | Use PDFKit, store on S3                          |
| 6  | AWS infra (Terraform)      | infra,aws                | S3 buckets, Lambda, ECS tasks                    |
| 7  | Integrate OpenAI assistant | ai,enhancement           | Chat/voice assistant for staff                   |
| 8  | Dockerize client & server  | infra,docker             | Add Dockerfiles and docker-compose               |
| 9  | CI/CD pipeline             | infra,devops             | Set up GitHub Actions                            |
| 10 | Deploy on AWS              | infra,aws,devops         | End-to-end deployment scripts                    |


########### .github/workflows/ci.yml ###################
name: CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    services:
      mongo:
        image: mongo:6
        ports: ['27017:27017']
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install dependencies (server + client)
        run: |
          cd server && npm install
          cd ../client && npm install

      - name: Lint & Test Backend
        run: |
          cd server
          npm run lint
          npm test

      - name: Lint & Test Frontend
        run: |
          cd client
          npm run lint
          npm test


######### .github/workflows/cd.yml

name: CD

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Setup AWS Credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Deploy Infra with Terraform
        run: |
          cd infra/aws/terraform
          terraform init
          terraform apply -auto-approve

      - name: Deploy Server (ECS / Lambda)
        run: |
          cd infra/aws
          ./deploy_server.sh

      - name: Deploy Client (S3 + CloudFront)
        run: |
          cd infra/aws
          ./deploy_client.sh


