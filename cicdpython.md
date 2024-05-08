Sure, let's set up a Python project on a server step by step, including configuration files, commands, and tool configurations.

### Project 1: Python Project Setup

#### Step 1: Install Python and Virtual Environment

```bash
# Install Python (assuming Python 3.x)
sudo apt update
sudo apt install python3 python3-pip

# Install virtual environment
sudo apt install python3-venv
```

#### Step 2: Create a Virtual Environment for the Project

```bash
# Create a directory for the project
mkdir my_python_project
cd my_python_project

# Create a virtual environment
python3 -m venv venv

# Activate the virtual environment
source venv/bin/activate
```

#### Step 3: Install Dependencies and Set Up Project

```bash
# Install necessary dependencies (example: Flask)
pip install flask

# Create a Python file (example: app.py)
touch app.py
```

#### Step 4: Example Sample Code (app.py)

```python
# app.py

from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'

if __name__ == '__main__':
    app.run(debug=True)
```

#### Step 5: Install SonarQube Scanner for Python

```bash
# Download SonarQube Scanner for Python
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.6.2.2472-linux.zip

# Unzip the downloaded file
unzip sonar-scanner-cli-4.6.2.2472-linux.zip

# Move the extracted folder to a desired location (e.g., /opt)
sudo mv sonar-scanner-4.6.2.2472-linux /opt/sonar-scanner

# Add sonar-scanner bin directory to PATH
export PATH=$PATH:/opt/sonar-scanner/bin
```

#### Step 6: Configure SonarQube Scanner for Python

Create a `sonar-project.properties` file in your project directory:

```properties
# sonar-project.properties

sonar.projectKey=my_python_project
sonar.projectName=My Python Project
sonar.projectVersion=1.0
sonar.sources=.
sonar.sourceEncoding=UTF-8
sonar.language=python
sonar.python.coverage.reportPaths=coverage.xml
```
Sure, let's detail the steps to set up Jenkins and SonarQube using Docker, and then configure a Jenkins pipeline to build a Python project and perform SonarQube analysis.

### Step 7: Set Up Jenkins and SonarQube with Docker

#### 1. Create a Docker Compose file (`docker-compose.yml`):

```yaml
version: '3'

services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    ports:
      - "8080:8080"
    volumes:
      - jenkins_home:/var/jenkins_home
    restart: always

  sonarqube:
    image: sonarqube:community
    container_name: sonarqube
    ports:
      - "9000:9000"
    environment:
      - SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
    restart: always

volumes:
  jenkins_home:
  sonarqube_data:
  sonarqube_extensions:
```

#### 2. Run Docker Compose:

```bash
docker-compose up -d
```

Certainly! Let's delve into the detailed setup for configuring a Jenkins pipeline for a Python project.

### Step 8: Configure Jenkins Pipeline for Python Project

#### 1. Access Jenkins Web Interface:

Open your web browser and navigate to `http://localhost:8080`.

#### 2. Unlock Jenkins:

Upon accessing Jenkins for the first time, you will be prompted to unlock it. Follow the instructions displayed on the screen.

#### 3. Install Required Plugins:

Before setting up the pipeline, ensure the following plugins are installed:

- SonarQube Scanner: This plugin enables Jenkins to execute SonarQube analysis.
- Pipeline: This plugin allows defining pipelines using Jenkinsfile.

To install plugins:

- Go to Jenkins dashboard.
- Click on "Manage Jenkins" > "Manage Plugins".
- Navigate to the "Available" tab.
- Search for the plugins mentioned above and install them.

#### 4. Configure SonarQube Server:

1. Go to Jenkins dashboard.
2. Click on "Manage Jenkins" > "Configure System".
3. Scroll down to the "SonarQube servers" section.
4. Click on "Add SonarQube" and fill in the details:
   - Name: Provide a name for the SonarQube server.
   - Server URL: Enter the URL of your SonarQube server (e.g., `http://localhost:9000`).
   - Authentication Token: Generate an authentication token from your SonarQube server and paste it here.

#### 5. Create Jenkins Pipeline Job:

1. Click on "New Item" on the Jenkins dashboard.
2. Enter a name for the pipeline job (e.g., "Python Project").
3. Select "Pipeline" as the job type and click "OK".
4. Scroll down to the "Pipeline" section and define pipeline script from SCM (Source Control Management).
5. Choose your preferred SCM (e.g., Git, SVN) and provide the repository URL.
6. Configure additional settings such as credentials if required.

#### 6. Create Jenkinsfile:

In your project repository, create a file named `Jenkinsfile` with the following content:

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'python3 -m venv venv'
                sh 'source venv/bin/activate && pip install -r requirements.txt'
            }
        }
        stage('Test') {
            steps {
                sh 'source venv/bin/activate && pytest'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarQubeScanner'
                    withSonarQubeEnv('SonarQubeServer') {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }
    }
}
```

This Jenkinsfile defines the pipeline stages:
- **Build**: Sets up a virtual environment and installs project dependencies.
- **Test**: Runs tests using pytest (replace this with your preferred testing framework).
- **SonarQube Analysis**: Executes SonarQube analysis using the SonarQube scanner.

#### 7. Execute Jenkins Pipeline:

1. Save the pipeline job configuration.
2. Trigger the pipeline manually or set up triggers based on code changes in your SCM repository.
3. Jenkins will execute the pipeline, including SonarQube analysis, and display the results in the Jenkins dashboard.
