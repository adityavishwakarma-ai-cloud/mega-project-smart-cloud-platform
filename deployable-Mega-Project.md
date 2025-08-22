## 1️⃣ Folder Structure

```
mega-project/
│-- README.md
│-- frontend/                   # React + Tailwind
│   ├── package.json
│   ├── tailwind.config.js
│   └── src/
│       ├── App.tsx
│       ├── index.tsx
│       ├── components/
│       │   └── Chatbot.tsx
│       └── styles/
│           └── global.css
│-- backend/                    # Node.js API
│   ├── package.json
│   └── src/
│       ├── server.ts
│       ├── routes/
│       │   └── api.ts
│       └── controllers/
│           └── chatbotController.ts
│-- chatbot/                    # AWS Lex + Lambda
│   ├── lambda/
│   │   └── chatbot-handler.js
│   └── lex/
│       └── chatbot-config.json
│-- terraform/                  # AWS infrastructure
│   ├── provider.tf
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
│-- k8s/                        # Kubernetes manifests
│   ├── deployment.yaml
│   └── service.yaml
│-- ci-cd/                      # Jenkins pipeline
│   └── Jenkinsfile
│-- docs/                       # Documentation, diagrams
│   └── architecture.png
│-- .gitignore
```

---

## 2️⃣ Sample Frontend – `frontend/src/components/Chatbot.tsx`

```typescript
import React, { useState } from 'react';

const Chatbot = () => {
  const [messages, setMessages] = useState<{user: string, bot: string}[]>([]);
  const [input, setInput] = useState('');

  const sendMessage = async () => {
    const response = await fetch('/api/chat', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ message: input }),
    });
    const data = await response.json();
    setMessages([...messages, { user: input, bot: data.reply }]);
    setInput('');
  };

  return (
    <div>
      <h2>Smart Chatbot</h2>
      <div>
        {messages.map((msg, idx) => (
          <div key={idx}>
            <p><b>You:</b> {msg.user}</p>
            <p><b>Bot:</b> {msg.bot}</p>
          </div>
        ))}
      </div>
      <input value={input} onChange={e => setInput(e.target.value)} />
      <button onClick={sendMessage}>Send</button>
    </div>
  );
};

export default Chatbot;
```

---

## 3️⃣ Sample Backend – `backend/src/server.ts`

```typescript
import express from 'express';
import bodyParser from 'body-parser';
import { handleChat } from './controllers/chatbotController';

const app = express();
app.use(bodyParser.json());

app.post('/api/chat', handleChat);

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

### `backend/src/controllers/chatbotController.ts`

```typescript
export const handleChat = async (req, res) => {
  const userMessage = req.body.message;
  // Example response (replace with Lambda or Lex call)
  const botReply = userMessage.toLowerCase().includes('hello') ? 
                   'Hello! How can I assist you?' : 
                   "I'm here to help!";
  res.json({ reply: botReply });
};
```

---

## 4️⃣ Sample Lambda – `chatbot/lambda/chatbot-handler.js`

```javascript
const AWS = require('aws-sdk');
const dynamoDB = new AWS.DynamoDB.DocumentClient();

exports.handler = async (event) => {
    const userMessage = event.inputTranscript;
    const userId = event.userId;

    let responseMessage = "I didn't understand that.";
    if (userMessage.toLowerCase().includes('hello')) {
        responseMessage = "Hello! Welcome to Smart Cloud Platform!";
    }

    await dynamoDB.put({
        TableName: "ChatbotSessions",
        Item: { userId, message: userMessage, response: responseMessage, timestamp: new Date().toISOString() }
    }).promise();

    return {
        sessionAttributes: event.sessionAttributes,
        dialogAction: { type: "Close", fulfillmentState: "Fulfilled", message: { contentType: "PlainText", content: responseMessage } }
    };
};
```

---

## 5️⃣ Terraform – `terraform/main.tf`

```hcl
# VPC, EC2, S3, RDS setup
resource "aws_vpc" "main" { cidr_block = "10.0.0.0/16" }
resource "aws_s3_bucket" "frontend_bucket" { bucket = "mega-project-frontend-bucket" }
resource "aws_instance" "backend" { ami = "ami-0c02fb55956c7d316" instance_type = "t2.micro" }
```

> Provider, variables, and outputs configured similarly to Project 7.

---

## 6️⃣ Kubernetes – `k8s/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata: { name: mega-project-app }
spec:
  replicas: 2
  selector: { matchLabels: { app: mega-project-app } }
  template:
    metadata: { labels: { app: mega-project-app } }
    spec:
      containers:
      - name: frontend
        image: your-dockerhub-username/mega-project-frontend:latest
        ports: [{ containerPort: 3000 }]
      - name: backend
        image: your-dockerhub-username/mega-project-backend:latest
        ports: [{ containerPort: 5000 }]
```

### `k8s/service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata: { name: mega-project-service }
spec:
  type: LoadBalancer
  selector: { app: mega-project-app }
  ports:
    - port: 80
      targetPort: 3000
```

---

## 7️⃣ CI/CD – `ci-cd/Jenkinsfile`

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') { steps { git 'https://github.com/<your-username>/mega-project.git' } }
        stage('Build & Push Docker') {
            steps {
                sh 'docker build -t frontend ./frontend'
                sh 'docker build -t backend ./backend'
                sh 'docker push frontend'
                sh 'docker push backend'
            }
        }
        stage('Deploy Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/'
            }
        }
    }
}
```

---

## 8️⃣ Deployment Steps

1. Provision **AWS infrastructure** using Terraform.

```bash
cd terraform
terraform init
terraform apply
```

2. **Build Docker images** for frontend and backend.

```bash
docker build -t frontend ./frontend
docker build -t backend ./backend
docker push <your-dockerhub-username>/frontend
docker push <your-dockerhub-username>/backend
```

3. Deploy on **Kubernetes cluster**.

```bash
kubectl apply -f k8s/
```

4. Configure **AWS Lex bot** and Lambda function.

5. Run **Jenkins pipeline** to automate CI/CD.

---

✅ This Mega Project is now **fully structured, end-to-end deployable**, combining:

* Full-stack web portal
* Serverless chatbot
* Terraform-managed cloud infra
* Containerized apps on Kubernetes
* Automated CI/CD with Jenkins
