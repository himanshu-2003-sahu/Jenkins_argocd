# Project Guide for Jenkins, Docker, and Trivy

This README provides a step-by-step guide to set up **Jenkins**, **Docker**, and **Trivy** on a Linux system. Follow the commands and instructions carefully.

## Prerequisites
- An EC2 instance or a Linux machine.
- Access to the terminal with `sudo` privileges.
- Internet connection for downloading packages.

## Steps

### Install and Start Docker
1. Update the system:
   ```bash
   yum update -y
Install Docker:
bash
Copy code
yum install -y docker
Start Docker service:
bash
Copy code
systemctl start docker
Enable Docker service to start on boot:
bash
Copy code
systemctl enable docker
Install Jenkins
Install Java:
bash
Copy code
yum install java-17-amazon-corretto -y
Add the Jenkins repository:
bash
Copy code
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
Install Jenkins:
bash
Copy code
yum install -y jenkins
Start and enable Jenkins service:
bash
Copy code
systemctl start jenkins
systemctl enable jenkins
Check Jenkins status:
bash
Copy code
systemctl status jenkins
Access Jenkins
Retrieve the initial admin password:
bash
Copy code
cat /var/lib/jenkins/secrets/initialAdminPassword
Open Jenkins in a browser:
vbnet
Copy code
http://<EC2-public-IP>:8080
If the page does not load, update the inbound security rules of your EC2 instance to allow port 8080.
Configure Docker for Jenkins
Add the jenkins user to the docker group:
bash
Copy code
usermod -aG docker jenkins
Restart Jenkins service:
bash
Copy code
systemctl restart jenkins
Create a DockerHub Account and Login
Create an account on DockerHub.
Log in to Docker:
bash
Copy code
docker login
Enter your DockerHub username and password when prompted.
Install Trivy
Navigate to the /etc/yum.repos.d/ directory:
bash
Copy code
cd /etc/yum.repos.d/
Add the Trivy repository:
bash
Copy code
cat << EOF | sudo tee -a /etc/yum.repos.d/trivy.repo
[trivy]
name=Trivy repository
baseurl=https://aquasecurity.github.io/trivy-repo/rpm/releases/\$basearch/
gpgcheck=1
enabled=1
gpgkey=https://aquasecurity.github.io/trivy-repo/rpm/public.key
EOF
Install Trivy:
bash
Copy code
yum install -y trivy
Jenkins Jobs
Job 1: Pull Dockerfile and Build the Image
Create a New Jenkins Job:
Go to the Jenkins Dashboard.
Click New Item, enter a name (e.g., Build_Docker_Image), and select Freestyle project.
Configure the Job:
Source Code Management:
Select Git and add the repository URL:
arduino
Copy code
https://github.com/Ashik-Domain/Bookinfo_app.git
Use the branch main.
Build Steps:
Add the following shell command in the Build section:
bash
Copy code
cd src/productpage
docker build -t productpage-app .
Post-Build Actions:
Add Build other projects and specify the next job (e.g., Test_Docker_Image).
Save the Job.
Job 2: Test the Docker Image with Trivy
Create a New Jenkins Job:
Name it Test_Docker_Image.
Configure the Job:
Add a Build Step:
bash
Copy code
trivy image --severity HIGH,CRITICAL productpage-app
Post-Build Actions:
Add Build other projects and specify the next job (e.g., Push_to_DockerHub).
Save the Job.
Job 3: Push the Image to DockerHub
Create a New Jenkins Job:
Name it Push_to_DockerHub.
Configure the Job:
Build Steps:
Add the following shell command:
bash
Copy code
docker tag productpage-app <your_dockerhub_username>/productpage-app:latest
docker push <your_dockerhub_username>/productpage-app:latest
Replace <your_dockerhub_username> and <your_dockerhub_password> with your DockerHub credentials. Alternatively, use the Jenkins credentials ID to inject them dynamically.
Save the Job.
Connecting the Jobs
Set Upstream and Downstream Triggers:
Go to Job 1 (Build_Docker_Image) and configure its Post-Build Action to trigger Job 2.
Similarly, go to Job 2 (Test_Docker_Image) and configure its Post-Build Action to trigger Job 3.
Additional Commands
Check free memory:
bash
Copy code
free -m
List Docker images:
bash
Copy code
docker images
Notes
Ensure that you have sufficient privileges when running these commands. Use sudo when necessary.
Refer to the official Trivy documentation for more details on installation.
Next Steps
After completing the Jenkins pipeline, you can use ArgoCD to deploy the Docker image to a Kubernetes cluster and manage the application's lifecycle. For more details, refer to the DevOps ArgoCD Project.

Bookinfo Repository
For the developer app:
Bookinfo GitHub Repository
History of Commands
The following commands were executed during the setup process:

bash
Copy code
# Update and install necessary packages
yum update -y
yum install -y docker
systemctl start docker
yum install java-17-amazon-corretto -y

# Configure Jenkins
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
yum install -y jenkins
systemctl start jenkins
systemctl enable jenkins
systemctl status jenkins

# DockerHub login
docker login

# Configure Jenkins user permissions
usermod -aG docker jenkins
systemctl restart jenkins

# Install Trivy
cd /etc/yum.repos.d/
cat << EOF | sudo tee -a /etc/yum.repos.d/trivy.repo
[trivy]
name=Trivy repository
baseurl=https://aquasecurity.github.io/trivy-repo/rpm/releases/\$basearch/
gpgcheck=1
enabled=1
gpgkey=https://aquasecurity.github.io/trivy-repo/rpm/public.key
EOF
yum install -y trivy
rust
Copy code

This updated README is structured for clarity and includes essential commands, configuratio
