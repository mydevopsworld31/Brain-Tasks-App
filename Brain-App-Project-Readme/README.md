Brain Tasks App Deployment (DevOps Project)
Project Overview:
-> This project demonstrates the deployment of a React-based application using modern DevOps practices. The application is containerized using Docker and deployed on Kubernetes using Amazon EKS. The project also includes monitoring and version control integration.

TECH STACK Uses:
- AWS EC2 - Amazon EKS (Kubernetes) - Docker & DockerHub - Kubernetes (kubectl) - Git & GitHub - Amazon CloudWatch

Application Details:
- Application: Brain Tasks App - Source Repository: https://github.com/Vennilavanguvi/Brain-Tasks-App.git - Port: 3000 (local), 80 (Docker/Kubernetes)

Implementation STEPs:
--> 1️⃣ Application Setup - Cloned the GitHub repository - Installed dependencies using npm - Built production-ready files ```bash git clone https://github.com/Vennilavanguvi/Brain-Tasks-App.git cd Brain-Tasks-App npm install npm run build
--> 2️⃣ Dockerization - Created a Dockerfile to containerize the application - Used Nginx as a web server - Built and tagged Docker image ```bash docker build -t brain-app . docker tag brain-app mydockerimage31/brain-app:v1
--> 3️⃣ DockerHub Integration - Logged into DockerHub - Pushed Docker image ```bash docker push mydockerimage31/brain-app:v1
--> 4️⃣ Kubernetes Deployment (Amazon EKS) - Created EKS cluster - Configured kubectl - Created deployment and service YAML files #### Deployment YAML - Deploys containerized app - Uses replica set for scalability #### Service YAML - Type: LoadBalancer - Exposes application to internet ```bash kubectl apply -f deployment.yaml kubectl apply -f service.yaml
--> 5️⃣ Scaling Issue & Solution - Faced pod scheduling issue due to low resources (t3.micro) - Error: “Too many pods” - Solution: Scaled node group from 2 to 3 nodes
--> 6️⃣ Application Access Application exposed using AWS LoadBalancer: 🌐 **Live URL:** http://a63aee061b39847d38bd25db59f6f39a-1219897690.ap-south-1.elb.amazonaws.com
--> 7️⃣ Monitoring (CloudWatch) - Logs monitored using Amazon CloudWatch - Verified pod logs using kubectl ```bash kubectl logs <pod-name>

KEY LEARNING:
-> - Docker containerization - Kubernetes deployment & services - Amazon EKS cluster management - Debugging real-world issues (pod scheduling) - Node scaling in Kubernetes - Monitoring using CloudWatch

Conclusion Successfully deployed a scalable React application using DevOps tools and AWS services. This project demonstrates end-to-end deployment from development to production using containerization and orchestration.


