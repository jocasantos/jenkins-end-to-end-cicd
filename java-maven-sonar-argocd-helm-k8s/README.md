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
       1.2 Test it using Docker way (adviced)

   2. Create a EC2 instance and install Jenkins on it.
      2.1 Free tier isn't enough, so you need to upgrade the instance type for "large" (8Gb) memory
      2.2 Configure the security group to allow all trafic (dont do it in production, is just for a demo purpose)
      2.2 Connect by SSH to the instance and install Jenkins
      2.3 Install Java
         ```bash
         sudo apt update
         sudo apt install openjdk-17-jdk
         ```
      2.4 Install Jenkins
         ```bash
         curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
         /usr/share/keyrings/jenkins-keyring.asc > /dev/null
         echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
         https://pkg.jenkins.io/debian binary/ | sudo tee \
         /etc/apt/sources.list.d/jenkins.list > /dev/null
         sudo apt-get update
         sudo apt-get install jenkins
         ```
      2.5 Start Jenkins
            `sudo systemctl start jenkins`
      2.6 Open Jenkins in your browser
            `<your-ec2-public-ip>:8080`
      2.7 Get the initial password  
            `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`
      2.8 Install default and most used plugins    




    3. Install the necessary Jenkins plugins:
       1.1 Git plugin
       1.2 Maven Integration plugin
       1.3 Pipeline plugin
       1.4 Kubernetes Continuous Deploy plugin

    4. Create a new Jenkins pipeline:
       2.1 In Jenkins, create a new pipeline job and configure it with the Git repository URL for the Java application.
       2.2 Add a Jenkinsfile to the Git repository to define the pipeline stages.

    3. Define the pipeline stages:
        Stage 1: Checkout the source code from Git.
        Stage 2: Build the Java application using Maven.
        Stage 3: Run unit tests using JUnit and Mockito.
        Stage 4: Run SonarQube analysis to check the code quality.
        Stage 5: Package the application into a JAR file.
        Stage 6: Deploy the application to a test environment using Helm.
        Stage 7: Run user acceptance tests on the deployed application.
        Stage 8: Promote the application to a production environment using Argo CD.

    4. Configure Jenkins pipeline stages:
        Stage 1: Use the Git plugin to check out the source code from the Git repository.
        Stage 2: Use the Maven Integration plugin to build the Java application.
        Stage 3: Use the JUnit and Mockito plugins to run unit tests.
        Stage 4: Use the SonarQube plugin to analyze the code quality of the Java application.
        Stage 5: Use the Maven Integration plugin to package the application into a JAR file.
        Stage 6: Use the Kubernetes Continuous Deploy plugin to deploy the application to a test environment using Helm.
        Stage 7: Use a testing framework like Selenium to run user acceptance tests on the deployed application.
        Stage 8: Use Argo CD to promote the application to a production environment.

    5. Set up Argo CD:
        Install Argo CD on the Kubernetes cluster.
        Set up a Git repository for Argo CD to track the changes in the Helm charts and Kubernetes manifests.
        Create a Helm chart for the Java application that includes the Kubernetes manifests and Helm values.
        Add the Helm chart to the Git repository that Argo CD is tracking.

    6. Configure Jenkins pipeline to integrate with Argo CD:
       6.1 Add the Argo CD API token to Jenkins credentials.
       6.2 Update the Jenkins pipeline to include the Argo CD deployment stage.

    7. Run the Jenkins pipeline:
       7.1 Trigger the Jenkins pipeline to start the CI/CD process for the Java application.
       7.2 Monitor the pipeline stages and fix any issues that arise.

This end-to-end Jenkins pipeline will automate the entire CI/CD process for a Java application, from code checkout to production deployment, using popular tools like SonarQube, Argo CD, and Kubernetes.
