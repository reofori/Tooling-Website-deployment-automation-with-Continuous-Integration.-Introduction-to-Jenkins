# Tooling-Website-deployment-automation-with-Continuous-Integration.-Introduction-to-Jenkins
Tooling Website deployment automation with Continuous Integration. Introduction to Jenkins

In previous Project, we introduced horizontal scalability concept, which allow us to add new Web Servers to our Tooling Website and we have successfully deployed a set up with 2 Web Servers and also a Load Balancer to distribute traffic between them. If it is just two or three servers - it is not a big deal to configure them manually. Imagine that you would need to repeat the same task over and over again adding dozens or even hundreds of servers.

DevOps is about Agility, and speedy release of software and web solutions. One of the ways to guarantee fast and repeatable deployments is Automation of routine tasks.

In this project we are going to start automating part of our routine tasks with a free and open source automation server - Jenkins. It is one of the mostl popular CI/CD tools, it was created by a former Sun Microsystems developer Kohsuke Kawaguchi and the project originally had a named "Hudson".

Acording to Circle CI, Continuous integration (CI) is a software development strategy that increases the speed of development while ensuring the quality of the code that teams deploy. Developers continually commit code in small increments (at least daily, or even several times a day), which is then automatically built and tested before it is merged with the shared repository.

In our project we are going to utilize Jenkins CI capabilities to make sure that every change made to the source code in GitHub https://github.com/<yourname>/tooling will be automatically be updated to the Tooling Website.

## Task
Enhance the architecture prepared in Project 8 by adding a Jenkins server, configure a job to automatically deploy source codes changes from Git to NFS server.

#### Here is how your updated architecture will look upon completion of this project:

