Practical 1 (04/08/2026)
Lab #1: Git and GitHub
1. Creating a new branch & merging it to the main branch.
2. Creating a merge conflict & resolving it.
All commands in this practical are run in Git Bash. Working folder: ~/Desktop/prac1
Setup: Configure Git
1) Set your Git identity (replace the placeholders with your own GitHub username and email).
Git Bash
git config --global user.name "<your-github-username>"
git config --global user.email "<your-email>"
2) Create the project folder and the first file.
Git Bash
cd ~/Desktop
mkdir prac1
cd prac1
git init
git branch -M main
nano app.py
Type this into nano, then save with Ctrl+O → Enter and exit with Ctrl+X:
app.py
print("This is an file")
3) Commit the file.
Git Bash
git add .
git commit -m "new file created"
4) Create an empty repository named prac1 on GitHub (https://github.com/new). Do not tick “Add a README” (an empty repo avoids a push rejection). Then connect it and push.
Git Bash
git remote add origin https://github.com/<your-github-username>/prac1.git
git push -u origin main
If you get “remote origin already exists”, run git remote remove origin first and then the add command again. A browser window opens for GitHub sign-in on the first push.
1. Creating a new branch & merging it to the main branch
1) Check the commit history.
Git Bash
git log
If the log fills the screen, press q to exit.
2) Create a new branch called login and switch to it.
Git Bash
git checkout -b login
nano login.py
Content of login.py (save: Ctrl+O → Enter, exit: Ctrl+X):
login.py
print("this is the new login feature branch")
3) Commit and push the login branch.
Git Bash
git add .
git commit -m "login file added"
git push -u origin login
On GitHub, open the prac1 repo → click the branch dropdown (main) → you will see both main and login.
4) Switch to main, merge login into it and push.
Git Bash
git checkout main
git merge login
git push -u origin main
Expected: “Fast-forward … login.py | 1 +”. GitHub now shows README.md (if any), app.py and login.py on main.
2. Creating a merge conflict & resolving it
1) Switch to the login branch and change app.py.
Git Bash
git checkout login
nano app.py
Replace the line in app.py with:
app.py (login branch)
print("This is an file on which i am creating a merge conflict")
Git Bash
git add .
git commit -m "creating merge conflict"
2) Switch to main and change the same line differently.
Git Bash
git checkout main
nano app.py
app.py (main branch)
print("This line was changed on the main branch")
Git Bash
git add .
git commit -m "changed the content"
3) Merge login into main – this creates the conflict.
Git Bash
git merge login
Expected: “CONFLICT (content): Merge conflict in app.py … Automatic merge failed”.
4) Open the file to resolve the conflict.
Git Bash
nano app.py
The file now looks like this:
app.py (with conflict markers)
<<<<<<< HEAD
print("This line was changed on the main branch")
=======
print("This is an file on which i am creating a merge conflict")
>>>>>>> login
Delete the markers and keep only the line you want, e.g.:
app.py (resolved)
print("This line was changed on the main branch")
5) Commit the resolution and push.
Git Bash
git add .
git commit -m "merge conflict solved"
git push -u origin main
On GitHub, app.py on main now shows the resolved line with the commit “merge conflict solved”.

