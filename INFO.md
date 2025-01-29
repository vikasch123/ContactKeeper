# **ContactKeeper DevOps Project Documentation**

## **1. Project Overview**
### **1.1 Introduction**
ContactKeeper is a web application that allows users to manage and store their contacts securely. This project aims to deploy ContactKeeper on **AWS EC2** using **Docker, Nginx, and CI/CD pipelines** while implementing DevOps best practices.

### **1.2 DevOps Objectives**
- Deploy ContactKeeper as a web service on AWS EC2.
- Automate infrastructure setup using **Terraform**.
- Containerize the application using **Docker**.
- Implement **CI/CD** with GitHub Actions.
- Monitor application performance using **Prometheus and Grafana**.
- Ensure security best practices with **AWS IAM and security groups**.

---

## **2. Technology Stack**
- **Cloud Provider:** AWS (EC2, S3, RDS, IAM)
- **Infrastructure as Code:** Terraform
- **Configuration Management:** Ansible
- **Containerization:** Docker, Docker Compose
- **Web Server:** Nginx
- **Database:** PostgreSQL / MongoDB (if required by the app)
- **CI/CD:** GitHub Actions
- **Monitoring:** Prometheus + Grafana
- **Logging:** ELK Stack (Elasticsearch, Logstash, Kibana)
- **Security:** AWS IAM, Security Groups, SSL (Let's Encrypt)

---

## **3. Infrastructure Setup with Terraform**
### **3.1 Prerequisites**
- Install Terraform: `brew install terraform` (Mac) or `sudo apt install terraform` (Linux)
- AWS CLI configured with credentials

### **3.2 Terraform Script to Provision EC2 Instance**
```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "contactkeeper" {
  ami           = "ami-0c55b159cbfafe1f0"  # Ubuntu 22.04 AMI ID
  instance_type = "t2.micro"
  key_name      = "your-keypair"
  security_groups = ["contactkeeper_sg"]
  user_data = file("setup.sh")
}
```

### **3.3 Security Group Setup**
```hcl
resource "aws_security_group" "contactkeeper_sg" {
  name        = "contactkeeper_sg"
  description = "Allow inbound traffic"
  
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

### **3.4 Deploy Terraform Infrastructure**
```sh
tf init
tf apply -auto-approve
```

---

## **4. Application Deployment with Docker**
### **4.1 Dockerfile for ContactKeeper**
```dockerfile
FROM node:18
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
CMD ["npm", "start"]
EXPOSE 3000
```

### **4.2 Docker Compose for Multi-Container Setup**
```yaml
version: "3.8"
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    depends_on:
      - db

  db:
    image: postgres
    restart: always
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: contactkeeper
```

### **4.3 Deploying the Docker Containers**
```sh
docker-compose up -d
```

---

## **5. CI/CD Pipeline with GitHub Actions**
### **5.1 GitHub Actions Workflow (.github/workflows/deploy.yml)**
```yaml
name: Deploy to EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3
      
      - name: SSH into EC2 & Deploy
        uses: appleboy/ssh-action@v0.1.10
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ubuntu
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            cd /home/ubuntu/ContactKeeper
            git pull origin main
            docker-compose down
            docker-compose up -d --build
```

---

## **6. Monitoring & Logging**
### **6.1 Install Prometheus & Grafana**
```sh
docker run -d -p 9090:9090 --name prometheus prom/prometheus
```
```sh
docker run -d -p 3000:3000 --name grafana grafana/grafana
```
### **6.2 Configure Prometheus for Application Metrics**
```yaml
scrape_configs:
  - job_name: 'contactkeeper'
    static_configs:
      - targets: ['localhost:3000']
```
### **6.3 Log Management with ELK Stack**
- Install **Elasticsearch, Logstash, and Kibana**
- Use **Filebeat** to forward logs

---

## **7. Security & Best Practices**
- **IAM Roles & Policies:** Restrict EC2 permissions.
- **Security Groups:** Open only necessary ports.
- **SSL Setup:** Use **Let's Encrypt** for HTTPS.
- **Regular Backups:** Configure AWS **S3 snapshots** for database backups.

---

## **8. Conclusion**
This DevOps implementation ensures **scalability, automation, and reliability** of ContactKeeper. Further improvements can include **Kubernetes deployment, auto-scaling, and multi-region failover**.

