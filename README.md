# Nginx in Docker
### Prerequesites
- Docker Installed
### Step1: Configure a domain
- Configure a domain name for the IP. This IP should be attached to the VM. 
### Step2: Create a docker network
```
docker network create app-network
```
### Step2: Run a simple react application
Run the following command to run a simple react application, where we are about to expose this react application through nginx.
```
docker run -d -p 3000:80 --name react --network app-network --restart always darwinwilmut/react-docker
```
