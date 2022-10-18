# Sybrin - Janco's DevOps Test:

## Included in the readme is all the steps and filenames required to reproduce the tasks.


## Task 1:
### By far the easiest of the tasks, this was done mostly through a GUI, and verification of all the services was done through the command line.
Steps:

-Install docker from - https://docs.docker.com/desktop/install/windows-install/

-After installation enable WSL 2 instead of Hyper-V/nothing.

-Install kubernetes from within docker settings menu.



-After kubernetes is installed I had access to the 'kubectl' commands and verified that everything was working,  some of the commands I used were:

```
kubectl get nodes
kubectl get ns
kubectl proxy
```



-Sample user creation/token and installing k8-dashboard:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.1/aio/deploy/recommended.yaml
kubectl apply -f C:\Users\dayzd\.kube\dashboard-adminuser.yaml 
kubectl -n kubernetes-dashboard create token admin-user --duration 300h0m0s
```



-To deploy the 'simple-web' container, I created `simple-web-dep.yaml` and simply applied it to kubernetes:
```
kubectl apply -f C:\Users\dayzd\.kube\simple-web-dep.yaml
```

## Task 2:
### Setting up Jenkins <-> Kubernetes was a fun challange and introduced me to some new concepts. The steps I took to get everything working were:
-Create and apply a jenkins-dep.yaml to kubernetes:
```
kubectl apply -f C:\Users\dayzd\.kube\jenkins-dep.yaml
```
-After deploying jenkins, it was accessible from localhost:30000 I used 'docker cp' to get the initial password, I saw that it's available from log files as well.
```
docker ps (get jenkins container ID)
docker cp 8dacbc0cc5e4:/var/jenkins_home/secrets/initialAdminPassword C:\Users\dayzd\Downloads
```
-After the initial setup of jenkins + user creation I added the kubernetes plugin and restarted.

-Setting up kubernetes/jenkins agents I left most values on default, the only values I had to change were:
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
-I also made sure to decrease the jenkins default agents to 0.

-Jenkins was now ready for deployments.


## Task 3:
### This task taught me a lot, I spent most of my time debugging and learning.
Using `dotnet run` you're able to run and access the pages, however If you use `dotnet publish + dotnet exec` there is a build error. I've spend many hours trying to get this to work without having to change heaps of source code but this just was not feasible. 


## Question 1:





## Question 2:




## Question 3:
