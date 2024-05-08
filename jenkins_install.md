
Below is the step-by-step guide to installing Jenkins on Debian/Ubuntu with necessary commands and configuration:

### Prerequisites

Ensure that your system meets the minimum hardware requirements:
- **RAM**: 256 MB minimum (4 GB+ recommended)
- **Disk Space**: 1 GB minimum (50 GB+ recommended)

Ensure Java is installed:
```bash
sudo apt update
sudo apt install openjdk-17-jre
java -version
```

### Step 1: Download and Add Jenkins Key

```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
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

### Step 3: Add Jenkins Repository

```bash
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
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
Direct Console Log Output to systemd-journald
Jenkins console log output will be directed to systemd-journald. You can troubleshoot using
```
journalctl -u jenkins.service.

````


### Post-Installation Steps

#### Verify Jenkins Installation

Check the status of Jenkins service:
```bash
sudo systemctl status jenkins
```

#### Access Jenkins Web Interface

Jenkins should now be running. Access the web interface using your browser by navigating to `http://your_server_ip_or_domain:8080`.


