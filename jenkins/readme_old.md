## Install Jenkins
Link https://www.jenkins.io/doc/book/installing/linux/

sudo apt update
sudo apt install openjdk-11-jre

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins

sdo systemctl status jenkins

to browse jenkins: http://192.168.56.3:8080/login?from=%2F


ssh into VM
sudo su
su - jenkins

Create keys: ssh-keygen -t ed25519 -C "footbar@example.com"
Keys will be into this directory: /var/lib/jenkins/.ssh/

-->share public key to github:
settings > deploy keys > add deploy keys
allow write access

-->Create  credentials to store private key
Dashboard > Manage jenkins > Credentials > System > Global credentials> add credentials

ssh-keyscan github.com >> ~/.ssh/known_hosts

---------------------build automatically---------------------------

/var/lib/jenkins/workspace/git