Practical 2 (01/09/2026)
Lab #2: Setting Up Jenkins and Creating CI/CD Pipelines
Aim: To install and configure Jenkins on a local system and create an automated CI/CD pipeline that integrates with a version control system, enabling continuous integration and continuous deployment of a sample application.
Jenkins login used throughout: Username admin  |  Password pass@123
1) Install / update WSL. Open Command Prompt as Administrator:
Command Prompt (Admin)
wsl --status
wsl --update
2) Configure your GitHub account in Git and clone a repository.
Git Bash
git config --global user.name "<your-github-username>"
git config --global user.email "<your-email>"
cd ~/Downloads
git clone https://github.com/<your-github-username>/prac1.git
3) Install Java JDK 25 – Windows x64 MSI Installer.
Download page: https://www.oracle.com/java/technologies/downloads/#jdk25-windows → JDK 25 tab → Windows → x64 MSI Installer
Direct link: https://download.oracle.com/java/25/latest/jdk-25_windows-x64_bin.msi
Run the installer with default options.
4) Install Jenkins and set it up (Environment Setup: Jenkins Installation & Initial Configuration).
Download: https://www.jenkins.io/download/ → Windows (Stable / LTS installer). Run it.
Run service as LocalSystem, keep port 8080, and when asked for the Java home give your JDK path:
JDK path (version folder may differ)
C:\Program Files\Java\jdk-25.0.4.1\
5) Go to http://localhost:8080 and set up your account. Copy the initial administrator password from:
Password file
C:\ProgramData\Jenkins\.jenkins\secrets\initialAdminPassword
Or print it directly in an Admin Command Prompt:
Command Prompt (Admin)
type C:\ProgramData\Jenkins\.jenkins\secrets\initialAdminPassword
Paste it on the Unlock Jenkins page → Continue → Install suggested plugins.
On Create First Admin User fill in:
Create First Admin User
Username:  admin
Password:  pass@123
Confirm:   pass@123
Full name: Admin
Email:     <your-email>
Click Save and Continue → Save and Finish → Start using Jenkins.
6) Click the gear icon → Plugins → Available plugins → search Blue Ocean → tick it → Install.
Direct URL
http://localhost:8080/manage/pluginManager/available
7) After installation go back to the home page → top-right menu (☰) → Open Blue Ocean.
Direct URL
http://localhost:8080/blue
8) Click Create a new Pipeline.
If a Git error appears: Manage Jenkins (gear) → Tools → Git installations → set “Path to Git executable” to where git.exe is on your PC (find it with where git in Command Prompt). Default:
Path to Git executable
C:\Program Files\Git\bin\git.exe
9) Click Git → under Connect to a Git repository, enter your cloned folder path after file:/// (use / instead of \):
Repository URL
file:///C:/Users/Admin/Downloads/prac1
Replace Admin with your Windows user folder name if it is different.
Credentials
Username: admin
Password: pass@123
Click Create Pipeline.
10) Jenkins says “You don’t have any branches that contain a Jenkinsfile” → click Create Pipeline again.
11) Click + to add a stage → name it Build → + Add step → Windows Batch Script → enter in the Script box:
Build / Windows Batch Script
echo "Compiling"
If Save fails with “refusing to update checked out branch”, open Git Bash in the repo folder and run the command below once, then click Save again:
Git Bash (inside the repo folder)
git config receive.denyCurrentBranch updateInstead
Click Save → Description: Build Created → keep Commit to main selected (it shows master if that is your repo’s branch) → Save & run.
12) After saving you will see your build has failed! Stop the Jenkins service: press Win + R and type:
Run (Win + R)
services.msc
Find Jenkins → Right-click → Stop.
Or in an Admin Command Prompt:
Command Prompt (Admin)
net stop jenkins
13) Run Notepad as administrator (Start → type notepad → Run as administrator). Then File → Open, set the filter to All files, and paste this path:
File to open
C:\Program Files\Jenkins\jenkins.xml
14) Find the line that starts with <arguments> and replace the whole line with:
jenkins.xml
<arguments>-Xrs -Xmx256m -Dhudson.lifecycle=hudson.lifecycle.WindowsServiceLifecycle -Dhudson.plugins.git.GitSCM.ALLOW_LOCAL_CHECKOUT=true -jar "C:\Program Files\Jenkins\jenkins.war" --httpPort=8080 --webroot="%ProgramData%\Jenkins\war"</arguments>
The only change is the added -Dhudson.plugins.git.GitSCM.ALLOW_LOCAL_CHECKOUT=true flag, which lets Jenkins check out a local file:/// repository. If your line looks slightly different (other paths or memory value), keep your line and just insert that flag right after WindowsServiceLifecycle. Save the file (Ctrl+S).
15) Start Jenkins again: Win + R → services.msc → Jenkins → Right-click → Start. Or:
Command Prompt (Admin)
net start jenkins
Re-run the pipeline in Blue Ocean (↻ Replay) and see that it is now working (green tick).
16) Click the Edit (pencil) button → click + after Build → name the stage Test → + Add step → Windows Batch Script:
Test / Windows Batch Script
echo "running test :)"
Save → Description: Test created → keep Commit to main selected (it shows master if that is your repo’s branch) → Save & run.
17) Test is running successfully (Start → Build → Test → End all green).
18) Similarly, implement the Deploy stage using the same procedure:
Deploy / Windows Batch Script
echo "deploying finished"
Save → Description: Deploy created → Save & run. All stages (Build → Test → Deploy) turn green.
Resulting Jenkinsfile (Blue Ocean commits this to the repo automatically – for reference):
Jenkinsfile
pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        bat 'echo "Compiling"'
      }
    }
    stage('Test') {
      steps {
        bat 'echo "running test :)"'
      }
    }
    stage('Deploy') {
      steps {
        bat 'echo "deploying finished"'
      }
    }
  }
}