![image](https://github.com/user-attachments/assets/76a1ede9-a4bb-42c8-ba13-3f27dd4a3a9c)


## Step 1 - Install Jenkins server
1. Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"
![image](https://github.com/user-attachments/assets/2089d478-ddbb-47a4-bb99-eeb019223da1)


2. Install JDK (since Jenkins is a Java-based application).

```
sudo apt update
```
![image](https://github.com/user-attachments/assets/c6aeebfc-b06e-431a-9d02-901aa56fc066)



```
sudo apt install default-jdk-headless
```
![image](https://github.com/user-attachments/assets/e85f4225-3675-41cc-9a61-2c058d3795af)

3. Install Jenkins

```
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

Make sure Jenkins is up and running
```
sudo systemctl status jenkins
```
![image](https://github.com/user-attachments/assets/cd84b878-92df-407f-86ae-d94266d7c194)



4. By default Jenkins server uses TCP port 8080 - open it by creating a new Inbound Rule in your EC2 Security Group.
![image](https://github.com/user-attachments/assets/3dbd0d5c-2561-47fc-95c8-1e50445f3068)


5. Perform initial Jenkins setup.

From your browser access http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080

![image](https://github.com/user-attachments/assets/8c44b0a4-b9dc-4d1f-84d2-c467d87b2182)


We will be prompted to provide a default admin password

Retrieve it from your server:
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Then you will be asked which plugings to install - choose suggested plugins.
![image](https://github.com/user-attachments/assets/7eb44257-0ad4-48fe-8d52-2708211bca19)


![image](https://github.com/user-attachments/assets/3823ca8b-9fe0-4f0a-b2d7-63309f612366)



## Step 2 - Configure Jenkins to retrieve source codes from GitHub using Webhooks.

In this part, we will learn how to configure a simple Jenkins job/project (these two terms can be used interchangeably). This job will  be triggered by GitHub webhooks and will execute a 'build' task to retrieve codes from GitHub and store it locally on Jenkins server.

1. Enable webhooks in we GitHub repository settings.

Go to Forked Repo Tooling In own github and Inside the tooling repo need to do this.

![image](https://github.com/user-attachments/assets/42b95cf5-5b67-4b10-bf5d-81caf5fc6c04)


After Saving It Going to show

2. Go to Jenkins web console, click "New Item" and create a "Freestyle project"
![image](https://github.com/user-attachments/assets/d8ca4e02-1f52-4328-9106-29418d61b0ee)



To connect GitHub repository, We will need to provide its URL, we can copy from the repository itself
![image](https://github.com/user-attachments/assets/3468f451-3572-4754-9d16-d1a6c3d8f1ee)


In configuration of Jenkins freestyle project choose Git repository, provide there the link to Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

![image](https://github.com/user-attachments/assets/f7b1f3da-77c7-440c-a658-58d8ab32104f)

Save the configuration and let us try to run the build.


Got Error while building the jobs 
![image](https://github.com/user-attachments/assets/c1ef8d2e-0bf7-4106-ad62-a2f4df86d85a)
It is because of miss-configuration we need to edit configuration and have to change master to main branch. Need to select to trigger the pipeline.

![image](https://github.com/user-attachments/assets/12a61d88-ef30-4cac-ad08-3b91c3ab8989)

Save the configuration and let us try to run the again build.

![image](https://github.com/user-attachments/assets/5ff733de-40a0-4b5a-81e7-f19906f514e2)

It run successfully.

But this build does not produce anything and it runs only when we trigger it manually. Let us fix it.

3. Click "Configure" your job/project and add these two configurations
In Build trigger
![image](https://github.com/user-attachments/assets/d9cff3ea-b5bf-4035-b3e6-249f155c4d9f)
Save the configuration


4. Configure "Post-build Actions" to archive all the files - files resulted from a build are called "artifacts"
![image](https://github.com/user-attachments/assets/75ee0ae8-f1b9-4afb-aa89-3d3c8c3af4ec)

Under the form field for Files to archive, use ** to select all .
![image](https://github.com/user-attachments/assets/c136a336-5848-4ada-b049-6f76388d9176)

Save the configuration.

5. Now, go ahead and make some change in any file in we GitHub repository (e.g. README.MD file) and push the changes to the main branch. We will see that a new build has been launched automatically (by webhook) and we can see its results - artifacts, saved on Jenkins server.

![image](https://github.com/user-attachments/assets/12a45a7b-e8ec-4b77-a372-757c0ba3722b)

Inside artifact file we have;

![image](https://github.com/user-attachments/assets/ca9a6234-96d6-4ec6-be25-a360524b414c)



we have now configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as 'push' because the changes are being 'pushed' and files transfer is initiated by GitHub). There are also other methods: trigger one job (downstreadm) from another (upstream), poll GitHub periodically and others.


6. By default, the artifacts are stored on Jenkins server locally,

```
ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/
```
![image](https://github.com/user-attachments/assets/dcb8f356-2b1f-4e52-8012-390ed9ce3db2)


## Step 3 - Configure Jenkins to copy files to NFS server via SSH

Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to /mnt/apps directory.
Jenkins is a highly extendable application and there are 1400+ plugins available. We will need a plugin that is called "Publish Over SSH".

1. Install "Publish Over SSH" plugin.

On main dashboard select "Manage Jenkins" and choose "Manage Plugins" menu item.
On "Available" tab search for "Publish Over SSH" plugin and install it.

![image](https://github.com/user-attachments/assets/c29983f4-7371-40e0-bb2c-f606c21c2381)


2. Configure the job/project to copy artifacts over to NFS server.

- On main dashboard select "Manage Jenkins" and choose "Configure System" menu item.
- Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to our NFS server:

![image](https://github.com/user-attachments/assets/05ccb986-4acc-4aff-9d25-a5cbcf518c4c)


a. Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)

![image](https://github.com/user-attachments/assets/de11f43f-9fac-4a73-8fae-0dcdf1b2570a)


Paste the private here
![image](https://github.com/user-attachments/assets/255d4f8b-6d64-4116-9dc0-1e1abd696caf)


b. Arbitrary name
c. Hostname - can be private IP address of your NFS server.
d. Username - ec2-user (since NFS server is based on EC2 with RHEL 8).
e. Remote directory - /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server

![image](https://github.com/user-attachments/assets/fabbe780-0944-4458-95ca-d8eb0c99fb03)


Test the configuration and make sure the connection returns Success. Remember, that TCP port 22 on NFS server must be open to receive SSH connections.
![image](https://github.com/user-attachments/assets/5e828d93-74a7-4117-8fa0-72654ccfaeee)

3. Save the configuration, open  Jenkins job/project configuration page and add another one "Post-build Action"

![image](https://github.com/user-attachments/assets/e96d8e4d-e609-4a38-8d68-b19d2a1e8c7a)

Configure it to send all files probuced by the build into our previouslys define remote directory. In our case we want to copy all files and directories - so we use **. If you want to apply some particular pattern to define which files to send - use this syntax.

![image](https://github.com/user-attachments/assets/9c28a2e1-6857-4a32-bd00-edbceba38456)
Save this configuration and go ahead, change something in README.MD file in your GitHub Tooling repository.


Webhook will trigger a new job and in the "Console Output" of the job you will find something like this:
```
ssh: Transferred 24 file(s)
Finished: SUCCESS
```


If you find an error, change security settings of  NFS Server

```
sudo chown -R nobody:nobody /mnt
sudo chmod -R 777 /mnt
```

![image](https://github.com/user-attachments/assets/6d018e0c-a8cb-4e8e-825d-894c13a1c893)



To make sure that the files in /mnt/apps have been udated - connect via SSH/Putty to  NFS server and check README.MD file.

```
cat /mnt/apps/README.md
```
#### CONGRATULATION ✨✨✨✨🎊🎊 You have deployed your first automation tool on AWS with CI using Jenkins

#### Conclusion:

This project demonstrates the successful implementation of a Continuous Integration and Continuous Deployment (CI/CD) pipeline for a tooling website using Jenkins. By automating the deployment process, we have significantly improved the efficiency and reliability of our software delivery workflow.

Key achievements of this project include:
Setting up a Jenkins server to orchestrate the CI/CD pipeline.
Integrating version control (Git) with Jenkins for automated builds and deployments.
Implementing automated testing to ensure code quality and reduce the risk of bugs in production.
Configuring Jenkins to deploy the tooling website to a production environment automatically.
Demonstrating the benefits of CI/CD practices in terms of faster release cycles and improved collaboration among team members.
This project serves as an excellent introduction to Jenkins and CI/CD concepts, providing a solid foundation for further exploration of advanced DevOps practices. By adopting these automation techniques, we have taken a significant step towards more efficient and reliable software development and deployment processes.
