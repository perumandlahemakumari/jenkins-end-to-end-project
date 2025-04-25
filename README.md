
# Jenkins End-to-End CI/CD Project using AWS EC2

This project demonstrates a complete CI/CD pipeline setup using Jenkins, Maven, SonarQube, Nexus, and Tomcat across AWS EC2 instances. It pulls a Java web app from GitHub, builds it with Maven, performs code analysis with SonarQube, uploads artifacts to Nexus, and deploys the `.war` file to Tomcat.

---

## 🔧 Architecture & Setup

### EC2 Instances Required

| Service     | EC2 Type   | EBS Volume |
|-------------|------------|------------|
| Jenkins     | t2.micro   | Default    |
| Tomcat      | t2.micro   | Default    |
| SonarQube   | t2.medium  | 20 GB      |
| Nexus       | t2.medium  | 20 GB      |

- Launch all 4 instances using the **same PEM key**.
- SSH into each instance and install respective services.

---

## 🧩 Jenkins Plugin Requirements

Install the following plugins from **Manage Jenkins > Plugins**:

- **SonarQube Scanner**: For code analysis.
- **Nexus Artifact Uploader**: To upload artifacts to Nexus.
- **SSH Agent**: To securely send files to remote servers.

---

## ⚙️ Jenkins Pipeline Job Setup

Create a **Scripted Pipeline** job in Jenkins. Below are the stages for the pipeline.

---

### 📝 Jenkinsfile (Scripted Pipeline)

```groovy
node {

    stage("Clone Code") {
        git "https://github.com/perumandlahemakumari/jenkins-end-to-end-project.git"
    }

    stage("Build") {
        def mavenHome = tool name: "maven3", type: "maven"
        def mavenCMD = "${mavenHome}/bin/mvn"
        sh "${mavenCMD} clean package"
    }

    stage("SonarQube Code Analysis") {
        withSonarQubeEnv('mysonar') {
            def mavenHome = tool name: "maven3", type: "maven"
            def mavenCMD = "${mavenHome}/bin/mvn"
            sh "${mavenCMD} sonar:sonar"
        }
    }

    stage("Upload to Nexus") {
        nexusArtifactUploader(
            artifacts: [[
                artifactId: 'suneel',
                classifier: '',
                file: 'target/suneel.war',
                type: 'war'
            ]],
            credentialsId: 'nexus',
            groupId: 'devops',
            nexusUrl: '52.66.37.1:8081/',
            nexusVersion: 'nexus3',
            protocol: 'http',
            repository: 'Hema.repo',
            version: '1.0-SNAPSHOT'
        )
    }

    stage("Deploy to Tomcat") {
        sshagent(['tomcat']) {
            sh "scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/jenkins-project/target/suneel.war ec2-user@13.203.201.246:/opt/tomcat/webapps"
        }
    }

}
```

---

## 🧪 SonarQube Setup

1. Go to SonarQube dashboard.
2. Navigate to **My Account > Security**.
3. Generate a **token**.
4. In Jenkins:
   - Go to **Manage Jenkins > Configure System**.
   - Scroll to **SonarQube Servers**.
   - Add server with:
     - **Name**: `mysonar`
     - **URL**: `http://<SonarQube-Public-IP>:9000`
     - **Credentials**: Token-based authentication

---

## 📦 Nexus Repository Setup

1. Login to Nexus (URL: `http://<Nexus-IP>:8081`).
2. Create a new repository:
   - **Name**: `Hema.repo`
   - **Format**: `maven2`
   - **Type**: `hosted`
   - **Version Policy**: `Snapshot`
   - **Deployment Policy**: `Allow Redeploy`

---

## 🧑‍💻 Notes

- Replace the **IP addresses** in the Jenkinsfile with your EC2 instance public IPs.
- Make sure Java, Maven, Git, and Tomcat are properly installed and running on the respective instances.
- Jenkins workspace path (`/var/lib/jenkins/workspace/...`) may vary based on job name.

---

## 📚 Resources

- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [SonarQube Documentation](https://docs.sonarsource.com/)
- [Nexus Repository Docs](https://help.sonatype.com/)
- [Apache Tomcat Setup](https://tomcat.apache.org/)

---

## ✅ Project Flow Summary

1. **Code** is pulled from GitHub.
2. **Maven** builds and packages the code.
3. **SonarQube** scans and analyzes quality.
4. **Nexus** stores the `.war` artifact.
5. **Tomcat** deploys the final `.war` file.

---

Happy Deploying 🚀