Practical 3 (06/10/2026)
Lab #3: Setting Up Docker, Clone and Dockerize a GitHub Repository
Replace the placeholders before running: <your-github-username> = your GitHub username, <repo-name> = your website’s GitHub repo name, <app-name> = a short name for your Docker image/container (lowercase, no spaces – Docker rejects capitals).
1) Install WSL through Command Prompt (Admin), then restart the PC when it finishes.
Command Prompt (Admin)
wsl --install
2) Download Docker Desktop and install it.
https://docs.docker.com/desktop/setup/install/windows-install/ → Docker Desktop for Windows – x86_64
Open Docker Desktop after installing; the whale icon in the taskbar should show Docker Desktop running.
3) Verify installation.
Command Prompt
docker --version
4) Run your first container.
Command Prompt
docker run hello-world
Expected: “Hello from Docker! This message shows that your installation appears to be working correctly.”
5) Open Git Bash, go to Downloads and clone the website repository (any static HTML/CSS/JS site repo).
Git Bash
cd ~/Downloads
git clone https://github.com/<your-github-username>/<repo-name>.git
cd <repo-name>
ls
You should see your website files. index.html must be in the top folder of the repo (not inside a sub-folder), because nginx serves that folder as the website.
6) Create a Dockerfile.
Git Bash
touch Dockerfile
echo "FROM nginx:alpine" > Dockerfile
echo "COPY . /usr/share/nginx/html" >> Dockerfile
echo "EXPOSE 80" >> Dockerfile
cat Dockerfile
Dockerfile contents:
Dockerfile
FROM nginx:alpine
COPY . /usr/share/nginx/html
EXPOSE 80
7) Build the Docker image.
Git Bash
docker build -t <app-name> .
8) After it finishes, verify the image.
Git Bash
docker images
Expected: <app-name>:latest is listed (along with hello-world:latest).
9) Run the Docker container.
Git Bash
docker run -d -p 8081:80 --name <app-name>-container <app-name>
docker ps
10) Open the Dockerized website in the browser:
Browser
http://localhost:8081/

