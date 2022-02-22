=================================================================================================
STEPS TO BE FOLLOWED-TO Implement AZURE AKS and Deploy SpringBoot Application along with POSTGRESDB
=================================================================================================
Scenario:
TO Implement alpha AKS and Deploy SpringBoot Application along with POSTGRESDB

Steps-to-be-followed:

# List of Pre-Requisites:
- Install Git
- Install JAVA JDK
- Install Gradle/Maven
- Install Docker

- AKS CLI Configuration
- Create AKS Cluster
- Install Kubernetes CLI - kubectl


# GIT INSTALLATION
sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt-get install git -y

# JAVA JDK INSTALLATION
sudo su
cd
sudo apt-get update -y
sudo apt install openjdk-11-jdk -y
java -version
export JAVA_HOME='/usr/lib/jvm/java-1.11.0-openjdk-amd64'

# MAVEN INSTALLATION
sudo su
cd
sudo apt-get update -y
sudo apt-get install unzip -y
wget https://dlcdn.apache.org/maven/maven-3/3.8.3/binaries/apache-maven-3.8.3-bin.zip
unzip apache-maven-3.8.3-bin
mv apache-maven-3.8.3/ maven/
mv maven/ /usr/local/bin/
sudo nano .bashrc
export M2_HOME=/usr/local/bin/maven/
export PATH=$PATH:/usr/local/bin/maven/bin/
mvn --version
(OR)
apt-get update -y
apt-get upgrade -y
apt-get install maven -y

# GRADLE INSTALLATION
wget https://services.gradle.org/distributions/gradle-7.2-bin.zip
unzip gradle-7.2-bin.zip
mv gradle-7.2  gradle/
mv gradle/  /usr/local/bin/
export PATH=$PATH:/usr/local/bin/gradle/bin
sudo nano .bashrc
export GRADLE_HOME=/usr/local/bin/gradle
export PATH=${GRADLE_HOME}/bin:${PATH}

# DOCKER INSTALLATION
sudo apt-get update -y
sudo apt-get install docker.io -y

# List of Commands to be used
- For Local Commits
	git add .
	git commit -m "<<message>>"
	git push -u origin master

- ON AZURE VM Instance
 	git clone https://github.com/ssadcloud/dkr-ion-springboot-app-aks.git
	cd dkr-ion-springboot-app-aks
	mvn clean package
	mvn package -DskipTests=true
	mvn clean package -DskipTests=true -Dstart-class=com.example.postgresdemo.PostgresDemoApplication

# DOCKER Commands to push Images to DockerHub:
	docker build -t ssadcloud/dion:latest .
	docker login
	docker push -t ssadcloud/dion:latest

# AZURE CLI INSTALLATION AND DOCKER IMAGE UPLOAD TO ACR
	sudo apt-get update -y
	apt install azure-cli -y
	az login
	az acr login -n ssadcloudacr

# DOCKER Commands to push Images to ACR:
	docker build -t ssadcloudacr.azurecr.io/ionsbapp:latest .
	docker push ssadcloudacr.azurecr.io/ionsbapp:latest

# CREATE AKS CLUSTER CREATION
	- Move to Azure Portal to Create AZURE KUBERNETES SERVICE(AKS)

#KUBECTL INSTALLATION
	curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
	sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
	kubectl version --client


# To Delete Images:
	docker ps -aq
	docker stop <<container-id>>
	docker rm <<container-id>>
	docker rmi -f <<image-id>>

# DEPLOYMENT TO alpha AKS/KOPS CLUSTER

	# Encode USERNAME and PASSWORD of Postgres using following commands:
	echo -n "postgresadmin" | base64 && echo -n "admin123" | base64

	# Create the Secret using kubectl apply:
	kubectl apply -f postgres-secrets.yml

	# Create PV and PVC for Postgres using yaml file:
	kubectl apply -f postgres-storage.yaml

	# Deploying Postgres with kubectl apply:
	kubectl apply -f postgres-deployment.yaml
	kubectl apply -f postgres-service.yaml

	# Create a config map with the hostname of Postgres
	kubectl create configmap hostname-config --from-literal=postgres_host=$(kubectl get svc postgres -o jsonpath="{.spec.clusterIP}")

	# Deploy Spring Application:
	kubectl apply -f springboot-deployment.yml
	kubectl apply -f springboot-service.yml

	# To Get Services ,Secrets:
	kubectl get secrets
	kubectl get configmaps
	kubectl get pv
	kubectl get pvc
	kubectl get deploy
	kubectl get pods
	kubectl get svc


	# Connection Command
	kubectl get pods -l app=postgres
	kubectl logs my-pod
		kubectl exec --stdin --tty <pod-reference> -- /bin/bash
	psql -h localhost -U postgres    # Use when role assicoated with PostgresSQL Denies the request
																	 # [psql: FATAL:  role "root" does not exist]

	## Postgress SQL Connection from Compute Engine VM

	#To Create database(if it doesnt exist)
	$ CREATE database alphauserdb;

	#To change database(if it doesnt exist)
	$ \c alphauserdb;

	#To create a table(if it doesnt exist)
	$ CREATE table user_data (id SERIAL PRIMARY KEY,firstName VARCHAR NOT NULL,lastName VARCHAR NOT NULL);

	#To describe a table
	\d table_name or \d+ table_name to describe a table.

	# To Insert Data:
	insert into public.user_data (id,first_name,last_name) values (1,'Karan','Singh');
	insert into public.user_data (id,first_name,last_name) values (2,'Aditi','Varma');
	insert into public.user_data (id,first_name,last_name) values (3,'Chaya','Devi');

	# TO list the Data
	SELECT * FROM user_data;

	# To List All Tables
	\dt

	#DELETE and DROPS
	$ DELETE FROM user_data WHERE id = 1;
	$ DROP TABLE <<table-name>>
	$ DROP DATABASE <<db-name>>

# To Delete Pods , Volumes, Secrets, configmaps:
	kubectl delete deploy spring-boot-postgres-sample postgres
	kubectl delete svc spring-boot-postgres-sample postgres
	kubectl delete pvc postgres-pv-claim
	kubectl delete pv postgres-pv-volume
	kubectl delete secrets postgres-secrets
	kubectl delete configmaps hostname-config

# To Delete Cluster
  - Move to AZURE PORTAL and delete cluster
