# Sybrin - Janco's DevOps Test:

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

### Setting up Jenkins <-> Kubernetes was a fun challange and introduced me to some new concepts. The steps I took to get everything working were:
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


## Task 3:

### This task taught me a lot, I spent most of my time debugging and learning.
Using `dotnet run` you're able to run both apps, however If you use `dotnet publish // dotnet exec` It resulted in errors. I've concluded that the only way to fix this is by altering a shit ton of source code. I tried for many hours to get this to work, and tried various fixes, but in the end I think the jump to .NET6 broke some things in this project. I think I easily have around 200+ failed builds for this project :)

#### The steps I took:

### SQL Server:

- Create and apply sqlserver-dep.yaml to kubernetes, this has the deployment and services included as well as a custom password for the server.
```
kubectl apply -f C:\Users\dayzd\.kube\sqlserver-dep.yaml
```
- Create an initial database called mssqllocaldb(thit is needed for the connection string in the application).
```
CREATE DATABASE mssqllocaldb;
```

### eShopOnWeb:

- I had to Fork the repo(https://github.com/ENIAC-ZA/eShopOnWeb) and make changes to `/src/Web/appsettings.json` & `/src/PublicApi/appsettings.json`. The changes were needed to point to the SQL-Server and to define URLs/Ports. The updated .json files look like this



- Web app:
<img src="https://i.imgur.com/cLAlBPC.png" height="350" />

- Api:
<img src="https://i.imgur.com/9ZtkKco.png" height="350" />

- Create the services for the apps in kubernetes:

```
kubectl apply C:\Users\dayzd\.kube/eShop-services.yaml
```



#### Now I start running into problems:

I can test if everything is working by cloning the repo, restoring the project and running it using `dotnet run` in the pipeline. This test pipe is called 'test-jenkins-pipe.yaml'
If I do this, the output of the pipe looks good and we can see no errors:
Web App:
<img src="https://i.imgur.com/nUh6mcf.png" height="350" />
Api App:
<img src="https://i.imgur.com/jCJmxNy.png" height="350" />

The only issue is that using `dotnet run` only runs the application in an interactive-terminal session. Not useful for deploying.

- Restoring and publishing the projects works flawlessly, and you're presented with nice .DLL files. Trying to deploy these files using `dotnet` or `dotnet exec` just does not want to work. The output is identical for both apps:
<img src="https://i.imgur.com/hw9Sf2F.png" height="100" />

- Going to the line that throws the error you're presented with this:

<img src="https://i.imgur.com/tRuxJXK.png" height="250" />


I have tried every combination of launch parameters to try and fix this issue, I tried making a runttimeconfig.json file. But I just couldn't get this error to go away. Something is bugging out, and it's not getting the values from the appsettings.json file

### The complete pipe I built can be found at 'eShopp-jenkins-pipe.yaml' and should work flawlessly if the `System.InvalidOperationException` issue could be solved.
quick summary:
- SQL-Server works.
- Both apps restore.
- Both apps connect to the database, tables get created.
- Both apps publish.
- Both apps "run". `dotnet run`
- No apps "deploy". `dotnet` // `dotnet exec`

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

#### By far the thing I found most intereting was learning how to build and use CI/CD pipelines, I understood the concept. But seeing just how simple it can be blew my mind away, I immediately started thinking about projects I could apply this to and I just fell in love. Going forward, anything that I put into some sort of release enviornment will absolutely run through jenkins + kube. 



## Question 4:

#### First off let me say that If I had to list all the problems I encountered I could write a book, this was my first time using Kubernetes and Jenkins so there was a lot of trail and error invloved. However after getting the hang of it and understanding the basics, most of my problems stemmed from the eShopOnWeb project. I spent probably 5%-10% of my time with the first 2 tasks and 90%-95 with the last one.

With every single one of the issues I encountered I always tried Googl-ing/Youtub-ing and just debugging as much as possible, there is nothing special to my methods, and it's very seldom that I use a different method to get something working. The internet is my friend, and it's the only reason I got any of this working :D

Some funky issues I encountered:

- Installing dotnet tools: 

For this I initially used `dotnet tool install  dotnet-ef --global` and I got an error, I spend hours debugging this only to realise that the I needed to do a local installation instead of a global one. Most of the fixes for the error I got pointed me in the complete opposite direction.

- Connecting the app to SQLServer: 

Setting up the enviornment for the apps was super easy, actually getting them to connect was a pain in the butt. The problem stemmed from a super case sensitive connect string that looks like this : `Server=10.104.12.34,1433\\SQL2022;Database=mssqllocaldb;User Id=SA;Password=test123-;Initial Catalog=Microsoft.eShopOnWeb.CatalogDb;` If you mess up the order, or leave out ANYTHING It just would not connect.

- Actually getting `dotnet run` to run the application: 

This was actually just a weird one for me, the apps would not run witout me having to delete the `wwwroot` folder from each app's directory. From what I've found out, this is due to `dotnet run` not wanting to replace files, even if they are created at runtime.

- Volatile storage/ Load-Shedding: 

This is the one thing that kept me on edge at all times, I never set up anything with 'persistent volume'//'persistent volume claims', this meant that every time I lost power I had to do a whole bunch of reconfiguring. I tried making a persistent volume and a persistent volume claim, gave my containers admin rights but they consistently failed with `insufficient rights`. Instead of wasting my time any further I decided to just focus on the tasks and reconfigure everything when the power goes out :( .
