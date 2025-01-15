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
       1.1 See > README.md of spring-boot-app
       1.1 `cd java-maven-sonar-argocd-helm-k8s/spring-boot-app`
       1.3 `mvn clean package` for artifact generation
       1.2 Test it using Docker way (adviced), tag the image with your dockerhub username <dockerhub-username>/ultimate-cicd-pipeline:v1> and push it to your dockerhub repository

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
      - In Jenkins, create a new pipeline job and configure it with the Git repository URL for the Java application.
      - Add a Jenkinsfile to the Git repository to define the pipeline stages (already done).
   
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
      - Go https://operatorhub.io/operator/argocd-operator and follow the instructions
   
   8. Add your docker and github credentials to Jenkins
      - For GitHub -> Go to Jenkins -> `Manage Jenkins` -> `Credentials` -> `System` -> `Global Credentials` -> `Add Credentials` -> `Secret Text` -> `Add the token` (id: github)
      - For Docker -> Go to Jenkins -> `Manage Jenkins` -> `Credentials` -> `System` -> `Global Credentials` -> `Add Credentials` -> `Username with password` -> `Add the credentials` (id: docker-cred)

   9. Dont forget to change the `jenkinsfile` with your own configurations
      - Change the path for your docker image in spring-boot-app-manifests > deployment.yml 

   10. Run the Jenkins pipeline and verify that the Java application is built, tested, and deployed to Kubernetes using Argo CD.







This end-to-end Jenkins pipeline will automate the entire CI/CD process for a Java application, from code checkout to production deployment, using popular tools like SonarQube, Argo CD, and Kubernetes.
