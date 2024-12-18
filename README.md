# Author : Thuy Linh CO


# CI/CD Pipeline with GitHub Actions and Ansible

## **Project Overview**
This repository demonstrates the setup of a complete CI/CD pipeline using **GitHub Actions** to build, test, and deploy an application, alongside **Ansible** for automated server provisioning and application deployment. The goal was to:

1. Implement a **CI/CD pipeline** that automates the process of testing, building Docker images, and deploying the application.
2. Use **Ansible** to configure the server environment and provision Docker containers for:
   - **Database**
   - **Backend API**
   - **Proxy server (HTTPD)**
3. Document the solutions and challenges encountered during the TP (Travaux Pratiques).

---

## **Technologies Used**
- **GitHub Actions** for CI/CD pipeline
- **Ansible** for server provisioning
- **Docker** for containerization
- **Maven** for building the Java backend
- **Spring Boot** for the backend API
- **PostgreSQL** as the database
- **Apache HTTPD** as the proxy server
- **SonarCloud** for code quality analysis

---

## **Actions Performed**

### 1. **CI/CD Pipeline with GitHub Actions**
- **Workflow 1:** *CI Pipeline*
   - Triggered on push or pull request to `main` and `develop` branches.
   - Steps:
     - Set up Java and Maven to build and test the backend.
     - Perform code quality analysis with SonarCloud.
     - Build Docker images for **backend**, **database**, and **proxy**.
     - Push images to Docker Hub only on the `main` branch.

- **Workflow 2:** *Deployment With Ansible*
   - Triggered after the CI workflow completes.
   - Steps:
     - Set up SSH access to the server using GitHub Secrets.
     - Deploy the application using Ansible to provision Docker containers on the remote server.

### 2. **Server Provisioning with Ansible**
- Created an Ansible `setup.yml` file to configure SSH connection and server details.
- Defined roles for modularity:
   - **docker:** Install Docker and ensure it is running.
   - **database:** Deploy PostgreSQL container with environment variables.
   - **backend:** Deploy the backend API container.
   - **proxy:** Configure and deploy HTTPD as a reverse proxy.
   - **network:** Create a custom Docker network to link all containers.

- Commands used to test and run Ansible locally:
   ```bash
   ansible-playbook -i ansible/inventories/setup.yml ansible/playbook.yml
   ```

### 3. **Application Deployment**
- Deployed three Docker containers:
   - **Database**: PostgreSQL container using environment variables for configuration.
   - **Backend API**: Spring Boot application packaged into a JAR using Maven.
   - **Proxy**: HTTPD server configured to redirect requests to the backend.


- Docker images pushed to Docker Hub:
   - **Backend:** `thuuuuyyy/tp-devops-api:latest`
   - **Database:** `thuuuuyyy/tp-devops-db:latest`
   - **Proxy:** `thuuuuyyy/tp-devops-http:latest`

---

## **Challenges and Solutions**

1. **SSH Connection Failure in GitHub Actions**
   - **Problem:** SSH connection failed due to missing private key.
   - **Solution:** Added the private key as a secret (`SSH_PRIVATE_KEY`) in GitHub and configured SSH in the workflow:
     ```yaml
     - name: Configure SSH
       run: |
         mkdir -p ~/.ssh
         echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
         chmod 600 ~/.ssh/id_rsa
     ```

2. **Ansible Role Not Found**
   - **Problem:** Roles were placed in the wrong directory (`inventories/roles`).
   - **Solution:** Moved roles to the correct `ansible/roles` directory.

3. **Application Crashing in Backend**
   - **Problem:** Backend could not find the database due to incorrect environment variables.
   - **Solution:** Updated the `docker_container` task with correct `DATABASE_URL`:
     ```yaml
     env:
       DATABASE_URL: postgres://user:password@my-database/dbname
     ```

---

## **Answers to questions**

1. **What are testcontainers?**
   Testcontainers are Java libraries that allow you to run Docker containers during tests. They provide lightweight, disposable databases or other services for integration testing.

2. **Quality Gate Configuration**
   - SonarCloud was integrated to analyze code quality during the Maven build process.
   - Configuration in the GitHub Actions workflow:
     ```yaml
     - name: Build and analyze
       run: mvn -B verify sonar:sonar \
         -Dsonar.projectKey=thuy29_devops-livecoding \
         -Dsonar.organization=thuy29 \
         -Dsonar.host.url=https://sonarcloud.io \
         -Dsonar.login=${{ secrets.SONAR_TOKEN }}
     ```

3. **Documented Playbook**
   - Playbook (`playbook.yml`) includes roles for Docker installation, database setup, backend deployment, and proxy configuration.

---

## **How to Reproduce the pipeline**

1. Clone this repository:
   ```bash
   git clone https://github.com/your-username/devops-livecoding-main.git
   ```

2. Set up GitHub Secrets:
   - `SSH_PRIVATE_KEY`: Your SSH private key
   - `SONAR_TOKEN`: Token for SonarCloud
   - `DOCKER_USERNAME` and `DOCKER_PASSWORD`: Credentials for Docker Hub

3. Run the workflows:
   - CI Pipeline: Triggered on push or PR to `main`
   - Deployment: Triggered after CI completion.


