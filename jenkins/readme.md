### Install Jenkins
Link https://www.jenkins.io/doc/book/installing/linux/

- Use the commands to install Jenkins:
```
sudo apt update
sudo apt install openjdk-11-jre

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

- To browse Jenkins, use your VM IP and port 8080: `http://192.168.56.3:8080/`
- You need to ssh into your VM (vagrant ssh) and use this command to retrieve the initial password: `sudo cat /var/lib/jenkins/secrets/initialAdminPassword` and use the password to complete the Jenkins configuration

- Command to start/stop/restart and check status of Jenkins service:
```
sudo systemctl stop jenkins   ## to check status

sudo systemctl start jenkins    ## to start jenkins

sudo systemctl restart jenkins    ## to restart jenkins

sudo systemctl status jenkins    ## to status check
```

- Notes about Jenkins:
  - jenkins home directory is /var/lib/jenkins
  - job workspace directory is /var/lib/jenkins/workspace/<job-name>

## Activities
### Freestyle job practice:
- Simple Freestyle job: Create a freestyle job named `freestyle-job`
  - In `Build Steps` choose `Execute shell` and write any script. for example: `echo "this is my first jenkins job"`. Then save and build this job

- Simple Freestyle job with Git Repo: Create another freestyle job named `freestyle-job-with-git`
  - Use a public github repository, so that we don't need to use any credential to authenticate the repository
  - Familia with different options in the job:
    - Source Code Management -> Git,
    - Build Triggers -> Poll SCM,
    - Build Environment -> Delete workspace before build starts,
    - Build Steps -> Execute shell
    - use the below script to be familiar with workspace, git repository files in workspace, executing script, etc.
```
echo "workspace directory:"
pwd
echo "list of files in workspace directory"
ls -rlth
echo "executing tests..."
bash tests.sh
echo "list of files after executing tests"
ls -rlth
```

### Authenticate with Private repo using SSH keys
* create ssh keys (private & public key) for jenkins user by using the following commands:
  - ssh into VM: `vagrant ssh`
  - switch to jenkins user: `sudo su - jenkins`
  - create keys: `ssh-keygen -t ed25519 -C "foobar@example.com"`
  - keys will be into this directory and list the keys: `ls -rlth /var/lib/jenkins/.ssh/`
  - go to the directory where you the keys: `cd .ssh`

* share your public key to github - creating a deploy key:
  - Browse your git repository
  - Go to Settings -> Deploy Keys -> Add Deploy key
  - Enter a title
  - Copy public key `id_ed25519.pub` from your VM and paste there
  - Allow write access and save

* Create Jenkins credentials to store private key `id_ed25519`
  - Jenkins home -> Manage Jenkins -> Credentials

* Pre-populate the SSH keys for github.com into known_hosts file: `ssh-keyscan github.com >> ~/.ssh/known_hosts`

* Create a freestyle job and use a private repository to test the SSH integration you did above:
  - use ssh url of the repository and the credential you created above
  - use your mobile network to build your job, ssh is blocked into Cester network

### Git Webhook to auto trigger Jenkins job on git event
- Create an Vitual machine into AWS cloud with Public IP address and configure security group properly
- SSH into the VM and install Jenkins into that VM and configure it
- Go to git repository `Settings -> Webhook -> Add webhook` and use the URL `http://<VM-Public-IP>:8080/github-webhook/`
- Create a freestyle job `webhook-test`, use your git repository to connect the job, check `Build Triggers -> GitHub hook trigger for GITScm polling` and Save
- Push a commit to your repo and that commit should trigger a build


### Pipeline Job
- Pipeline: Devs -> commit push to github repo -> webhook triggers build job -> test job -> deploy job

- Pipeline using freestyle job:
  - Create 3 freestyle jobs named `build`, `test` and `deploy` and make them trigger automatically using Post-build Actions
  - Manually trigger `build` job. both test and deploy jobs should be triggered as well

- Pipeline using Pipeline job (Using Script):
  - https://www.jenkins.io/doc/book/pipeline/jenkinsfile/
```
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building..'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}
```
- To learn more on syntax: https://www.jenkins.io/doc/book/pipeline/syntax/

- Parallel stages (https://www.jenkins.io/doc/book/pipeline/syntax/#parallel): (modified script)
```
pipeline {
    agent any
    stages {
        stage('Non-Parallel Stage') {
            steps {
                echo 'This stage will be executed first.'
            }
        }
        stage('Parallel Stage') {
            failFast true
            parallel {
                stage('Branch A') {
                    agent any
                    steps {
                        echo "On Branch A"
                    }
                }
                stage('Branch B') {
                    agent any
                    steps {
                        echo "On Branch B"
                    }
                }
                stage('Branch C') {
                    agent any
                    stages {
                        stage('Nested 1') {
                            steps {
                                echo "In stage Nested 1 within Branch C"
                            }
                        }
                        stage('Nested 2') {
                            steps {
                                echo "In stage Nested 2 within Branch C"
                            }
                        }
                    }
                }
            }
        }
    }
}
```
  - Install Blueocean plugin to see parallel stages


- Pipeline using Pipeline job (Using Jenkinsfile):
  - Create `Jenkinsfile` into your git repository and paste your script there
  - Create a pipeline job using git SCM

### Sample Maven Build Pipeline
- Create a pipeline job and use this git repo in Pipeline definition as SCM: https://github.com/jenkins-docs/simple-java-maven-app/tree/master
- Use `jenkins/Jenkinsfile` as Script path (Jenkinfile location)
- This job uses docker to build image, so install docker engine into your VM (find installation in internet)
- The `jenkins` user needs permission to use docker commands. So use execute these commands inside your VM:  `sudo usermod -a -G docker jenkins` or `sudo chmod 666 /var/run/docker.sock`
- Install the plugins: docker plugin, docker pipeline, docker-build-step, maven integration plugin
- Once every is done, build the job
