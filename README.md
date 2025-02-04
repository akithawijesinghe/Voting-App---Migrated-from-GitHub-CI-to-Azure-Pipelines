# **Example Voting App - Migrated from GitHub CI to Azure Pipelines**

## **Overview**
This project demonstrates the migration of a GitHub CI pipeline to Azure Pipelines for a **multi-architecture microservice application**. The application consists of multiple services running across different containers, utilizing **Python, Node.js, .NET, Redis, and PostgreSQL**.

---

## **Application Description**
### **Example Voting App**
A simple distributed application running across multiple Docker containers.

### **Technologies Used**
- **Frontend**: Python (Flask)
- **Messaging Queue**: Redis
- **Worker Service**: .NET
- **Database**: PostgreSQL
- **Results Service**: Node.js (Express.js)
- **Containerization**: Docker, Kubernetes, Docker Swarm
- **CI/CD**: Azure DevOps Pipelines

### **Running the Application Locally**
Ensure you have Docker installed and running.

To build and run the application:
```sh
docker compose up
```
- **Vote App**: [http://localhost:8080](http://localhost:8080)
- **Results App**: [http://localhost:8081](http://localhost:8081)

To deploy on **Docker Swarm**:
```sh
docker swarm init
docker stack deploy --compose-file docker-stack.yml vote
```

To deploy on **Kubernetes**:
```sh
kubectl create -f k8s-specifications/
```
- **Vote Web App**: Available on port `31000`
- **Results Web App**: Available on port `31001`

To remove Kubernetes resources:
```sh
kubectl delete -f k8s-specifications/
```

---

## **Azure DevOps Migration Steps**

### **1. Import GitHub Repository to Azure DevOps**
1. Create a new **Azure DevOps Project**.
2. Copy the **GitHub HTTPS URL** of the repository.
3. Go to **Azure Repos > Import Repository**.
4. Select **Git** as the repository type and paste the copied URL.
5. Go to **Branches** and set the **main** branch as the default.

### **2. Set Up Azure Container Registry (ACR)**
1. Create a new **Resource Group** in Azure.
2. Create an **Azure Container Registry (ACR)** for storing container images.

### **3. Set Up a Self-Hosted Agent in Azure DevOps**
1. Go to **Azure DevOps > Project Settings > Agent Pools**.
2. Click **New Self-Hosted Agent Pool**.
3. Create an **Azure Virtual Machine (VM)** and connect via SSH:
   ```sh
   ssh azureuser@your-vm-ip
   ```
4. Install and configure the agent:
   ```sh
   mkdir myagent && cd myagent
   wget https://vstsagentpackage.azureedge.net/agent/4.248.0/vsts-agent-linux-x64-4.248.0.tar.gz
   sudo apt update
   tar zxvf vsts-agent-linux-x64-4.248.0.tar.gz
   ./config.sh
   ```
5. Provide the Azure DevOps server URL:
   ```sh
   https://dev.azure.com/{your-organization}
   ```
6. Generate and enter a **Personal Access Token (PAT)**.
7. Specify the **Agent Pool Name** created in step 1.
8. Install Docker on the VM:
   ```sh
   sudo apt install docker.io
   sudo usermod -aG docker azureuser
   sudo systemctl restart docker
   ```
9. Start the agent:
   ```sh
   ./run
   ```

### **4. Set Up Azure DevOps Pipelines**
1. Go to **Pipelines** in Azure DevOps.
2. Click **Create Pipeline**.
3. Select **Azure Repos Git** and choose the **Voting App** repository.
4. Choose **Docker Build and Push Image** as the pipeline template.
5. Sign in to Azure and select:
   - **Azure Subscription** (e.g., Free Trial / Azure for Students)
   - **Container Registry**
   - **Image Name** and **Dockerfile**
6. Edit the `azure-pipelines.yml` file and rename the pipeline to `Result-Service`.
7. Click **Save and Run**.

### **5. Create Pipelines for Other Services**
Repeat the above steps to create pipelines for:
- `Vote-Service`
- `Worker-Service`

---

## **Pipeline Configuration (`azure-pipelines.yml`)**
Each service has a dedicated Azure DevOps pipeline configured to build, push, and deploy the respective containerized application.

### **Example: `Result-Service` Pipeline**
```yaml
trigger:
  paths:
    include:
      - result/*

pool:
  name: 'azureagent'

stages:
- stage: Build
  displayName: Build
  jobs:
  - job: Build
    displayName: Build
    steps:
    - task: Docker@2
      displayName: Build An image 
      inputs:
        containerRegistry: '$(dockerRegistryServiceConnection)'
        repository: '$(imageRepository)'
        command: 'build'
        Dockerfile: 'result/Dockerfile'
        tags: '$(tag)'
- stage: Push
  displayName: Push
  jobs:
  - job: Push
    displayName: Push
    steps:
    - task: Docker@2
      displayName: Push an image
      inputs:
        containerRegistry: '$(dockerRegistryServiceConnection)'
        repository: '$(imageRepository)'
        command: 'build'
        Dockerfile: 'result/Dockerfile'
        tags: '$(tag)'

```

---

## **Conclusion**
This project successfully migrates the **GitHub CI/CD pipeline** to **Azure DevOps Pipelines** for a **multi-service microservices application**. The new setup allows efficient **containerized application deployment** using **Azure Container Registry (ACR)** and **Azure Virtual Machines (VMs) as self-hosted agents**.

### **Key Benefits of Azure Pipelines Migration**
✅ Improved CI/CD pipeline management with **Azure DevOps**  
✅ Automated **Docker image builds and deployments**  
✅ Enhanced security with **self-hosted agents**  
✅ Scalable deployment using **Azure Kubernetes Service (AKS)** and **Azure Container Registry (ACR)**

---