Practical 4 (06/10/2026)
Lab #4: Continuous Integration and Deployment using Git, Jenkins and Docker
Jenkins login: Username admin  |  Password pass@123. Docker Desktop must be running. Use the same <repo-name> and <app-name> as in Practical 3.
1) Open Git Bash, go to Downloads and clone the repository (skip if already done in Practical 3).
Git Bash
cd ~/Downloads
git clone https://github.com/<your-github-username>/<repo-name>.git
cd <repo-name>
ls
2) Create a Dockerfile (skip if already created).
Git Bash
touch Dockerfile
echo "FROM nginx:alpine" > Dockerfile
echo "COPY . /usr/share/nginx/html" >> Dockerfile
echo "EXPOSE 80" >> Dockerfile
cat Dockerfile
Commit it so Jenkins can see it (do this even if you created the Dockerfile in Practical 3 – Jenkins only reads committed files):
Git Bash
git add Dockerfile
git commit -m "Dockerfile added"
3) Add Docker to Windows PATH and restart Jenkins.
Add Docker to Windows PATH – press Win + R and type:
Run (Win + R)
sysdm.cpl
Go to Advanced → Environment Variables. Under System variables: select Path → Edit → New → add:
Path entry
C:\Users\Admin\AppData\Local\Programs\DockerDesktop\resources\bin
If Docker was installed for all users the folder is C:\Program Files\Docker\Docker\resources\bin. Run where docker in Command Prompt to confirm.
Click OK → OK → OK.
Restart Jenkins – press Win + R and type:
Run (Win + R)
services.msc
Find Jenkins → Right-click → Restart. Or in an Admin Command Prompt:
Command Prompt (Admin)
net stop jenkins
net start jenkins
4) Create a new Jenkins pipeline: http://localhost:8080/blue → New Pipeline → Git → enter:
Repository URL
file:///C:/Users/Admin/Downloads/<repo-name>
Credentials
Username: admin
Password: pass@123
Click Create Pipeline → Create Pipeline again to open the editor (Agent: any).
5) Add a Build stage that builds the Docker image: + → name Build → Add step → Windows Batch Script:
Build / Windows Batch Script
where docker
docker --version
docker build -t <app-name> .
6) Click the + after Build → name the stage Run Docker Container → Windows Batch Script:
Run Docker Container / Windows Batch Script
docker rm -f <app-name>-container >nul 2>&1 & docker run -d -p 8081:80 --name <app-name>-container <app-name>
7) Next stage: Verify Docker Container → Windows Batch Script:
Verify Docker Container / Windows Batch Script
docker ps
If Save fails with “refusing to update checked out branch”, run git config receive.denyCurrentBranch updateInstead in Git Bash inside the repo folder, then Save again.
8) Run the pipeline: Save → Description Docker pipeline → keep Commit to main selected (it shows master if that is your repo’s branch) → Save & run. All stages (Build → Run Docker Container → Verify Docker Container) turn green.
9) Verify in the terminal and open the site.
Git Bash
docker ps
Browser
http://localhost:8081/
Resulting Jenkinsfile (for reference):
Jenkinsfile
pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        bat '''where docker
docker --version
docker build -t <app-name> .'''
      }
    }
    stage('Run Docker Container') {
      steps {
        bat 'docker rm -f <app-name>-container >nul 2>&1 & docker run -d -p 8081:80 --name <app-name>-container <app-name>'
      }
    }
    stage('Verify Docker Container') {
      steps {
        bat 'docker ps'
      }
    }
  }
}

