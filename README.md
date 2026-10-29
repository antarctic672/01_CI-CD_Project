# Java Web Application — CI/CD Pipeline

## Project Overview

This project demonstrates how a Java web application can be built and deployed using a simple CI/CD pipeline. The goal was to connect several DevOps tools and understand how they work together in a real environment.

The project uses AWS EC2, GitHub, Jenkins, Maven, Docker, and Apache Tomcat. Two EC2 instances separate the build process from the application runtime: one runs Jenkins, while the other is prepared for Docker deployment.

Source code: [GitHub Repository: registration-app](https://github.com/antarctic672/registration-app?utm_source=chatgpt.com)

## Architecture

**GitHub**

The source code is stored in a GitHub repository. The `main` branch is used as the source for the Jenkins build. The project contains separate `server` and `webapp` Maven modules.

**Jenkins Server**

Jenkins runs on an AWS EC2 instance and manages the build process through a configured job.

**Maven**

Maven compiles the project and packages the web application into a WAR file, which is used as the deployment artifact.

**Docker-Host**

A separate EC2 instance hosts Docker. The WAR file is intended to be transferred from Jenkins over SSH and deployed inside a Tomcat container.

**Apache Tomcat**

Tomcat runs inside the Docker container and serves the Java web application.

## CI/CD Workflow

The planned deployment workflow consists of these stages:

1. Code changes are pushed to GitHub.
2. A GitHub webhook notifies Jenkins about the push.
3. Jenkins retrieves the latest code from the `main` branch.
4. Maven builds the project and generates the WAR file.
5. Jenkins transfers the artifact to Docker-Host over SSH.
6. Docker builds an image containing Tomcat and the application.
7. The container serves the application through port **8086** on the host, mapped to port **8080** inside the container.

## Infrastructure and Networking

The infrastructure consists of two Ubuntu-based AWS EC2 instances.

- **Jenkins Server:** source code checkout and application build.
- **Docker-Host:** container runtime and application hosting.

The main network ports are:

- **8080** — Jenkins web interface.
- **22** — SSH access and server communication.
- **8086** — external access to the application.

AWS Security Groups control access to these ports. The required rules must allow connections from the appropriate sources.

## Jenkins and GitHub Integration

A Jenkins job was configured to retrieve the repository and run the Maven build. The GitHub webhook trigger was also enabled to support automatic builds after code changes.

This setup provides the foundation for a CI workflow without requiring every build to be started manually.

## Project Configuration

The repository contains the `server` and `webapp` modules, the root `pom.xml`, and the project documentation.

During setup, I updated the Maven Compiler Plugin configuration to version `3.13.0` and configured Java release `8` to match the project's compilation requirements.

I then ran the Maven build and verified that the WAR artifact was generated in `webapp/target`. This artifact is intended to be transferred to Docker-Host for deployment.

## Troubleshooting

###  Jenkins Build Was Aborted

One of the Jenkins builds stopped unexpectedly and was marked as **Aborted**. Some Maven modules did not run.

I checked the Jenkins Console Output, but the reason for the interruption was not immediately clear. I then checked the server's available memory and found that the instance had less than 1 GB of RAM, with approximately 230 MB available at the time.

This made limited memory a possible cause to investigate, and I was right The experience showed me why it is important to check both Jenkins logs and server resources when troubleshooting a failed build.

### SSH Password Authentication

I used username/password authentication for SSH access to Docker-Host. After setting the Ubuntu user's password, I checked the SSH server configuration.

I found conflicting settings: the main SSH configuration file allowed password authentication, while another configuration file disabled it(it was an additional configuration file from AWS, embedded into VPS).

## Project Goals and Outcomes

The project provided practical experience with:

- Configuring Jenkins, Java, and Maven on AWS EC2.
- Building a multi-module Java application and generating a WAR artifact.
- Connecting GitHub to Jenkins and configuring a webhook trigger.
- Separating build and deployment responsibilities across two servers.
- Preparing a Docker and Tomcat deployment environment.
- Investigating Jenkins build interruptions.
- Troubleshooting SSH connectivity, authentication, and AWS Security Group settings.

## Conclusion

This project brought together several tools commonly used in DevOps: AWS, GitHub, Jenkins, Maven, Docker, and Apache Tomcat.

The project improved my understanding of CI/CD workflows, server configuration, and container-based deployment.

The work was again based on the "Virtual TechBox" video. His videos really help and inspire me to keep learning. Because it is really hard to come up with a project on your own, especially for a role like that (DevOps). This time I tried to do some work by myself: set up EC2 instances, dependencies for them; set up a Jenkins server; installed maven and docker; created and linked the webhook. Almost for everything related to the configuration of the pipeline - I used the help from the video. But, I really noticed a change - after the big introduction project 00, I started to understand the pipeline better, in a way that when the author from "Virtual TechBox" moves along the pipeline configuration, I can sometimes predict what is going to be the next step or what he is going to put inside a specific option/bracket.

Also, regarding the troubleshooting. This time, there were some really annoying issues which made me struggle for a moment. But they made me realise that the solution to some problems could be very obvious and easy to fix. Nothing wrong with checking very basic things.
