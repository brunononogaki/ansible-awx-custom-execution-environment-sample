# Ansible AWX - Sample of a Custom Execution Environment
This is a step-by-step guide for dummies to build a custom Ansible Execution Environment, push to quay.io registry, pull in Ansible AWX and run.
This example uses CentOS 9 to build the image.

## 1. Preparing the environment

### 1.1 Install Docker

#### 1.1.1 Install procedure

```
sudo yum update
sudo yum install -y yum-utils
sudo yum config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io
````

#### 1.1.2 Start Docker

```
sudo systemctl start docker
sudo usermod -aG docker $USER
sudo systemctl enable /usr/lib/systemd/system/docker.service
sudo systemctl stop firewalld
sudo systemctl disable firewalld
````
You will need to logout and login again, so your user can run docker comands. Try running:
```
docker ps -a
````

#### 1.1.3 Start Docker-Compose (optional)

```
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
````

### 1.2 Install git
```
sudo yum install git
```

### 1.3 Install ansible-builder
```
pip install ansible-builder
```
Note: You might run this inside a Python Venv if you want


## 2. Clone this repository
```
git clone https://github.com/brunononogaki/ansible-awx-custom-execution-environment-sample
````

## 3. Edit the files
You can edit the files as needed:
+ execution-environment.yml: This is the main manifest file, and we are using the Red Hat **ee-minimal-rhel9:1.0.0-720** as Base Image for our Execution Environment. You can find more images [here](https://catalog.redhat.com/search?searchType=containers&application_categories_list=Automation&p=1&build_categories_list=Automation%20execution%20environment).

+ requirements.txt: Add here the Python Libs dependencies that you want to be installed in your execution environment

+ requirements.yml: Add here the Ansible Collections dependencies that you want to be installed in your execution environment

## 4. Login in RedHat registry
Before you build this image, you have to create a free account at redhat.com and you will need to login docker in Red Hat registry, so it can pull the base image.
```
docker login registry.redhat.io
```

## 5. Build the image locally
Finally, we are ready to build the image in your Local machine:
```
ansible-builder build -t network-custom-ee -v 3
```
It will take a while for it to build, so relax and wait. If you see any error, you might take a look, maybe it is an invalid library you declared in requirements files, or something like that. At the end, you can see you image by running:
```
docker image ls
```

## 6. Push the image to a registry
For this demo, we will push the image to [quay.io](https://quay.io) registry. You can create a free 30-days trial account.

### 6.1 Tag the image
First lets create a tag for the image. Run **docker image ls** again to confirm the name of your image:
```
REPOSITORY               TAG          IMAGE ID       CREATED          SIZE
network-custom-ee        latest       541f93c37af3   30 minutes ago   1.34GB
```

Now run the command below to create a tag (replace to your username in quay.io)
```
docker tag network-custom-ee quay.io/<your-quay.io-username>/network-custom-ee
```

Run **docker image ls** again to see if the tag was created:
```
REPOSITORY                          TAG          IMAGE ID       CREATED          SIZE
network-custom-ee                   latest       541f93c37af3   34 minutes ago   1.34GB
quay.io/xxxxxxx/network-custom-ee   latest       541f93c37af3   34 minutes ago   1.34GB
```

### 6.2 Login in Quay.io
Now login your docker in quay.io
```
docker login quay.io
```

### 6.3 Push the image
Run the command below, replacing your username, to push the image to the registry:
```
docker push quay.io/<your-quay.io-username>/network-custom-ee
```
After finished, you should see your image in your [repository](https://quay.io/repository) in quay.io.

### 6.4 Make the image public (optional)
Optionally, you can login to [quay.io](https://quay.io/repository), click in your image, go to Settings and click **Make public**. I recommend you do so, so you don't need to configure credentials in AWX, and also, private repositories in quay.io required a paid plan.

## 7. Add the image in Ansible AWX
Now it is time to configure AWX. If you need a help to install it, you can follow this [guide](https://github.com/brunononogaki/ansible-awx-install-with-operator).

### 7.1 Create Execution Environment
In Ansible AWX, go to Administration -> Execution Environments -> Add and fill:
+ Name: network-custom-ee
+ Image: quay.io/<your-quay.io-username>/network-custom-ee

### 7.2 Attach the Execution Environment to a Template
Go to your Job Template, Edit, and add this Execution Environment

### 7.3 Launch the template
In your first run it might take a while for AWX to pull the new image.