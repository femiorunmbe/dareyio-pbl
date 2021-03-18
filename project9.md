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
![image](https://user-images.githubusercontent.com/53876750/111560782-88ccd680-8793-11eb-82ef-188d245398c4.png)

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
![image](https://user-images.githubusercontent.com/53876750/111560814-a26e1e00-8793-11eb-99ec-6987ebb6b1be.png)

![image](https://user-images.githubusercontent.com/53876750/111560914-d21d2600-8793-11eb-8774-3aebf70eabfe.png)

8. Then you will be asked which plugings to install - choose suggested plugins.


Once plugins installation is done - create an admin user and you will get your Jenkins server address.

**Step 2 - Configure Jenkins to retrieve source codes from GitHub using Webhooks**
1. Enable webhooks in your GitHub repository settings
![image](https://user-images.githubusercontent.com/53876750/111560939-e4975f80-8793-11eb-808c-fc653ef1b553.png)

2. Go to Jenkins web console, click “New Item” and create a “Freestyle project”

3. To connect your GitHub repository, you will need to provide its URL, you can copy from the repository itself

4. In configuration of your Jenkins freestyle project choose Git repository, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

5. Click “Configure” your job/project and add these two configurations
* Configure triggering the job from GitHub webhook
* Configure “Post-build Actions” to archive all the files - files resulted from a build are called “artifacts”.

**Step 3 - Configure Jenkins to copy files to NFS server via SSH**
1. Install “Publish Over SSH” plugin.
On main dashboard select “Manage Jenkins” and choose “Manage Plugins” menu item.
On “Available” tab search for “Publish Over SSH” plugin and install it

2. Configure the job/project to copy artifacts over to NFS server.
On main dashboard select “Manage Jenkins” and choose “Configure System” menu item.
Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server
![image](https://user-images.githubusercontent.com/53876750/111561243-7010f080-8794-11eb-9852-32fafe65201f.png)

3. Save the configuration, open your Jenkins job/project configuration page and add another one “Post-build Action”
  Configure it to send all files probuced by the build into our previouslys define remote directory. In our case we want to copy all files and directories - so we use **
  ![image](https://user-images.githubusercontent.com/53876750/111561369-a0588f00-8794-11eb-84a3-2c09b6ab8f2e.png)

4. Save this configuration and go ahead, change something in README.MD file in your GitHub Tooling repository.

Webhook will trigger a new job and in the “Console Output” of the job you will find something like this:
SSH: Transferred 25 file(s)
Finished: SUCCESS
![image](https://user-images.githubusercontent.com/53876750/111561449-bf572100-8794-11eb-92f1-166fd843c28f.png)

![image](https://user-images.githubusercontent.com/53876750/111561464-c716c580-8794-11eb-95d7-984899bee516.png)

