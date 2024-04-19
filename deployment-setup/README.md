# Board Game

Brief description of the project setup to deploy.

## Table of Contents

1. [AWS CLI Setup](#aws-cli-setup)
2. [kubectl Setup](#kubectl-setup)
3. [eksctl Setup](#eksctl-setup)
4. [Cleanup EKS Cluster](#cleanup-eks-cluster)
5. [Docker Installation](#docker-installation)
6. [SonarQube](#sonarqube)
7. [Nexus](#nexus)
8. [Prometheus Setup](#prometheus-setup)
9. [Blackbox Exporter](#blackbox-exporter)
10. [Node Exporter for Jenkins](#node-exporter-for-jenkins)
11. [Grafana Setup](#grafana-setup)
12. [Edit Prometheus Configuration](#edit-prometheus-configuration)
13. [Restart Prometheus](#restart-prometheus)
14. [Dashboard IDs](#dashboard-ids)

## 1. AWS CLI Setup 

Instructions for setting up the AWS CLI, including downloading the package, installing dependencies, and verifying the installation.

```
# Example code snippets
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip -y 
unzip awscliv2.zip
sudo ./aws/install
ls -l /usr/local/bin/aws
/usr/local/bin/aws --version

```
## kubectl setup 

```
sudo curl --silent --location -o /usr/local/bin/kubectl   https://s3.us-west-2.amazonaws.com/amazon-eks/1.22.6/2022-03-09/bin/linux/amd64/kubectl

sudo chmod +x /usr/local/bin/kubectl 

kubectl version --short --client

```

## eksctl setup

```
eksctl create cluster --name demo --region us-east-1 --nodegroup-name my-nodes --node-type t3.small --managed --nodes 2

```

## cleanup eks cluster

```
eksctl delete cluster --name demo --region us-east-1

```

## install docker 

```
#!/bin/bash

# Update package manager repositories
sudo apt-get update

# Install necessary dependencies
sudo apt-get install -y ca-certificates curl

# Create directory for Docker GPG key
sudo install -m 0755 -d /etc/apt/keyrings

# Download Docker's GPG key
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

# Ensure proper permissions for the key
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add Docker repository to Apt sources
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update package manager repositories
sudo apt-get update

sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin 

```
## sonarqube 

```
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

## nexus

```
docker run -d --name nexus -p 8081:8081 sonatype/nexus3:latest
```

## pom.xml file for nexus

```
	<distributionManagement>
    <repository>
         <id>maven-releases</id>
         <url>http://44.203.164.50:8081/repository/maven-releases/</url>
     </repository>
     <snapshotRepository>
         <id>maven-snapshots</id>
         <url>http://44.203.164.50:8081/repository/maven-snapshots/</url>
     </snapshotRepository>
    </distributionManagement>
```

## prometheus setup 

```
wget https://github.com/prometheus/prometheus/releases/download/v2.51.2/prometheus-2.51.2.linux-amd64.tar.gz

tar -xvf prometheus-2.51.2.linux-amd64.tar.gz
rm -rf prometheus-2.51.2.linux-amd64.tar.gz

cd prometheus-2.51.2.linux-amd64
# to run prometheus in background
./prometheus &

```

## black box expoter

```
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux-amd64.tar.gz

tar -xvf blackbox_exporter-0.25.0.linux-amd64.tar.gz

```

## node expoter in jenkins server to moniter jenkins

```
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz

```

## grafana setup 

```
sudo apt-get install -y adduser libfontconfig1 musl
wget https://dl.grafana.com/enterprise/release/grafana-enterprise_10.4.2_amd64.deb
sudo dpkg -i grafana-enterprise_10.4.2_amd64.deb

```

## edit prometheus.yml file to add blackbox and node expoters ips


```
scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['3.86.210.209:9100']

  - job_name: 'jenkins'
    metrics_path: '/prometheus'
    static_configs:
      - targets: ['3.86.210.209:8080']

#----> black box

scrape_configs:
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]  # Look for a HTTP 200 response.
    static_configs:
      - targets:
        - http://prometheus.io    # Target to probe with http.
        - https://prometheus.io   # Target to probe with https.
        - http://example.com:8080 # Target to probe with http on port 8080.
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115  # The blackbox exporter's real hostname:port.

```

## restart prometheus

```
pgrep promethus

```

## dashboard ids

```
7587 prom blackbox
1860 nodeprot jenkins
```
