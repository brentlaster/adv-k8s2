# Advanced Kubernetes
## Gaining Mastery over your Deployments
## Session labs for codespace only
## Revision 1.0 - 07/09/23

**Lab 1- Working with Kubernetes Probes **

**Purpose: In this lab, we'll see how to build Docker images from Dockerfiles.**

1. For this workshop, files that we need to use are contained in the directory **adv-k8s** in the home
directory on the disk. Under that directory are subdirectories for each of the main topics that we will
cover. For this section, we’ll use files in the roar-probes directory. (roar is the name of the sample app
we’ll be working with.) Change into that directory. In the terminal emulator, enter.

>
> **\$ cd roar-probes**
>

2. In this directory, we have Helm charts to deploy the database and webapp parts of our application. You
can use the “tree” command to see the overall structure if you are interested. Then create a namespace
named “probes” to hold the deployment. Finally, deploy the app into the cluster with the “helm install”
command. (In the last command, the first occurrence of “probes” indicates the namespace, the second
is the name of the release, and the “.” means using the files from this directory.)

>
> **\$ tree**
>
> **\$ k create ns probes**
>

3.  Now let's see how things are progressing. Take a look at the overall status of the pods, also showing the
labels on the pods

>
> **\$ k get pods -n probes --show-labels**
>


4.  You should see that while the web pod is running, the database pod is not ready at all. (If the web pod
isn't "Running" yet, it may still need time to startup. You can run the command again after a minute or
so to give it time to finish starting up.). Now, let’s do a “describe” operation on the pod itself. We’ll
specify the pod using one of its labels instead of having to use the pod name.
>
> **\$ k describe -n probes pod -l app=mysql**
>

5.  Note the error message near the bottom of the output mentioning the readiness probe failed. The
readiness probe in this case is just an exec of a command to invoke mysql. The error implies that the call
to “mysql” failed. But note that it doesn’t say it couldn’t find it. Rather, it wasn’t valid to call it that way
since it tried to invoke it without a valid name and password to login.

6. You can see the YAML for this in the deployment template in the corresponding Helm chart. In the file explorer to the left,
 select the file **roar-probes/charts/roar-db/templates/deployment.yaml** file and click on it to open it up in the files area above the terminal.
Find the section near the bottom with the **readinessProbe** spec.

7. We actually don’t need to have a command login to verify readiness – we just need to know the mysql
application responds. Let’s fix this by simply calling the “version” command - which we should be able
to do without a login

8. Edit the file and add a **--version** option between the **mysql** and **failureThreshold:** lines, as shown below:

Change

```
readinessProbe:
exec:
command:
- mysql
failureThreshold: 3
initialDelaySeconds: 5
```
To add the line shown in bold
```
readinessProbe:
exec:
command:
- mysql
```
**- --version**
```
failureThreshold: 3
initialDelaySeconds: 5separate terminal window, take a look at that and find the section near the bottom with the
readinessProbe spec. (The figure below shows how to open an additional terminal window.)
```
9. Upgrade the helm installation. After a minute or so, you can verify that you have a working mysql pod.
(You may have to wait a moment and then check again.)

>
> **\$ helm upgrade -n probes probes .**
>

10. At this point, you can get the service's nodeport.

>    
> **\$ kubectl get svc -n roar \--show-labels**
>

11.  Next, let's look at the running application.  Run the command below.

> 
> **\$ kubectl port-forward -n roar svc/roar-web 8089 &**
>
![Port pop-up](./images/kint6.png?raw=true "Port pop-up")