Practical 5 (27/10/2026)
Lab #5: Deploy a simple application in a Kubernetes cluster using Pods, Deployments, and Services
Use the same <app-name> as in Practical 3 (it is the Docker image name, and here also the Deployment/Service name). Run everything in Command Prompt (Admin). Docker Desktop must be running and the <app-name> image you built in Practical 3 must exist (check with docker images).
1) Install Kubernetes (kubectl) and verify installation. Type Y when asked to accept the agreements, then close and reopen Command Prompt.
Command Prompt (Admin)
winget install -e --id Kubernetes.kubectl
Command Prompt (Admin)
kubectl version --client
2) Check if Docker Desktop is installed.
Command Prompt (Admin)
docker --version
3) Download and install Minikube and verify.
https://minikube.sigs.k8s.io/docs/start/ → Operating system Windows → Architecture x86-64 → Release type Stable → Installer type .exe download → download and run the installer.
Alternative from the command line: winget install -e --id Kubernetes.minikube
Command Prompt (Admin)
minikube version
4) Start the Kubernetes cluster and verify.
Command Prompt (Admin)
minikube start --driver=docker
kubectl get nodes
Expected: “Done! kubectl is now configured to use "minikube" cluster”, and node minikube is Ready.
5) Verify the Kubernetes cluster.
Command Prompt (Admin)
kubectl cluster-info
kubectl get nodes
6) Check Kubernetes components.
Command Prompt (Admin)
kubectl get pods -A
All kube-system pods (coredns, etcd, kube-apiserver, kube-proxy, kube-scheduler, …) should be Running.
7) Create a Kubernetes Deployment and verify.
Command Prompt (Admin)
kubectl create deployment <app-name> --image=<app-name>
kubectl get deployments
READY shows 0/1 at this point because the image is only on your PC, not inside Minikube yet.
8) Load the Docker image into Minikube and configure the Deployment to use the local image.
Command Prompt (Admin)
docker images
minikube image load <app-name>:latest
kubectl edit deployment <app-name>
The deployment opens in Notepad. Find this line:
Find
imagePullPolicy: Always
Change it to:
Replace with
imagePullPolicy: Never
Save (Ctrl+S) and close Notepad. Kubernetes restarts the pod automatically. Never = use the image already loaded into Minikube instead of trying to download it from Docker Hub (without this, the pod gets stuck at ErrImagePull).
9) Expose the Deployment using a Kubernetes Service.
Command Prompt (Admin)
kubectl expose deployment <app-name> --type=NodePort --port=80
kubectl get services
10) Verify the deployment in the terminal and open it in the browser.
Command Prompt (Admin)
kubectl get pods
minikube service <app-name>
The pod should be Running. minikube service opens the site in your browser (e.g. http://127.0.0.1:<port>). Keep this terminal open – with the Docker driver on Windows the tunnel stops when it is closed.

Practical 6 (05/11/2026)
Lab #6: Installation, Setup and Basic Automation using Ansible on Windows (WSL)
Ubuntu (WSL) sudo password: pass@123
If wsl --status shows no Ubuntu distribution, install it first with wsl --install -d Ubuntu. On first launch it asks you to create a Linux username and password (password used here: pass@123).
1) Check WSL status.
Command Prompt (Admin)
wsl --status
Expected: Default Distribution: Ubuntu, Default Version: 2
2) Open Ubuntu using WSL (the prompt changes to a Linux shell).
Command Prompt (Admin)
wsl
3) Update Ubuntu packages (password: pass@123).
Ubuntu (WSL)
sudo apt update
4) Install Ansible and verify installation.
Ubuntu (WSL)
sudo apt install ansible -y
ansible --version
5) Check Python version.
Ubuntu (WSL)
python3 --version
6) Create the Ansible playbook (paste the whole block at once).
Ubuntu (WSL)
cd ~
cat > prac_playbook.yml <<'EOF'
---
- name: Basic Ansible Automation
  hosts: localhost
  connection: local
  tasks:
    - name: Create a file
      ansible.builtin.file:
        path: "{{ ansible_env.HOME }}/ansible_file.txt"
        state: touch
 
    - name: Copy content to another file
      ansible.builtin.copy:
        content: "This file is created using Ansible automation."
        dest: "{{ ansible_env.HOME }}/ansible_message.txt"
EOF
7) Execute the Ansible playbook.
Ubuntu (WSL)
ansible-playbook prac_playbook.yml
Expected PLAY RECAP: localhost : ok=3 changed=2 unreachable=0 failed=0. The “No inventory was parsed” and deprecation warnings are normal and can be ignored.
8) Verify the created files.
Ubuntu (WSL)
ls -l ~/ansible_file.txt ~/ansible_message.txt
9) Display file content.
Ubuntu (WSL)
cat ~/ansible_message.txt
Expected: This file is created using Ansible automation.
