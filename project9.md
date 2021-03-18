# Tooling Website deployment automation with Continuous Integration.
### Project 9: Introduction to Jenkins

**Step 1 - Install Jenkins server**
1. Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it “Jenkins” 
  ![image](https://user-images.githubusercontent.com/53876750/111547840-13084100-877a-11eb-914f-8d8512075874.png)

2. Install JDK (since Jenkins is a Java-based application)
```
sudo apt update

sudo apt install default-jdk-headless
```

3. Install Jenkins
```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -

sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'

sudo apt update

sudo apt-get install jenkins
```

4. Make sure Jenkins is up and running
```
sudo systemctl status jenkins
```

5. By default Jenkins server uses TCP port 8080 - open it by creating a new Inbound Rule in your EC2 Security Group

6. Perform initial Jenkins setup.
From your browser access 
```
http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080
```

7. You will be prompted to provide a default admin password
Retrieve it from your server:
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
8. Then you will be asked which plugings to install - choose suggested plugins.


Once plugins installation is done - create an admin user and you will get your Jenkins server address.

**Step 2 - Configure Jenkins to retrieve source codes from GitHub using Webhooks**
1. Enable webhooks in your GitHub repository settings

2. Go to Jenkins web console, click “New Item” and create a “Freestyle project”

