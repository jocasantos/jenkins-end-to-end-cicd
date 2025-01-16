# Jenkins Pipeline for Java based application using Maven, SonarQube, Argo CD and Kubernetes

![Screenshot 2023-03-28 at 9 38 09 PM](https://user-images.githubusercontent.com/43399466/228301952-abc02ca2-9942-4a67-8293-f76647b6f9d8.png)


Here are the step-by-step details to set up an end-to-end Jenkins pipeline for a Java application using SonarQube, Argo CD, and Kubernetes:

Prerequisites:

   -  Java application code hosted on a Git repository (already done) 
   -  Jenkins server
   -  Kubernetes cluster
   -  Argo CD

Steps:

   1. Test the Java application locally:
      - See > README.md of spring-boot-app
      - `cd java-maven-sonar-argocd-helm-k8s/spring-boot-app`
      - `mvn clean package` for artifact generation
      - Test it using Docker way (adviced), tag the image with your dockerhub username <dockerhub-username>/ultimate-cicd-pipeline:v1> and push it to your dockerhub repository

   2. Create a EC2 instance and install Jenkins on it.
      - Free tier isn't enough, so you need to upgrade the instance type for "large" (8Gb) memory
      - Configure the security group to allow all trafic (dont do it in production, is just for a demo purpose)
      - Connect by SSH to the instance and install Jenkins
      - Install Java
         ```bash
         sudo apt update
         sudo apt install openjdk-17-jdk
         ```
      - Install Jenkins
         ```bash
         curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
         /usr/share/keyrings/jenkins-keyring.asc > /dev/null
         echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
         https://pkg.jenkins.io/debian binary/ | sudo tee \
         /etc/apt/sources.list.d/jenkins.list > /dev/null
         sudo apt-get update
         sudo apt-get install jenkins
         ```
      - Start Jenkins
            `sudo systemctl start jenkins`
      - Open Jenkins in your browser
            `<your-ec2-public-ip>:8080`
      - Get the initial password  
            `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`
      - Install default and most used plugins    
      - Install `docker pipeline` plugin
      - Install `sonarqube scanner` plugin
      - Restart Jenkins (best practices when install new plugins).

   3. Create a new Jenkins pipeline:
      - In Jenkins, create a new pipeline job
      - Pipeline: `Pipeline script from SCM`
      - SCM: `Git`
      - Repository URL: `your-repo-url`
      - Branches to build: `*/main`
      - Script Path: `spring-boot-app/Jenkinsfile`
      - Save
   
   4. Install SonarQube locally and configure it:
         ```bash
         apt install unzip
         adduser sonarqube and change the user to sonarqube
         wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
         unzip *
         chmod -R 755 /home/sonarqube/sonarqube-9.4.0.54424
         chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-9.4.0.54424
         cd sonarqube-9.4.0.54424/bin/linux-x86-64/
         ./sonar.sh start
         ```
      - Hurray !! Now you can access the `SonarQube Server` on `http://<ip-address>:9000` (admin/admin)
      - Go to `My Account` -> `Security` -> `Generate Token` -> `Copy the token`
      - Go to Jenkins -> `Manage Jenkins` -> `Credentials` -> `System` -> `Global Credentials` -> `Add Credentials` -> `Secret Text` -> `Add the token`

   5. Install docker on your instance
      ```bash	
      sudo apt update
      sudo apt install docker.io
      ```
      - Grant permissions to the Jenkins user to run Docker commands
      ```bash
      sudo su - 
      usermod -aG docker jenkins
      usermod -aG docker ubuntu
      systemctl restart docker
      ```
      - Restart Jenkins
      `http://<ec2-instance-public-ip>:8080/restart`
   
   6. Start your Kubernetes cluster (I'll use minikube, but can use EKS or other)

   7. Install Argo CD on your Kubernetes cluster
      - Go https://operatorhub.io/operator/argocd-operator, press `Install` and follow the instructions
   
   8. Add your docker and github credentials to Jenkins
      - For GitHub -> Go to Jenkins -> `Manage Jenkins` -> `Credentials` -> `System` -> `Global Credentials` -> `Add Credentials` -> `Secret Text` -> `Add the token` (id: github)
      - For Docker -> Go to Jenkins -> `Manage Jenkins` -> `Credentials` -> `System` -> `Global Credentials` -> `Add Credentials` -> `Username with password` -> `Add the credentials` (id: docker-cred)

   9. Dont forget to change the `jenkinsfile` with your own configurations
      - Change the path for your docker image in spring-boot-app-manifests > deployment.yml 

   10. Run the Jenkins pipeline
      - Go to Jenkins -> `Your pipeline` -> `Build Now`
      - Dont worry if your pipeline fails, you can check the logs and fix the issues
      - If success, you can access see your sonar report on `http://<ip-address>:9000`

   11. Check the Argo CD
      - apply Argo CD manifest
      ```bash
      kubectl apply -f argocd-basic.yml
      ```
      - Check the NodePort for `argocd-server` 
      ```bash
      kubectl get svc
      ```
      - Access the Argo CD on `http://<ip-address>:<node-port>`
      - Get the password
      ```bash
      kubectl get secret
      kubectl edit secret example-argocd-cluster
      ```
      - Decode the password
      ```bash
      echo 'your-password' | base64 -d
      ```
      - Go to Argo CD UI and create a NEW APP
      - Application Name: example
      - Project Name: default
      - Sync Policy: Automatic
      - Repository URL: <your-repo-url>
      - Path: spring-boot-app-manifests
      - Cluster URL: https://kubernetes.default.svc
      - Namespace: default

   12. Check if the pods are runing on Kubernetes
      ```bash
      kubectl get pods
      ```
      - Expose pod to NodePort, to see the application running
      ```bash
      kubectl expose pod spring-boot-app-<check-your-pod-name> --type=NodePort --port=8080
      ```
      - Check the NodePort
      ```bash
      kubectl get svc
      ```
      - Access the application on `http://<minikube-ip>:<node-port>`
      `minikube ip`


This end-to-end Jenkins pipeline will automate the entire CI/CD process for a Java application, from code checkout to production deployment, using popular tools like SonarQube, Argo CD, and Kubernetes.

Congratulations! You have successfully set up the entire CI/CD process for a Java application using Jenkins, Maven, SonarQube, SHELL scripting, Argo CD, and Kubernetes :tada::tada::tada:
