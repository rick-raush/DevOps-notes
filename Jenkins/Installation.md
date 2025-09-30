&nbsp;

## 🔹 Jenkins Installation on Amazon Linux 2023 (EC2)

1.  **Update system packages**
    
    ```bash
    sudo dnf update -y
    ```
    
2.  **Install Java (required by Jenkins)**
    
    ```bash
    sudo dnf install java-17-amazon-corretto -y
    java -version
    ```
    
    (Jenkins supports Java 11 & 17; we used 17.)
    
3.  **Add Jenkins repo & import key**
    
    ```bash
    sudo curl -o /etc/yum.repos.d/jenkins.repo \
        https://pkg.jenkins.io/redhat-stable/jenkins.repo
    sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
    ```
    
4.  **Install Jenkins**
    
    ```bash
    sudo dnf install jenkins -y
    ```
    
5.  **Enable & start Jenkins service**
    
    ```bash
    sudo systemctl enable jenkins
    sudo systemctl start jenkins
    sudo systemctl status jenkins
    ```
    
6.  **Open port 8080 in AWS Security Group**
    
    - Inbound Rule → Custom TCP → Port `8080` → Source: `0.0.0.0/0` (or your IP).
7.  **Access Jenkins in browser**
    
    ```
    http://<EC2-Public-IP>:8080
    ```
    
8.  **Unlock Jenkins**
    
    ```bash
    sudo cat /var/lib/jenkins/secrets/initialAdminPassword
    ```
    
    - Paste key into setup page.
        
    - Install **Suggested Plugins**.
        
    - Create first admin user.
        
    - Confirm Jenkins URL (use Public IP or Elastic IP).
        

* * *

&nbsp;