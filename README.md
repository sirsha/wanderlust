# Wanderlust - Your Ultimate Travel Blog 🌍✈️

WanderLust is a simple MERN travel blog website ✈ This project is aimed to help people to contribute in open source, upskill in react and also master git.

![Preview Image](https://github.com/krishnaacharyaa/wanderlust/assets/116620586/17ba9da6-225f-481d-87c0-5d5a010a9538)

# Wanderlust Mega Project End to End Implementation

### In this demo, we will see how to deploy an end to end three tier MERN stack application on EKS cluster, utilizing Jenkins for continuous integration, ArgoCD for continuous deployment, and Prometheus and Grafana for monitoring.
#
### <mark>Project Deployment Flow:</mark>
<img src="https://github.com/sirsha/wanderlust/blob/devops/images/Untitled%20Diagram.drawio.png" />

#

## Tech stack used in this project:
- GitHub (Code)
- Docker (Containerization)
- Jenkins (CI)
- OWASP (Dependency check)
- SonarQube (Quality)
- Trivy (Filesystem Scan)
- ArgoCD (CD)
- Redis (Caching)
- AWS EKS (Kubernetes)
- Helm (Monitoring using grafana and prometheus)

### How pipeline will look after deployment:
- <b>CI pipeline to build and push</b>

![image](https://github.com/sirsha/wanderlust/blob/devops/images/Screenshot%202025-07-25%20205322.png)

- <b>CD pipeline to update application version</b>

![image](https://github.com/sirsha/wanderlust/blob/devops/images/diagram-export-7-25-2025-8_58_10-PM.png)
- <b>ArgoCD application for deployment on EKS</b>

![image](https://github.com/sirsha/wanderlust/blob/devops/images/Screenshot%202025-07-25%20210038.png)

#
> [!Important]
> Below table helps you to navigate to the particular tool installation section fast.

| Tech stack    | Installation |
| -------- | ------- |
| Jenkins Master | <a href="#Jenkins">Install and configure Jenkins</a>     |
| eksctl | <a href="#EKS">Install eksctl</a>     |
| Argocd | <a href="#Argo">Install and configure ArgoCD</a>     |
| Jenkins-Worker Setup | <a href="#Jenkins-worker">Install and configure Jenkins Worker Node</a>     |
| OWASP setup | <a href="#Owasp">Install and configure OWASP</a>     |
| SonarQube | <a href="#Sonar">Install and configure SonarQube</a>     |
| Email Notification Setup | <a href="#Mail">Email notification setup</a>     |
| Monitoring | <a href="#Monitor">Prometheus and grafana setup using helm charts</a>
| Clean Up | <a href="#Clean">Clean up</a>     |
#

### Pre-requisites to implement this project:
#
- Root user access
```bash
sudo su
```
> [!Note]
> This project will be implemented on N.virginia region (us-east-1).

- <b>Create 1 Master machine on AWS with 2CPU, 8GB of RAM (t2.large) and 29 GB of storage and install Docker on it.</b>

- <b>Open the below ports in security group of master machine and also attach same security group to Jenkins node (Here I have used same node as master for Jenkins installation)</b>
![image](https://github.com/sirsha/wanderlust/blob/devops/images/Screenshot%202025-07-25%20210844.png)


- Create an IAM role with <mark>administrator access</mark> attach it to the jenkins node <mark>Select Jenkins node EC2 instance --> Actions --> Security --> Modify IAM role</mark>
  ![image](https://github.com/user-attachments/assets/1a9060db-db11-40b7-86f0-47a65e8ed68b)

  #
- <b id="Argo">Install and Configure ArgoCD (Master Machine)</b>
  - <b>Create argocd namespace</b>
  ```bash
  kubectl create namespace argocd
  ```
  - <b>Apply argocd manifest</b>
  ```bash
  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
  ```
  - <b>Make sure all pods are running in argocd namespace</b>
  ```bash
  watch kubectl get pods -n argocd
  ```
  - <b>Install argocd CLI</b>
  ```bash
  curl --silent --location -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.4.7/argocd-linux-amd64
  ```
  - <b>Provide executable permission</b>
  ```bash
  chmod +x /usr/local/bin/argocd
  ```
  - <b>Check argocd services</b>
  ```bash
  kubectl get svc -n argocd
  ```
  - <b>Change argocd server's service from ClusterIP to NodePort</b>
  ```bash
  kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
  ```
  - <b>Confirm service is patched or not</b>
  ```bash
  kubectl get svc -n argocd
  ```
  - <b> Check the port where ArgoCD server is running and expose it on security groups of a worker node</b>
  ![image](https://github.com/sirsha/wanderlust/blob/devops/images/diagram-export-7-25-2025-9_12_56-PM.png)
  - <b>Access it on browser, click on advance and proceed with</b>
  ```bash
  <public-ip-worker>:<port>
  ```
  ![image](https://github.com/sirsha/wanderlust/blob/devops/images/diagram-export-7-25-2025-9_17_46-PM.png)
  - <b>Fetch the initial password of argocd server</b>
  ```bash
  kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
  ```
  - <b>Username: admin</b>
  - <b> Now, go to <mark>User Info</mark> and update your argocd password
 

#
## Steps to implement the project:
- <b>Go to Jenkins Master and click on <mark> Manage Jenkins --> Plugins --> Available plugins</mark> install the below plugins:</b>
  - OWASP
  - SonarQube Scanner
  - Docker
  - Pipeline: Stage View
#
- <b id="Owasp">Configure OWASP, move to <mark>Manage Jenkins --> Plugins --> Available plugins</mark> (Jenkins node)</b>
![image](https://github.com/user-attachments/assets/da6a26d3-f742-4ea8-86b7-107b1650a7c2)
#
- <b id="Sonar">After OWASP plugin is installed, Now move to <mark>Manage jenkins --> Tools</mark> (Jenkins node)</b>
![image](https://github.com/user-attachments/assets/3b8c3f20-202e-4864-b3b6-b48d7a604ee8)

#
- <b id="Owasp">OWASP dependency Check Results</b>
![image](https://github.com/sirsha/wanderlust/blob/devops/images/Screenshot%202025-07-25%20212609.png)

#
- <b id="Sonar">Sonar scanner Results</b>
![image](https://github.com/sirsha/wanderlust/blob/devops/images/Screenshot%202025-07-25%20213008.png)
#
- <b>Create a <mark>Wanderlust-CI/CD</mark> pipeline</b>
![image](https://github.com/sirsha/wanderlust/blob/devops/images/4.png)

- <b>Provide permission to docker socket so that docker build and push command do not fail (Jenkins Worker)</b>
```bash
chmod 777 /var/run/docker.sock
```
![image](https://github.com/user-attachments/assets/e231c62a-7adb-4335-b67e-480758713dbf)
#
- <b> Go to Master Machine and add our own eks cluster to argocd for application deployment using cli</b>
  - <b>Login to argoCD from CLI</b>
  ```bash
   argocd login 3.80.205.64:31823 --username admin
  ```
> [!Tip]
> 3.80.205.64:31823 --> This should be your argocd url

  ![image](https://github.com/sirsha/wanderlust/blob/devops/images/diagram-export-7-25-2025-9_38_03-PM.png)
  - <b>Check how many clusters are available in argocd </b>
  ```bash
  argocd cluster list
  ```
  ![image](https://github.com/sirsha/wanderlust/blob/devops/images/diagram-export-7-25-2025-9_42_14-PM.png)
  - <b>Get your cluster name</b>
  ```bash
  kubectl config get-contexts
  ```
  ![image](https://github.com/sirsha/wanderlust/blob/devops/images/diagram-export-7-25-2025-9_42_21-PM.png)
  - <b>Add your cluster to argocd</b>
  ```bash
  argocd cluster add bootcamp-sirsha-user@wanderlust.us-east-1.eksctl.io --name wanderlust-eks-cluster
  ```
  > [!Tip]
  > bootcamp-sirsha-user@wanderlust.us-east-1.eksctl.io --> This should be your EKS Cluster Name.

  ![image](https://github.com/sirsha/wanderlust/blob/devops/images/diagram-export-7-25-2025-9_42_28-PM.png)
  - <b> Once your cluster is added to argocd, go to argocd console <mark>Settings --> Clusters</mark> and verify it</b>
  ![image](https://github.com/sirsha/wanderlust/blob/devops/images/Screenshot%202025-07-25%20214855.png)
#
- <b>Open port 31000 and 31100 on worker node and Access it on browser</b>
```bash
<worker-public-ip>:31000
```
![image](https://github.com/sirsha/wanderlust/blob/devops/images/Screenshot%202025-07-25%20215527.png)
![image](https://github.com/sirsha/wanderlust/blob/devops/images/Screenshot%202025-07-25%20215923.png)
![image](https://github.com/sirsha/wanderlust/blob/devops/images/Screenshot%202025-07-25%20220203.png)
- <b>Email Notification</b>
![image](https://github.com/sirsha/wanderlust/blob/devops/images/Screenshot%202025-07-25%20220345.png)

#
## How to monitor EKS cluster, kubernetes components and workloads using prometheus and grafana via HELM (On Master machine)
#
- After Installation Expose Prometheus and Grafana to the external world through Node Port
> [!Important]
> change it from Cluster IP to NodePort after changing make sure you save the file and open the assigned nodeport to the service.

```bash
kubectl edit svc stable-kube-prometheus-sta-prometheus -n prometheus
```
![image](https://github.com/sirsha/wanderlust/blob/devops/images/Screenshot%202025-07-25%20220821.png)
![image](https://github.com/user-attachments/assets/ed94f40f-c1f9-4f50-a340-a68594856cc7)

#
- Verify service
```bash
kubectl get svc -n prometheus
```

#
- Now,let’s change the SVC file of the Grafana and expose it to the outer world
```bash
kubectl edit svc stable-grafana -n prometheus
```
![image](https://github.com/sirsha/wanderlust/blob/devops/images/Screenshot%202025-07-25%20221017.png)

#
#
- Check grafana service
```bash
kubectl get svc -n prometheus
```
#
- Get a password for grafana
```bash
kubectl get secret --namespace prometheus stable-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```
> [!Note]
> Username: admin

#
- Now, view the Dashboard in Grafana
![image](https://github.com/sirsha/wanderlust/blob/devops/images/Screenshot%202025-07-25%20223900.png)
![image](https://github.com/sirsha/wanderlust/blob/devops/images/Screenshot%202025-07-25%20223725.png)

#
## Clean Up
- <b id="Clean">Delete eks cluster</b>
```bash
eksctl delete cluster --name=wanderlust --region=us-east-1
```

#

