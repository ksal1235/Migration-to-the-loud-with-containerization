# Migration to the Сloud with containerization (Docker & Docker Compose):

In this project, we will use a CI tool that is already well-known to us Jenkins - for Continous Integration (CI). So, when it is time to write Jenkinsfile, we will update our Terraform code to spin up an EC2 instance for Jenkins and run Ansible to install & configure it.

To begin our migration project from VM based workload, we need to implement a Proof of Concept (POC). In many cases, it is good to start with a small-scale project with minimal functionality to prove that technology can fulfill specific requirements. So, this project will be a precursor before you can move on to deploy enterprise-grade microservice solutions with Docker. And so, Project 21 through to 30 will gradually introduce concepts and technologies as we move from POC onto enterprise level deployments.

We can start with our own workstation or spin up an EC2 instance to install Docker engine that will host our Docker containers.

Remember our Tooling website? It is a PHP-based web solution backed by a MySQL database - all technologies we are already familiar with and which we shall be comfortable using by now.

So, let us migrate the Tooling Web Application from a VM-based solution into a containerized one.
