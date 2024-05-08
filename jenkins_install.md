
Below is the step-by-step guide to installing Jenkins on Debian/Ubuntu with necessary commands and configuration:

### Prerequisites

Ensure that your system meets the minimum hardware requirements:
- **RAM**: 256 MB minimum (4 GB+ recommended)
- **Disk Space**: 1 GB minimum (50 GB+ recommended)

Ensure Java is installed:
```bash
sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version
```

### Step 2: Configure Firewall (if applicable)

If you have a firewall installed, you must add Jenkins as an exception. Replace `YOURPORT` with the port you want to use (e.g., 8080).

```bash
YOURPORT=8080
PERM="--permanent"
SERV="$PERM --service=jenkins"

sudo firewall-cmd $PERM --new-service=jenkins
sudo firewall-cmd $SERV --set-short="Jenkins ports"
sudo firewall-cmd $SERV --set-description="Jenkins port exceptions"
sudo firewall-cmd $SERV --add-port=$YOURPORT/tcp
sudo firewall-cmd $PERM --add-service=jenkins
sudo firewall-cmd --zone=public --add-service=http --permanent
sudo firewall-cmd --reload
```

### Step 3: Download and Add Jenkins Key and Jenkins Repository

```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

### Step 4: Install Jenkins

```bash
sudo apt-get update
sudo apt-get install jenkins
```
   - Follow the prompts to complete the installation process.

### Step 5: **Configure Jenkins as a service**:
   - Jenkins may not start automatically after installation. To configure it as a service, create a systemd service file. Create or edit the file `/etc/systemd/system/jenkins.service` using a text editor (e.g., `nano` or `vim`), and add the following content:
     ```
     [Unit]
     Description=Jenkins Automation Server
     Wants=network-online.target
     After=network-online.target

     [Service]
     Type=simple
     User=jenkins
     Group=jenkins
     Environment="JENKINS_HOME=/var/lib/jenkins"
     ExecStart=/usr/bin/java -jar /usr/share/jenkins/jenkins.war
     Restart=always
     StandardOutput=syslog
     StandardError=syslog
     SyslogIdentifier=jenkins

     [Install]
     WantedBy=multi-user.target
     ```
   - Save the file and close the text editor.

6. **Create a Jenkins user**:
   - Create a dedicated user for running Jenkins. You can do this with the following command:
     ```
     sudo useradd -m -s /bin/bash jenkins
     or
     sudo useradd -r jenkins -s /usr/sbin/nologin
     ```

7. **Set Jenkins home directory**:
   - Jenkins requires a home directory for storing configuration and data. By default, this is set to `/var/lib/jenkins`, but you can customize it by setting the `JENKINS_HOME` environment variable in the systemd service file.

8. **Start and enable Jenkins service**:
   - Start the Jenkins service and enable it to start on system boot with the following commands:
     ```
     sudo systemctl start jenkins
     sudo systemctl enable jenkins
     ```



### Post-Installation Steps
Direct Console Log Output to systemd-journald
Jenkins console log output will be directed to systemd-journald. You can troubleshoot using
```
journalctl -u jenkins.service.

````
#### Verify Jenkins Installation

Check the status of Jenkins service:
```bash
sudo systemctl status jenkins
```

#### Access Jenkins Web Interface

Jenkins should now be running. Access the web interface using your browser by navigating to `http://your_server_ip_or_domain:8080`.
===========================================================================

Sure, below is a step-by-step guide to install Jenkins using Docker Compose:

### 1. Create a directory for Jenkins and navigate into it:

```bash
mkdir jenkins && cd jenkins
```

### 2. Create a `docker-compose.yml` file:

```bash
touch docker-compose.yml
```

### 3. Open `docker-compose.yml` file in a text editor (e.g., `nano` or `vi`):

```bash
nano docker-compose.yml
```

### 4. Add the following content to `docker-compose.yml`:

```yaml
version: '3'

services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins_home:/var/jenkins_home
    restart: always
    environment:
      TZ: "Asia/Kolkata"  # Specify your timezone here
```

or 
```
version: '3'

services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins_home:/var/jenkins_home
    restart: always

  dind:
    image: docker:dind
    container_name: jenkins-docker
    privileged: true
    environment:
      DOCKER_TLS_CERTDIR: "/certs"
      TZ: "Asia/Kolkata" 
    volumes:
      - jenkins-docker-certs:/certs/client
    ports:
      - "2376:2376"
    command: ["--storage-driver", "overlay2"]

volumes:
  jenkins_home:
  jenkins-docker-certs:
```

Replace `"Your/Timezone"` with your actual timezone (e.g., `"America/New_York"`).

### 5. Save and close the file.

For `nano`, press `Ctrl + X`, then `Y`, and then press `Enter`.

### 6. Run Jenkins using Docker Compose:

```bash
docker-compose up -d
```

### 7. Access Jenkins Web Interface:

Jenkins should now be running. Access the web interface using your browser by navigating to `http://localhost:8080`.

### 8. Unlock Jenkins:

Follow the on-screen instructions to unlock Jenkins. You can find the initial administrator password in the logs or inside the Jenkins container.

### 9. Install Recommended Plugins:

After unlocking Jenkins, you will be prompted to install recommended plugins. Proceed with the installation.

### 10. Set Up Admin User:

Create an administrator user and complete the setup wizard.

### 11. Start Using Jenkins:

You can now start using Jenkins for your CI/CD pipelines.

### 12. Stop and Remove Jenkins Container (Optional):

To stop and remove the Jenkins container while preserving Jenkins data, run:

```bash
docker-compose down
```

=====================================================
**DOCKER**
Open up a terminal window.

Create a bridge network in Docker using the following docker network create command:
```
docker network create jenkins
```
In order to execute Docker commands inside Jenkins nodes, download and run the docker:dind Docker image using the following docker run command:
```
docker run --name jenkins-docker --rm --detach \
  --privileged --network jenkins --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 2376:2376 \
  docker:dind --storage-driver overlay2
```

```
docker run --name jenkins-blueocean --restart=on-failure --detach \
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  myjenkins-blueocean:2.440.3-1
```
Customize the official Jenkins Docker image, by executing the following two steps:

Create a Dockerfile with the following content:
```
FROM jenkins/jenkins:2.440.3-jdk17
USER root
RUN apt-get update && apt-get install -y lsb-release
RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
  https://download.docker.com/linux/debian/gpg
RUN echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
RUN apt-get update && apt-get install -y docker-ce-cli
USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean docker-workflow"
```
Build a new docker image from this Dockerfile, and assign the image a meaningful name, such as "myjenkins-blueocean:2.440.3-1":
```
docker build -t myjenkins-blueocean:2.440.3-1 .
```
If you have not yet downloaded the official Jenkins Docker image, the above process automatically downloads it for you.
