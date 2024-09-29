# python-flask-app-docker-k8s

Deploying a Python application on Minikube involves several steps, such as containerizing the application, setting up Kubernetes resources (like pods, services, and deployments), and ensuring scalability with replica management and auto-scaling. Here's a guide on how you can achieve this:

### Prerequisites
1. **Minikube** installed and running.
2. **Docker** installed (for building images).
3. **kubectl** installed to interact with Minikube.
4. A **Python app** (a Flask or Django app can be used as an example).

### Steps to Deploy and Scale a Python App in Minikube

### clone repo
```
git clone https://github.com/atulkamble/python-flask-app-docker-k8s.git
cd python-flask-app-docker-k8s
```

#### 1. **Create a Python App**
For this example, we will use a simple Flask app:

```python
# app.py
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello from Flask!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

#### 2. **Dockerize the Python App**

Create a `Dockerfile` for the Python app:

```Dockerfile
# Dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

In this example, `requirements.txt` should contain the dependencies:

```
Flask==2.0.1
```

#### 3. **Build the Docker Image**

If Minikube is running, set Docker to use Minikube’s Docker daemon:

```bash
eval $(minikube docker-env)
```

Now, build the Docker image:

```bash
docker build -t python-flask-app .
```

#### 4. **Create Kubernetes Deployment and Service**

Create a `deployment.yaml` file for Kubernetes:

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-flask-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: python-flask-app
  template:
    metadata:
      labels:
        app: python-flask-app
    spec:
      containers:
      - name: python-flask-container
        image: python-flask-app
        ports:
        - containerPort: 5000
---
apiVersion: v1
kind: Service
metadata:
  name: python-flask-service
spec:
  selector:
    app: python-flask-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
  type: NodePort
```

#### 5. **Apply the Kubernetes Configuration**

Run the following command to create the deployment and service in Minikube:

```bash
kubectl apply -f deployment.yaml
```

Check the status of the deployment and pods:

```bash
kubectl get pods
```

#### 6. **Access the App in Minikube**

Once the service is up, you can access the app by running:

```bash
minikube service python-flask-service
```

This will open the app in your default browser.

#### 7. **Scaling the App**

You can scale the application using `kubectl` to increase or decrease the number of replicas:

```bash
kubectl scale deployment python-flask-deployment --replicas=3
```

To check if the scaling worked, you can run:

```bash
kubectl get pods
```

#### 8. **Autoscaling the App**

To enable autoscaling based on CPU usage, you can use the Kubernetes Horizontal Pod Autoscaler (HPA). Create the autoscaler using the following command:

```bash
kubectl autoscale deployment python-flask-deployment --cpu-percent=50 --min=1 --max=5
```

This command creates an HPA that will scale the number of pods between 1 and 5 based on the average CPU usage across all pods in the deployment.

To monitor the HPA status:

```bash
kubectl get hpa
```

#### 9. **Monitor Application Logs**

You can view the logs of the application using:

```bash
kubectl logs <pod-name>
```

Or monitor logs for all pods in the deployment:

```bash
kubectl logs -l app=python-flask-app --all-containers=true
```

### Cleanup

Once you're done, you can delete the deployment and service:

```bash
kubectl delete -f deployment.yaml
```

Or delete everything related to the app:

```bash
kubectl delete deployment python-flask-deployment
kubectl delete service python-flask-service
```

#### Summary
This process shows how to deploy a Python application in Minikube using Docker and Kubernetes. It also demonstrates how to scale the application manually and automatically with the Horizontal Pod Autoscaler.
