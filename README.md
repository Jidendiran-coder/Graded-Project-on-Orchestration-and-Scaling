# MERN Microservices CI/CD Deployment with Jenkins, Docker, ECR, and EC2

This project demonstrates a complete CI/CD setup for deploying a **MERN-based microservices architecture** using:

- **Jenkins Pipelines**
- **Docker**
- **AWS ECR (Elastic Container Registry)**
- **EC2 instances (SSH deployment)**
- **Nginx reverse proxy**

## 🚀 What’s Included

- **Microservices Architecture**:
  - `helloService`: API for greeting functionality
  - `profileService`: API for user profile features
  - `frontend`: React app consuming both APIs

- **Dockerized Services**:
  - Each service has its own Dockerfile
  - Independent container builds and deployments

- **CI/CD Pipelines**:
  - Each service has its own Jenkinsfile
  - Pipelines build, tag, push to AWS ECR
  - EC2 hosts pull latest image and run containers

- **Nginx Reverse Proxy**:
  - Routes:
    - `/hello → :3001`
    - `/profile → :3002`
    - `/ → frontend (e.g., :3000)`

---

## ✨ Project Structure

```
mern-orchestration/
├── backend/
│   ├── helloService/
│   │   └── Dockerfile, index.js, Jenkinsfile-hello. etc.
│   └── profileService/
│       └── Dockerfile, index.js, Jenkinsfile-profile etc.
├── frontend/
│   └── Dockerfile, src/, Jenkinsfile-frontend, etc.
└── README.md
```

## 🔄 Jenkins CI/CD Pipeline Overview

Each Jenkinsfile performs the following stages:

1. **Checkout Git Repo**
2. **Build Docker Image**
3. **Login to AWS ECR**
4. **Push Docker Image**
5. **Tag as `latest` for consistent deployment**
6. **Clean Docker Images**
7. **SSH into EC2 & Deploy Container**
   - Stops old container (if any)
   - Pulls new image from ECR
   - Runs the updated container

---

## 🔐 Jenkins Credentials Used

| ID                      | Description                             |
|-------------------------|-----------------------------------------|
| `jide-access-key-id`   | AWS Access Key ID                       |
| `jide-secret-access-key` | AWS Secret Access Key                 |
| `jide-ec2`             | EC2 private SSH key                     |
| `ec2-backend-user`      | EC2 username (`ec2-user` or custom)     |
| `ec2-backend-host`      | EC2 public IP                           |
| `mern-database`         | MongoDB connection string               |
| `jide-github-access`   | GitHub token/credentials                |
| `mern-database`         | Database Url                            |
| `ec2-frontend-user`     | EC2 username (`ec2-user` or custom)     |
| `ec2-frontend-host`     | EC2 public IP                           |


---

## 🧪 How to Access the Services

After deployment, services are available at:

| Service          | Port   | Endpoint                      | Nginx Route             |
|------------------|--------|-------------------------------|-------------------------|
| Hello Service    | 3001   | http://<EC2-IP>:3001          | http://<domain>/hello   |
| Profile Service  | 3002   | http://<EC2-IP>:3002          | http://<domain>/profile |
| Frontend         | 3000   | http://<EC2-IP>:3000          | http://<domain>/        |

Make sure Nginx is configured to forward requests to the correct internal ports.

---

## 🛠 Sample Nginx Config

