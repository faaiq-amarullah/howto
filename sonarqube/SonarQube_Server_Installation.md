# Deploy SonarQube Server on Docker

## Pre-requisites

### System Requirements

- **OS**: Ubuntu 20.04+/CentOS 8+ or any modern Linux distribution
- **Dependencies**: Java Version 21, wget, unzip, vim or another text editor, Docker Version 20.10+, Docker Compose Version 2.0+, Helm Version 3
- **Resources**: Minimum 4GB RAM, 2 vCPUs, 10GB disk space
- **Recommended**: 8GB RAM, 4 vCPUs for production

### Database Requirements

Several external database engines are supported.

- **Postgresql**: 13 to 17
- **Microsoft SQL Server**: 2022 (MSSQL Server 16.0); 2019 (MSSQL Server 15.0); 2017 (MSSQL Server 14.0); 2016 (MSSQL Server 13.0); With bundled Microsoft JDBC driver
- **Oracle**: 23ai, 21C, 19C, XE Editions. Recommend to use the latest Oracle JDBC driver

> [!NOTE]
> The embedded H2 database is used by default. It is recommended for tests but not for production use.

## Deploy with Binary File

### Step 1: Pre-installation steps

Install java first:
```bash
sudo apt update
sudo apt install openjdk-21-jdk
java -version
```

You must ensure that:
- The maximum number of memory map areas a process may have (vm.max_map_count) is greater than or equal to 524288.
- The maximum number of open file descriptors (fs.file-max) is greater than or equal to 131072.
- The user running SonarQube Server can open **at least** 131072 file descriptors.
- The user running SonarQube Server can open **at least** 8192 threads.

You can setting up like this:

```bash
sudo vim /etc/sysctl.d/99-sonarqube.conf
```
```bash
vm.max_map_count=524288
fs.file-max=131072
```
```bash
sudo sysctl --system
```

Creating a dedicated user for SonarQube will be beneficial. You can create and configure the user as follows:

```bash
sudo useradd -d /opt/sonarqube -s /bin/bash -m --system sonarqube
```

Add following lines in the configuration file:

```bash
sudo vim /etc/security/limits.d/99-sonarqube.conf
```
```bash
sonarqube   -   nofile   131072
sonarqube   -   nproc    8192
```

### Step 2: Setup PostgreSQL Database

Automated repository configuration:

```bash
sudo apt install -y postgresql-common
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
```

Install PostgreSQL version 14

```bash
sudo apt install postgresql-14
```

Setup Database for SonarQube Server

```bash
# Enter psql shell
sudo -u postgres psql
```
```sql
CREATE USER sonarqube WITH PASSWORD 'SecurePassword!123';
CREATE DATABASE sonarqube OWNER sonarqube;
GRANT ALL PRIVILEGES ON DATABASE sonarqube TO sonarqube;

# Exit psql shell
quit
```

### Step 3: Install SonarQube Server

Download binary file and move it to the sonarqube home directory:

```bash
# Download SonarQube Binary
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-25.11.0.114957.zip

# Unzip the downloaded file
unzip sonarqube-25.11.0.114957.zip

# Move the file to the sonarqube home directory
sudo mv sonarqube-25.11.0.114957/* /opt/sonarqube

# Change ownerhsip the folder
sudo chown -R sonarqube:sonarqube /opt/sonarqube
```

Edit the configuration file:

```bash
sudo vim /opt/sonarqube/conf/sonar.properties
```
```bash
#Uncommet the below lines and add your credentials
sonar.jdbc.username=sonarqube
sonar.jdbc.password=SecurePassword!123

#Uncommmet the jdbc url section under postgresql
sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube

#Also change uncomment the blow lines and edit web host and port
sonar.web.host=0.0.0.0
sonar.web.port=9000
```

Run SonarQube server as a systemd service:

```bash
sudo vim /etc/systemd/system/sonarqube.service
```
```bash
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
User=sonarqube
Group=sonarqube
Restart=always
LimitNOFILE=131072
LimitNPROC=8192

[Install]
WantedBy=multi-user.target
```

Reload systemd, enable and start the sonarqube service:

```bash
sudo systemctl daemon-reload
sudo systemctl start sonarqube.service
sudo systemctl enable sonarqube.service
sudo systemctl status sonarqube.service
```

## Deploy with docker-run

Create network for internal communication

```bash
docker network create sonar
```

Create volumes to store data persistently even the container has been deleted. You need to create 3 volumes:
- `sonarqube_data`: contains data files, such as Elasticsearch indexes
- `sonarqube_logs`: contains SonarQube Server logs about access, web process, CE process, and Elasticsearch
- `sonarqube_extensions`: will contain any plugins you install and the Oracle JDBC driver if necessary.

Create the volumes with the following commands:

```bash
docker volume create --name sonarqube_data
docker volume create --name sonarqube_logs
docker volume create --name sonarqube_extensions
```

Run sonarqube container:

```bash
docker run -d --name sonarqube \
    -p 9000:9000 \
    --ulimit nofile=131072 \
    --ulimit nproc=8192 \
    -v sonarqube_data:/opt/sonarqube/data \
    -v sonarqube_logs:/opt/sonarqube/logs \
    -v sonarqube_extensions:/opt/sonarqube/extensions \
    sonarqube:community
```

> [!NOTE]
> Change image tag to use different SonarQube distribution. Check on [DockerHub](https://hub.docker.com/_/sonarqube).

Verify sonarqube container:

```bash
docker ps
docker logs sonarqube -f
```

## Run with docker-compose

Create `docker-compose.yml` file:

```yaml
version: '3.8'

services:
  sonarqube:
    image: sonarqube:community
    container_name: sonarqube
    depends_on:
      - postgres
    environment:
      - SONAR_JDBC_URL=jdbc:postgresql://postgres:5432/sonarqube
      - SONAR_JDBC_USERNAME=sonarqube
      - SONAR_JDBC_PASSWORD=sonarqube_password
      - SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_logs:/opt/sonarqube/logs
      - sonarqube_extensions:/opt/sonarqube/extensions
    ports:
      - "9000:9000"
    ulimits:
      nofile: 131072
      nproc: 8192
    networks:
      - sonar
    restart: unless-stopped

  postgres:
    image: postgres:17
    container_name: postgres-sonar
    environment:
      - POSTGRES_USER=sonarqube
      - POSTGRES_PASSWORD=sonarqube_password
      - POSTGRES_DB=sonarqube
    volumes:
      - postgres:/var/lib/postgresql/data
    networks:
      - sonar
    restart: unless-stopped

volumes:
  sonarqube_data:
    external: false
  sonarqube_logs:
    external: false
  sonarqube_extensions:
    external: false
  postgres:
    external: false

networks:
  sonar:
    driver: bridge
```

Deploy SonarQube:

```bash
# Start services
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f sonarqube

# Stop services
docker-compose down
```

> [!NOTE]
> Change image tag to use different SonarQube distribution. Check on [DockerHub](https://hub.docker.com/_/sonarqube).


## Deploy on Kubernetes

Add sonarqube helm repository:

```bash
helm repo add sonarqube https://SonarSource.github.io/helm-chart-sonarqube
helm repo update
```

Download default `values.yaml` file and edit on specific line:

```bash
helm show values sonarqube/sonarqube > values.yaml
vim values.yaml
```

```yaml
community:
  enabled: true

ingress:
  enabled: true
  hosts:
    - name: sonarqube.your-org.com
      path: /
  ingressClassName: nginx
  annotations: 
    nginx.ingress.kubernetes.io/proxy-body-size: "64m"

monitoringPasscode: "SecurePasscode!123"

persistence:
  enabled: true
```

Install helm:

```bash
helm upgrade --install -n sonarqube sonarqube sonarqube/sonarqube --values values.yaml --create-namespace
```

Verify installation:

```bash
helm list -n sonarqube
kubectl get all -n sonarqube
```

## Initial Configuration

1. Access SonarQube: http://your-server-ip-or-domain:9000
2. Default credentials: admin/admin
3. Change admin password on first login
4. Generate authentication token for CI/CD integration

## Reference
Visit [SonarQube Docs](https://docs.sonarsource.com/sonarqube-community-build/)