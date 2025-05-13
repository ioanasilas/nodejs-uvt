-----------------------------------------------------------------
# Jenkins on Rocky Linux 9
-----------------------------------------------------------------

What is Jenkins?
Jenkins is a Java-based automation server with which we can define continuous integration and continuous delivery (CI/CD) jobs or pipelines. Continuous integration (CI) is a DevOps practice in which team members commit code changes regularly to GIT, after which a build is started and automated tests are run. Continuous delivery (CD) is also a DevOps practice through which code changes are deployed to production.
How will Jenkins help us?
We will automate the process of building and deploying docker images. The build will be done by Jenkins whenever a code change is committed to GitHub. Then the new image will be pushed to DockerHub automatically, also from Jeknins.

Installing Jenkins in a virtual machine running Rocky Linux 9

Confirm that the system repositories are working (after starting the virtual machine, the first time you run the sudo command, it will ask for the user's password. In my case, the user is a student, yours may be different):
sudo dnf repolist

Step 1: Install OpenJDK 21 on Rocky Linux 9
List the available JDK versions on Rocky Linux 9:

sudo dnf search java-*-openjdk
 
We will install the java-21-openjdk package:

sudo dnf -y install java-21-openjdk
 
At the end of the installation we will see the message Complete! And some details about the installed packages, java and the required dependencies:
We check the version of Java installed by default on our virtual machine:

java -version
 
Step 2: Add the Jenkins repository to Rocky Linux
The Jenkins team maintains a repository of RPM packages for Jenkins. We will add this repository, then install packages from it. 
Use the wget command to download the jenkins.repo file and place it in the correct directory:

sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
 
Also import the GPG key used to sign Jenkins packages:

sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
 
We check if the Jenkins repository is available locally:

sudo dnf repolist
 
We can see on the last line, first column, the repo id called "jenkins".

Step 3: Install Jenkins Server on Rocky Linux 9
Jenkins installation command:

sudo dnf -y install jenkins

Start the Jenkins service on Rocky Linux 9 (after running the start command, wait about 1 minute for Jenkins to start):

sudo systemctl start jenkins
 
We need to configure Jenkins to start with the virtual machine (system boot):

sudo systemctl enable jenkins
 
We check if Jenkins is really running on our virtual machine:

sudo systemctl status jenkins
 
We can see the green words: active (running). This tells us that Jenkins is running. If for some reason it doesn't start, you will see a red message that will probably start with the word "fail" or "failed".

Step 4: Configure Jenkins on Rocky Linux 9 from the web interface
After installing and starting the service, go to the web browser console at the URL:
http://localhost:8080
You should see a welcome page and instructions on how to obtain your initial administrator password:
 
To see the initial password, we need to execute the command on the file /var/lib/jenkins/secrets/initialAdminPassword (The initial password is different for each installation, when you install, you will have a different initial password):

sudo cat /var/lib/jenkins/secrets/initialAdminPassword
 
Copy and paste the password on the Jenkins page into the Administrator Password box and then click the Continue button:
 
After you have added the password, the following window will appear, where you select “Select plugins to install”.  

In the next step, we search for “git” and we will install only the “Git” plugin. Other plugins will appear, so check carefully that they have the simple name Git, as in the following picture: 
 
We are waiting for all the necessary plugins to finish installing.

After this step, we will configure our administrative user and click the Save and Continue button: 

After this step, the Instance Configuration step will appear, we don't change anything, just click Save and Finish.

Click on the Start using Jenkins button: 

We have set up Jenkins and can start using it. The following picture shows the first page that appears after we have finished the setup.

At this point in the Jenkins installation, we assume that you already have “podman” installed on the virtual machine (identical to docker, you should have it installed from the podman course) and we assume that it is set to start at startup. 
We assume that you have “git” installed on the virtual machine (you should have it installed from the git course).
We also assume that you have an account on Docker Hub https://hub.docker.com/
We will need this account to push the image we will build from Jenkins to Docker Hub.
We assume that you have created a repository in DockerHub with the name "mynodejs" to be able to push images created with Jenkins:
Select Create repository on the DockerHub web page: 
Then add “mynodejs” to the Repository Name and click Create, just like in the image below: 
Steps needed to prepare Jenkins to push or upload the image we will build

Login as root:

sudo su -

We need to find out the Jenkins user id with the command:

id jenkins
 
We note the number “uid=981” of the jenkins user, we need to use it in the next command. In our case, it is 981. In your case, it may be a different number, depending on what applications have been installed on this linux instance or what users you have created or have created these installed applications.
We run the following commands so that the jenkins user has enough permissions to build a docker image (podman).

loginctl enable-linger 981    (replace 981 with the uid of jenkins installed on your computer)

echo jenkins:10000:65536 >> /etc/subuid

echo jenkins:10000:65536 >> /etc/subgid

Jenkins automation

Exercise:

Create a new Job in Jenkins that will build a docker image, tag it, and push it to docker hub.

Step 1.

Add username and password credentials for Docker Hub into Jenkins Credentials
Before preparing the job, we need to add the Docker Hub user and password as variables in Jenkins. To do this, we need to go to Manage Jenkins:
 
Click on Credentials
Click on System
Click on Global credentials (unrestricted)
Click on the blue button on the right Add Credentials
Here we add credentials as in the following picture:
We leave Kind as it is – Username with password.
We add our own username for Docker Hub, our own password and ID called DOCKER_CREDENTIALS and the description “Docker username and password”: 
Click on Create.

Step 2.

Go to Jenkins default page and select “New item”
 
Add a name, select “Freestyle project” and press Ok 

On the General step, add a description, then scroll to the next step 

In the Source Code Management step, select Git for Repository URL and add https://github.com/bogdanobogeanu/nodejs-uvt.git

Below, in the same step, we have the Branches to build setting, leave it default, master 
In the Triggers step, select Pool SCM and in the Schedule add: H/5 * * * *
This schedule will check in git every 5 minutes if there is a change in the code and will start the job that we create now.

In the Environment step, check the box with Use secret text(s) or file(s), click Add and select Username and password (separated):
 
Then we add the variables as in the picture below – Username Variable: DOCKER_USER and Password Variable: DOCKER_PASSWORD. The credentials will be selected automatically because we have only one credential added, the Docker username and password: 
At the Build Steps step, select Add build step and then Execute Shell. 
Then we add the build command to the window:

podman build -t mynodejs:1.0.0 .

We add another Build step and then Execute Shell and add the commands to start and test the image that we built in the previous step.

podman run --name mynodejs -d -p 8081:8080 -it mynodejs:1.0.0

sleep 10

curl localhost:8081/nodejs

We add another Build step and another Execute shell where we run the stop commands of the mynodejs container and then the delete command of the mynodejs container.

podman stop mynodejs

podman rm mynodejs
 
We add a final Build step and an Execute shell to log in to Docker Hub and set the tags of the image built by Jenkins. With the tags set correctly, we will be able to push the image created to Docker Hub. Jenkins server has a set of predefined variables, including BUILD_NUMBER , which we will use to tag the docker image:

podman login -u $DOCKER_USER -p $DOCKER_PASSWORD docker.io 

podman tag mynodejs:1.0.0 bogdan1980b/mynodejs:latest

podman push bogdan1980b/mynodejs:latest

podman tag mynodejs:1.0.0 bogdan1980b/mynodejs:${BUILD_NUMBER}

podman push bogdan1980b/mynodejs:${BUILD_NUMBER}

Replace the user bogdan1980b in the commands with your personal DockerHub user.

Then select Save and then Build now. You will see on the left, at the bottom, the build that we started. 

It is possible that the started job failed due to a system bug or some error in the build steps. 
If all the steps were performed correctly, we will see that our job was completed successfully: 
In case of errors, we can access the job's console output where we can see the logs. We just need to select the job in the bottom left and then Console Output. 

If everything is ok and the job completed without errors, you should be able to find the image in DockerHub. 
Check your DockerHub account to make sure you have the latest version of the image.
trigger rebuild
random
something