```nginx
server {
    listen 80;
    server_name api-mern.playbook.org.in;

    location /hello {
        proxy_pass http://localhost:3001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location /profile {
        proxy_pass http://localhost:3002;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

## 🚀 How to Trigger Each Jenkins Job

- **helloService**: Jenkinsfile-hello (backend/helloService)
- **profileService**: Jenkinsfile-profile (backend/profileService)
- **frontend**: Jenkinsfile-frontend (frontend)

## 🧰 Environment Variables Passed to Containers

**helloService/profileService**
- `PORT` (e.g., 3001, 3002)
- `MONGO_URL` (MongoDB connection string)

**frontend**
- `HELLO_SERVICE`: Public URL (e.g., http://<domain>/hello/)
- `USER_SERVICE`: Public URL (e.g., http://<domain>/profile/)


## 📄 Notes

- Containers run with `--restart unless-stopped` to survive EC2 reboots.
- Jenkins pipelines use latest tagging by default via `${IMAGE_TAG}`.
- Each EC2 instance must have Docker installed and the appropriate user added to the docker group.
- Create a Launch Template that includes:
  - An AMI with Docker installed
  - A user data script to pull the latest image from ECR and run the container
- Define an Auto Scaling Group (ASG) using the above Launch Template
- Add a Target Group and attach it to an Application Load Balancer (ALB)
- Configure scaling policies (CPU-based, request-count, schedule-based)
- This ensures new EC2 instances launched by the ASG are auto-configured with your backend containers.


---

## 📄 Outputs

### Backend Outputs

Jenkins Hello API

<img width="1467" height="760" alt="hello_jenkins" src="https://github.com/user-attachments/assets/068c216a-5e72-4344-a908-776f6ebdc0fc" />

ECR hello API

<img width="1470" height="730" alt="ecr_hello" src="https://github.com/user-attachments/assets/8cb0cb30-a3ae-4e91-8e25-fdd50f6e6917" />

Jenkins Profile API

<img width="1386" height="762" alt="profile_jenkins" src="https://github.com/user-attachments/assets/f5a638c2-b4fc-4560-8761-3ac25fe9ba1e" />

ECR Profile API

<img width="1470" height="655" alt="ecr_profile" src="https://github.com/user-attachments/assets/57d04cfd-ae88-4a13-afe6-d8aa8e0282ac" />

Docker Ouput - Backend

<img width="1470" height="237" alt="dockers_backend" src="https://github.com/user-attachments/assets/d344c530-2a32-4324-ac04-7b42e85b5745" />

Note: The backend can be run in 2 servers if needed for hello API and Profile APi, to demo purpose i have used single EC2 to both the API

Hello API Output with IP

<img width="739" height="161" alt="api_hello_with_ip" src="https://github.com/user-attachments/assets/7ed4232d-48c2-41b9-b727-4a032bf20bce" />

Hello API Output with Website Address

<img width="800" height="220" alt="api_hello" src="https://github.com/user-attachments/assets/0c730dd0-5a7d-4c7a-b209-408affc0161e" />

USER API Output with IP

<img width="656" height="316" alt="api_user_with_ip" src="https://github.com/user-attachments/assets/904f18c4-0cdd-4efe-855a-36921044040e" />

User API Output with Website Address

<img width="849" height="340" alt="api_user" src="https://github.com/user-attachments/assets/5ab054a1-5715-48c5-a65f-861f35bfbf93" />

### Frontend Outputs

Jenkins Front End

<img width="1389" height="723" alt="front_jenkins" src="https://github.com/user-attachments/assets/d213548c-8fbb-4732-bf6c-11c7ed503c4d" />

ECR frontend

<img width="1470" height="441" alt="ecr_frontend" src="https://github.com/user-attachments/assets/c9d4120d-6d00-4d9a-b737-87b67785fd8c" />

Website Output frontend Via IP

<img width="1006" height="446" alt="front_ip" src="https://github.com/user-attachments/assets/059d1d2d-3909-472c-a3ea-719662ecaa29" />

Website Output frontend Via Website

<img width="930" height="406" alt="front_domain" src="https://github.com/user-attachments/assets/0ff06b4f-9fcf-4cdc-ae85-57da4b63a9cb" />

AWS Instance List

<img width="1470" height="303" alt="aws_output" src="https://github.com/user-attachments/assets/9483e193-dbf7-4c33-a2aa-1e6348d58e6c" />



## ☸️ Jenkins + EKS Deployment (Optional Variant)

You can also deploy the MERN microservices stack to **Amazon EKS (Elastic Kubernetes Service)** with Jenkins:

### ✅ Steps:

#### 1. Create EKS Cluster:
- Use `eksctl`, Terraform, or AWS Console
- Make sure `kubectl` and `aws-iam-authenticator` are installed on the Jenkins agent

#### 2. Configure Kubeconfig in Jenkins Agent:

```
aws eks update-kubeconfig --region ap-south-1 --name jide-mern-cluster
```

#### 3. Build & Push Docker Images to ECR:

Same steps as your EC2-based pipelines

#### 4. Helm Deployment:

- Package your services into Helm charts (e.g., frontend, hello, profile)

In Jenkins, add a Helm deploy will have something like below:

```
helm upgrade --install mern-website . \
-f values.yaml \
--set-string image.repository="jidendirpy/mernwebsite" \
--set-string image.tag="v1" \
--set-string hello_service="<url>" \
--set-string profile_service="<url>" \
--set service.port=80 \
--set service.targetPort=3001 \
--set-string service.type="NodePort" \ 
--set replicaCount=1
```

The same helm will customised for the hello and profile services

#### 5. DNS and Ingress:

- Use ALB Ingress Controller or NGINX Ingress Controller
- Map domain names to backend/frontend services via Ingress resources

#### 6. Scaling and Monitoring:

- Use Horizontal Pod Autoscalers (HPA) for autoscaling based on CPU/requests











