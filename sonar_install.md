step-by-step guide to setting up SonarQube 10.5.1 on a Linux system:

### 1. Install Prerequisites:
SonarQube requires Java to be installed. You can install OpenJDK 11 using the following commands:
```bash
sudo apt update
sudo apt install openjdk-11-jdk
```

### 2. Download and Extract SonarQube:
Download the SonarQube 10.5.1 distribution:
```bash
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.5.1.90531.zip
```
Extract the downloaded archive to the `/opt` directory:
```bash
sudo unzip sonarqube-10.5.1.90531.zip -d /opt
sudo mv /opt/sonarqube-10.5.1.90531 /opt/sonarqube
```

### 3. Configure SonarQube:
Navigate to the `conf` directory inside the SonarQube installation directory:
```bash
cd /opt/sonarqube/conf
```
Edit the `sonar.properties` file:
```bash
sudo nano sonar.properties
```
Add or modify the following properties:
```properties
sonar.jdbc.username=your_database_username
sonar.jdbc.password=your_database_password
sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube

sonar.web.host=127.0.0.1
sonar.web.port=9000

sonar.path.data=/opt/sonarqube/data
sonar.path.temp=/opt/sonarqube/temp

sonar.log.level=INFO
sonar.log.dir=/opt/sonarqube/logs

sonar.search.javaOpts=-Xmx512m -Xms512m
```
Replace `your_database_username` and `your_database_password` with your actual database credentials. Ensure the JDBC URL is correct for your database.

### 4. Create a Systemd Service Unit File:
Create a systemd service file named `sonarqube.service` in the `/etc/systemd/system/` directory:
```bash
sudo nano /etc/systemd/system/sonarqube.service
```
Add the following content:
```ini
[Unit]
Description=SonarQube service
After=network.target

[Service]
Type=forking
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
User=sonarqube
Group=sonarqube
Restart=always
LimitNOFILE=65536
LimitNPROC=8192

[Install]
WantedBy=multi-user.target
```

### 5. Create a SonarQube User:
Create a dedicated user for running SonarQube:
```bash
sudo useradd -r sonarqube
```

### 6. Set Permissions:
Set the ownership and permissions for the SonarQube directories:
```bash
sudo chown -R sonarqube:sonarqube /opt/sonarqube
sudo chmod -R 775 /opt/sonarqube
```

### 7. Start and Enable SonarQube Service:
Start the SonarQube service and enable it to start on system boot:
```bash
sudo systemctl start sonarqube
sudo systemctl enable sonarqube
```

### 8. Access SonarQube Web Interface:
SonarQube should now be running. Access the web interface using your browser by navigating to `http://your_server_ip_or_domain:9000`.
Follow the on-screen instructions to set up your SonarQube administrator account and configure your SonarQube instance.


Sure, here's the step-by-step guide to install SonarQube using Docker Compose:

### Execute Update and Upgrade Commands

```bash
sudo apt update
sudo apt upgrade
```

### Install Docker

```bash
sudo apt install docker.io
docker --version
```

### Install Docker Compose

```bash
sudo apt install docker-compose -y
```

### Create Docker Compose File

```bash
sudo vi docker-compose.yml
```

### Add Following Lines to Docker Compose File and Save

```yaml
version: "3"
services:
  sonarqube:
    image: sonarqube:community
    restart: unless-stopped
    depends_on:
      - db
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://db:5432/sonar
      SONAR_JDBC_USERNAME: sonar
      SONAR_JDBC_PASSWORD: sonar
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_logs:/opt/sonarqube/logs
    ports:
      - "9000:9000"
  db:
    image: postgres:12
    restart: unless-stopped
    environment:
      POSTGRES_USER: sonar
      POSTGRES_PASSWORD: sonar
    volumes:
      - postgresql:/var/lib/postgresql
      - postgresql_data:/var/lib/postgresql/data
volumes:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_logs:
  postgresql:
  postgresql_data:
```

### Run Docker Compose Command

```bash
sudo docker-compose up -d
```

### Monitor Logs Until SonarQube is Operational

```bash
sudo docker-compose logs --follow
```

This guide will help you install SonarQube using Docker Compose, from updating and upgrading your system to running SonarQube and monitoring its logs until it's operational.

