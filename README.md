# **Cloud Native Resource Monitoring Python App on K8s!**

## Things you will Learn 🧩

1. Build a real-time Monitoring Application using Python (Flask, psutil) with live updating graphs.
2. Run a Python App locally.
3. Learn Docker and How to containerize a Python application:

   1. Creating Dockerfile
   2. Building DockerImage
   3. Running Docker Container
   4. Docker Commands
4. Create an ECR repository using Python Boto3 and push Docker Image to ECR.
5. Learn Kubernetes and Create EKS cluster and Nodegroups.
6. Create Kubernetes Deployments and Services using Python Kubernetes SDK.
7. Live monitoring with Plotly.js gauge charts and alert system for high CPU/memory usage.

## **Prerequisites**

(Things to have before starting the project)

* [x] AWS Account.
* [x] Programmatic access and AWS CLI configured.
* [x] Python 3 Installed (preferably Python 3.11).
* [x] Docker and Kubectl installed.
* [x] Code editor (VS Code).

# ✨ Let’s Start the Project ✨

## **Part 1: Deploying the Flask application locally**

### **Step 1: Clone the code**

```bash
git clone <repository_url>
```

### **Step 2: Install dependencies**

The application uses the **`psutil`**, **`Flask`**, **`Plotly`**, and **`boto3`** libraries. Install them using pip:

```bash
pip3 install -r requirements.txt
```

### **Step 3: Run the application**

```bash
python3 app.py
```

Visit: [http://localhost:5000/](http://localhost:5000/) to view the real-time monitoring dashboard.

## **Part 2: Dockerizing the Flask application**

### **Step 1: Dockerfile**

```Dockerfile
FROM python:3.9-slim-buster
WORKDIR /app
COPY requirements.txt .
RUN pip3 install --no-cache-dir -r requirements.txt
COPY . .
ENV FLASK_RUN_HOST=0.0.0.0
EXPOSE 5000
CMD ["flask", "run"]
```

### **Step 2: Build Docker image**

```bash
docker build -t cloud-monitoring-app .
```

### **Step 3: Run the container**

```bash
docker run -p 5000:5000 cloud-monitoring-app
```

Visit [http://localhost:5000/](http://localhost:5000/)

## **Part 3: Push Docker image to AWS ECR**

### **Step 1: Create ECR repository using boto3**

```python
import boto3

repository_name = 'my-cloud-native-repo'
ecr_client = boto3.client('ecr')
response = ecr_client.create_repository(repositoryName=repository_name)
print(response['repository']['repositoryUri'])
```

### **Step 2: Authenticate & push image**

Follow AWS ECR instructions to login, tag, and push:

```bash
eval $(aws ecr get-login-password --region <region>) | docker login --username AWS --password-stdin <account_id>.dkr.ecr.<region>.amazonaws.com

docker tag cloud-monitoring-app:latest <account_id>.dkr.ecr.<region>.amazonaws.com/my-cloud-native-repo:latest

docker push <account_id>.dkr.ecr.<region>.amazonaws.com/my-cloud-native-repo:latest
```

## **Part 4: Deploy to Kubernetes (EKS) using Python**

### **Step 1: Create EKS cluster and nodegroup**

Use AWS CLI, console, or eksctl to create an EKS cluster and attach worker nodes.

### **Step 2: Kubernetes Deployment and Service using Python**

```python
from kubernetes import client, config

config.load_kube_config()
api_client = client.ApiClient()

# Define deployment
deployment = client.V1Deployment(
    metadata=client.V1ObjectMeta(name="my-flask-app"),
    spec=client.V1DeploymentSpec(
        replicas=1,
        selector=client.V1LabelSelector(match_labels={"app": "my-flask-app"}),
        template=client.V1PodTemplateSpec(
            metadata=client.V1ObjectMeta(labels={"app": "my-flask-app"}),
            spec=client.V1PodSpec(containers=[
                client.V1Container(
                    name="flask-container",
                    image="<your_ecr_image_uri>",
                    ports=[client.V1ContainerPort(container_port=5000)]
                )
            ])
        )
    )
)

apps_v1 = client.AppsV1Api(api_client)
apps_v1.create_namespaced_deployment(namespace="default", body=deployment)

# Define service
service = client.V1Service(
    metadata=client.V1ObjectMeta(name="flask-service"),
    spec=client.V1ServiceSpec(
        selector={"app": "my-flask-app"},
        ports=[client.V1ServicePort(port=5000)]
    )
)

core_v1 = client.CoreV1Api(api_client)
core_v1.create_namespaced_service(namespace="default", body=service)
```

> 🔗 Make sure to replace `<your_ecr_image_uri>` with your actual image URI from ECR.

### **Step 3: Check deployment and port forward**

```bash
kubectl get deployments -n default
kubectl get services -n default
kubectl get pods -n default
kubectl port-forward service/flask-service 5000:5000
```

Access your live app at [http://localhost:5000/](http://localhost:5000/)
