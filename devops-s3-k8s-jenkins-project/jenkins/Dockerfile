# Jenkins LTS with AWS CLI inside
FROM jenkins/jenkins:lts-jdk17

USER root
RUN apt-get update && apt-get install -y     curl unzip awscli git     && rm -rf /var/lib/apt/lists/*

# Optional: install kubectl (useful for cluster ops if needed later)
RUN curl -L -o /usr/local/bin/kubectl https://storage.googleapis.com/kubernetes-release/release/v1.29.6/bin/linux/amd64/kubectl &&     chmod +x /usr/local/bin/kubectl

USER jenkins