12.  You should see a pop-up in your codespace that informs that `(i) Your application running on port 8089 is available.` and gives you a button to click on to `Open in browser`.  Click on that button. (If you don't see the pop-up, you can also switch to the `PORTS` tab at the top of the terminal, select the row with `8089`, and right-click and select `View in browser`.)

13.  What you should see in the browser is an application called **Apache Tomcat** running. Click at the end of the URL in the address bar and add the text `/roar/`.  Make sure to include the trailing slash.  Then hit enter and you should see the *roar* application running in the browser.

The complete URL should look something like
```console
https://gwstudent-cautious-space-goldfish-p7vpg5q55xx36944-8089.preview.app.github.dev/roar/
```
![Running app in K8s](./images/kint5.png?raw=true "Running app in K8s")
 
<p align="center">
**[END OF LAB]**
</p>

**Lab 2 - Exploring and Deploying into Kubernetes**

**Purpose:** In this lab, we'll start to learn about Kubernetes and its
object types, such as nodes and namespaces. We'll also deploy a version
of our app that has had Kubernetes yaml files created for it.

1.  Before we can deploy our application into Kubernetes, we need to
    have appropriate Kubernetes manifest yaml files for the different
    types of k8s objects we want to create. These can be separate files
    or they can be combined. For our project, there are separate ones
    (deployments and services for both the web and db pieces) already
    setup for you in the **k8s** directory. Change into that
    directory and take a look at the yaml file there for the Kubernetes
    deployments and services.
>
> **\$ cd ../k8s**
>
> **\$ cat *.yaml**
>

2.  We're going to deploy these into Kubernetes into a namespace. Take a
    look at the current list of namespaces and then let's create a new
    namespace to use.
>
> **\$ kubectl get ns**
>
> **\$ kubectl create ns roar**
>

3.  Now, let's deploy our yaml specifications to Kubernetes. We will use
    the apply command and the -f option to specify the file. (Note the
    -n option to specify our new namespace.)
>
> **\$ kubectl -n roar apply -f .**
>
After you run these commands, you should see output like the following:

```console
deployment.extensions/roar-web created
service/roar-web created
deployment.extensions/mysql created
service/mysql created
```

4.  Now, let's look at the pods currently running in our "roar"
    namespace (and also see their labels).
>
> **\$ kubectl get pods -n roar \--show-labels**
>

5.  Next, let's look at the running application.  Run the command below.

> 
> **\$ kubectl port-forward -n roar svc/roar-web 8089 &**
>
![Port pop-up](./images/kint6.png?raw=true "Port pop-up")

6.  You should see a pop-up in your codespace that informs that `(i) Your application running on port 8089 is available.` and gives you a button to click on to `Open in browser`.  Click on that button. (If you don't see the pop-up, you can also switch to the `PORTS` tab at the top of the terminal, select the row with `8089`, and right-click and select `View in browser`.)

7.  What you should see in the browser is an application called **Apache Tomcat** running. Click at the end of the URL in the address bar and add the text `/roar/`.  Make sure to include the trailing slash.  Then hit enter and you should see the *roar* application running in the browser.

The complete URL should look something like
```console
https://gwstudent-cautious-space-goldfish-p7vpg5q55xx36944-8089.preview.app.github.dev/roar/
```
![Running app in K8s](./images/kint5.png?raw=true "Running app in K8s")

8.  Let's check the logs of the database pod to see what the application is doing there. 
    Run the command below to see the logs of the database pod:
>
> **\$ kubectl logs -n roar -l app=roar-db**
>

9.  To get the overall view (description) of what's in the pod and what's happening
    with it, we can also use the "describe" command. Run the command below.
>
> **\$ kubectl -n roar describe pod -l app=roar-db**
>
    Near the bottom of this output, notice the *Events* messages which describe major events associated with the pod.

10.  To demonstrate the deployment functionality in Kubernetes, let's delete one of the pods. With your cursor in your terminal in the codespace, right-click and select the `Split Terminal` option. This will add a second terminal side-by-side with your other one.

![Splitting the terminal](./images/kint7.png?raw=true "Splitting the terminal")

11.  In the left terminal, run a command to start a `watch` of pods in the roar namespace.
>
> **\$ kubectl get -n roar pods -w**
>
12. Now in the right terminal, enter a command to delete the database pod. Watch the activity in the left terminal as the old pod is removed and a new, different one created by the deployment. (You can tell its a different pod by looking at the end of the name.)
>
> **\$ kubectl delete -n roar pod -l app=roar-db**
>

<p align="center">
**[END OF LAB]**
</p>
