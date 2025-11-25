# Deploy SonarQube Server on Docker

## Pre-requisites

### System Requirements

- **OS**: Ubuntu 20.04+/CentOS 8+ or any modern Linux distribution
- **Docker**: Version 20.10+
- **Docker Compose**: Version 2.0+
- **Database**: Embedded H2 database is used by default
- **Resources**: Minimum 4GB RAM, 2 vCPUs, 10GB disk space
- **Recommended**: 8GB RAM, 4 vCPUs for production

### Database Requirements

Several external database engines are supported.

- **Postgresql**: 13 to 17
- **Microsoft SQL Server**: 2022 (MSSQL Server 16.0); 2019 (MSSQL Server 15.0); 2017 (MSSQL Server 14.0); 2016 (MSSQL Server 13.0); With bundled Microsoft JDBC driver
- **Oracle**: 23ai, 21C, 19C, XE Editions. Recommend to use the latest Oracle JDBC driver

> [!NOTE]
> The embedded H2 database is used by default. It is recommended for tests but not for production use.

### Verify Pre-requisites

```bash
# Check Docker installation
docker --version
docker-compose --version

# Verify system resources
free -h
df -h
```

## Deploy with docker-run

1. Create network

```bash
docker network create sonar
```

2. Create volumes

```bash
docker volume create --name sonarqube_data
docker volume create --name sonarqube_logs
docker volume create --name sonarqube_extensions
```

3. Run sonarqube container

```bash
docker run -d --name sonarqube \
    -p 9000:9000 \
    -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true \
    -v sonarqube_data:/opt/sonarqube/data \
    -v sonarqube_logs:/opt/sonarqube/logs \
    -v sonarqube_extensions:/opt/sonarqube/extensions \
    sonarqube:community
```
 
4. Verify SonarQube container

```bash
docker ps
docker logs sonarqube -f
```

## Run with docker-compose

1. Create Docker Compose File

Create `docker-compose.yml`:

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

2. Deploy SonarQube

```bash
# Start services
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f sonarqube
```

## Initial Configuration

1. Access SonarQube: http://your-server-ip:9000
2. Default credentials: admin/admin
3. Change admin password on first login
4. Generate authentication token for CI/CD integration

## Reference
Visit [SonarQube Docs](https://docs.sonarsource.com/sonarqube-community-build/)