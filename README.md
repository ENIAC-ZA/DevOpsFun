# Janco's DevOps Test:

## Included in the readme is all the steps and filenames required to reproduce the tasks.


## Task 1:

### By far the simplest of the tasks, this was done mostly through a GUI, and verification of all the services was done through the command line.
Steps:

#### Docker:
- Install docker from - https://docs.docker.com/desktop/install/windows-install/

- After installation enable WSL 2 instead of Hyper-V/nothing.

- Install kubernetes from within docker settings menu.


#### Kubernetes:
- After kubernetes is installed I had access to the `kubectl` command set and verified that everything was working,  some of the commands I used were:

```
kubectl get nodes
kubectl get ns
kubectl proxy
```



- User creation/token generation, and installing k8-dashboard:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.1/aio/deploy/recommended.yaml
kubectl apply -f C:\Users\dayzd\.kube\dashboard-adminuser.yaml 
kubectl -n kubernetes-dashboard create token admin-user --duration 300h0m0s
```



- To deploy the 'simple-web' container, I created `simple-web-dep.yaml` and simply applied it to kubernetes:
```
kubectl apply -f C:\Users\dayzd\.kube\simple-web-dep.yaml
```

## Task 2:

### Setting up Jenkins <-> Kubernetes was a fun challenge and introduced me to some new concepts. The steps I took to get everything working were:
#### Kubernetes:
- Create and apply a jenkins-dep.yaml to kubernetes:
```
kubectl apply -f C:\Users\dayzd\.kube\jenkins-dep.yaml
```
- After deploying jenkins, it was accessible from localhost:8080 I used `docker cp` to get the initial password, I saw that it's available from log files as well.
```
docker ps (get jenkins container ID)
docker cp 8dacbc0cc5e4:/var/jenkins_home/secrets/initialAdminPassword C:\Users\dayzd\Downloads
```
#### Jenkins:
- After the initial setup of jenkins + user creation I added the kubernetes plugin and restarted.

- Setting up kubernetes/jenkins agents I left most values on default, the only values I had to change were:
```
Credentials --> kubernetes login-secret
WebSocket   --> True
Jenkins URL --> http://jenkins8080

Podlabel
|-Key   ---> jenkins
|-Value ---> executor
```
<img src="https://i.imgur.com/NwPyqfk.png" height="300" />
<img src="https://i.imgur.com/V5iMLkE.png" height="250" />

- I also made sure to decrease the jenkins default agents to 0.

Jenkins was now ready for deployments.

edit 01/11/2022 - I added the kubernetes CLI addon to be able to deploy to kubernetes.

## Task 3:
### No More Forks :)

Using my newfound knowledge, I was able to figure out where I went wrong first time round. Here are the steps I took to correctly deploy the e-ShopOnWeb application.
First I had to clone the repo and run docker-compose to create the images. Afterwards I pushed the images to dockerhub, and started working on the deployment files.
```
docker compose build
```
```
docker ps -a 
```
```
docker tag eshoppublicapi eniacza/eshopapi:latest
docker tag eshopwebmvc eniacza/eshopweb:latest
docker tag mcr.microsoft.com/azure-sql-edge eniacza/eshopsql:latest
```
```
docker push eniacza/eshopapi:latest
docker push eniacza/eshopweb:latest
docker push eniacza/eshopsql:latest
```

Next I had to create 3 deployment files for the 3 images, I had to add configMaps to the API and the Webapp to point them to eachother/database. Just to make sure the deployments work I manually imported and deleted them from kubernetes before moving on to jenkins.
```
kubectl apply -f C:\Users\dayzd\.kube\e-sql-dep.yaml
kubectl apply -f C:\Users\dayzd\.kube\e-web-dep.yaml
kubectl apply -f C:\Users\dayzd\.kube\e-api-dep.yaml
```


## Question 1:

This is the repo :D



## Question 2:
```
{
 "Janco":{
  
  "Info": {
    "Alive": true,
    "NeedJob": true,
    "Country": "South Africa",
    "City": "Pretoria",
    "Age": 25 
  },
  
  "Hobbies": {
    "Gaming": true,
    "Learning": true,
    "Relaxing": true,
    "Hang-Out": true
  },
  
  "Skills": {
    "Coding": "Decent",
    "Talking": "Average",
    "Driving": "Decent",
    "Flying": "Warning"
    }
  }
 }
} 

```


## Question 3:

#### By far the thing I found most interesting was learning how to build and use CI/CD pipelines, I understood the concept. But seeing just how simple it can be blew my mind away, I immediately started thinking about projects I could apply this to and I just fell in love. Going forward, anything that I put into some sort of release environment will absolutely run through Jenkins + Kube. 



## Question 4:

#### First off let me say that If I had to list all the problems I encountered I could write a book, this was my first time using Kubernetes and Jenkins so there was a lot of trial and error involved. However after getting the hang of it and understanding the basics, most of my problems stemmed from the eShopOnWeb project. I spent probably 5%-10% of my time with the first 2 tasks and 90%-95 with the last one.

With every single one of the issues I encountered I always tried Googling/Youtubing and just debugging as much as possible, there is nothing special to my methods, and it's very seldom that I use a different method to get something working. The internet is my friend, and it's the only reason I got any of this working :D

Some funky issues I encountered:

- Installing dotnet tools: 

For this I initially used `dotnet tool install  dotnet-ef --global` and I got an error, I spend hours debugging this only to realise that the I needed to do a local installation instead of a global one. Most of the fixes for the error I got pointed me in the complete opposite direction.

- Connecting the app to SQLServer: 

Setting up the environment for the apps was super easy, getting them to connect was a pain in the butt. The problem stemmed from a VERY case sensitive connection string that looks like this : `Server=10.104.12.34,1433\\SQL2022;Database=mssqllocaldb;User Id=SA;Password=test123-;Initial Catalog=Microsoft.eShopOnWeb.CatalogDb;` If you mess up the order, or leave out ANYTHING It just would not connect.

- Getting `dotnet run` to run the application: 

This was just a weird one for me, the apps would not run without me having to delete the `wwwroot` folder from each app's directory. From what I've found out, this is due to `dotnet run` not wanting to replace files, even if they are created at runtime.
