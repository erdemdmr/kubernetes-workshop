# kubernetes-workshop

: Workshop Overview
The Kubernetes web site describes Kubernetes as:

an open-source system for automating deployment, scaling, and management of containerized applications.

Kubernetes

This workshop is intended to give you a quick hands on introduction with using Kubernetes. In the process you will learn about some of the fundamental concepts of Kubernetes when deploying applications to it. The focus will be on what a developer would need to know to use the platform. It is not a workshop on how to run the Kubernetes platform.

2: Accessing the Cluster
For the exercises you will be doing, you will be using the kubectl command line program to interact with Kubernetes. This is provided for you via the interactive terminal session accessible through the Terminal tab, here in the workshop environment. You do not need to install anything on your own computer. You will be doing everything here through your web browser. There is no need to login as you are already connected to the Kubernetes cluster you will be using.

The workshop environment also provides you with a web based view into the Kubernetes cluster. This is available through the Console tab of the workshop environment. This is included so you can visually see the results of what you do in the exercises, but the exercises do not depend on it.

Before continuing, verify that the kubectl command runs and the workshop environment is also functioning. To do this run:

kubectl version
Did you type the command in yourself? If you did, click on the command here instead and you will find that it is executed for you. You can click on any command block here in the workshop notes which has the  icon shown to the right of it, and it will be copied to the interactive terminal and run for you. Other action blocks may also be used in this workshop, showing different icons, you can also click on these to trigger the action described.

When run, you should see output similar to:

Client Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.0", GitCommit:"70132b0f130acc0bed193d9
ba59dd186f0e634cf", GitTreeState:"clean", BuildDate:"2019-12-07T21:20:10Z", GoVersion:"go1.13.4", Compiler:"
gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.0", GitCommit:"70132b0f130acc0bed193d9
ba59dd186f0e634cf", GitTreeState:"clean", BuildDate:"2019-12-07T21:12:17Z", GoVersion:"go1.13.4", Compiler:"
gc", Platform:"linux/amd64"}
The version of Kubernetes being used may be different to the version shown here.

3: Deploying an Application
Now you have checked your access to the Kubernetes cluster is working, we are going to immediately jump in and deploy a complete application, consisting of a front end web application implementing a blog site, along with a PostgreSQL database for storing the blog posts.

This is to show you how quickly you can deploy a complete application to Kubernetes if you already have the configuration. Once the complete application has been deployed, we will delete the front end web application component, and deploy it again in steps so you can see how it fits together and how it uses Kubernetes.

The first part of the application we want to deploy is the PostgreSQL database. The set of resource files for deploying this can be found in the database directory.

ls -las database/
Each file in the directory contains a different resource definition which go together to make up the deployment for the application component.

Rather than try and dig into each file to work out what it defines, you can have Kubernetes tell you what resources it would create when the directory of resources is processed. To do this, run:

kubectl apply -f database/ --dry-run
This should output:

secret/blog-credentials created (dry run)
service/blog-db created (dry run)
persistentvolumeclaim/blog-database created (dry run)
deployment.apps/blog-db created (dry run)
The kubectl apply command in this case is what is used to create resources from a configuration file, or set of files contained in a directory. We used the --dry-run option, which tells us what objects would be created without creating any of the objects in the cluster. The --dry-run option also validates the resource definitions and will warn you if you they contain errors.

If you are ever uncertain about what a command does, or what options it accepts, you can run it with the --help option.

kubectl apply --help

4: Creating the Resources
By doing a dry run deployment, you have seen the resources that will be created. To actually deploy the database component, now run:

kubectl apply -f database/
As with the dry run, kubectl apply will list the resources, except this time the resources will be created.

secret/blog-credentials created
service/blog-db created
persistentvolumeclaim/blog-database created
deployment.apps/blog-db created
The key resource in this list is deployment. It specifies the name of the container image to be deployed for an application, how many instances should be started, and the strategy for how the deployment should be managed.

To monitor progress of the deployment, and know when it has completed, you can run the command:

kubectl rollout status deployment/blog-db
The argument is the full name of the resource, including the type of resource and the name for this instance. In this case the instance was called blog-db.

With the database deployed, now deploy the front end web application by running:

kubectl apply -f frontend/
This should output:

persistentvolumeclaim/blog-media created
deployment.apps/blog created
service/blog created
ingress.extensions/blog created
Run:

kubectl rollout status deployment/blog
to monitor and wait for it to be deployed.

Notice that an ingress object is created for the front end web application. This object sets up access to our web application using a publicly accessible URL.

In this example, the URL for accessing the web application will be:

http://blog-springone-w01-s1703.apps.asdf1.tanzu-devs.com
Visit the front end web application by clicking on this link. If it shows as not being available, keep refreshing the page until it is. This is necessary as it may take a few moments to reconfigure the ingress routing layer.

There will not be any blog posts displayed as yet. We will get to setting up and populating the database later.

5: Querying the Resources
The web console for a Kubernetes cluster can be used to display a visual representation, in your browser, of what resources have been created and the relationship between them, but most developers will interact with a Kubernetes cluster from the command line using kubectl.

To see a list of all the deployments in the current namespace which have already been created, run:

kubectl get deployment
This should yield output:

NAME      READY   UP-TO-DATE   AVAILABLE   AGE
blog      2/2     2            2           5m
blog-db   1/1     1            1           5m
To narrow in on a specific resource, the name of that resource can be added to the command:

kubectl get deployment/blog
This should then yield output for just the one resource.

NAME   READY   UP-TO-DATE   AVAILABLE   AGE
blog   2/2     2            2           5m
To see much more detailed information about a resource kubectl describe can be used.

kubectl describe deployment/blog
For a deployment, just the start of what you should see is:

Name:                   blog
Namespace:              lab-k8s-fundamentals-user1
CreationTimestamp:      Tue, 04 Feb 2020 03:06:56 +0000
Labels:                 app=blog
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=blog
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
    ....
This is still in a semi human readable form and isn't suitable for machine processing. To instead see the raw resource definition, you can use the -o yaml display output option to kubectl get.

kubectl get deployment/blog -o yaml
Or if you prefer to work with JSON rather than YAML, you can use:

kubectl get deployment/blog -o json

6: Labelling of Resources
The ability to use kubectl to query resource definitions and their current status is important because it is the resource definitions which drive Kubernetes and what it does.

As we already saw, it isn't a single resource definition which defines everything about the deployment of an application, but a set of resources.

In order to indicate that a set of resources are related, they can be labelled. You can then perform queries to look up resources based on the labels.

In order to narrow the results down so it shows just the resources for the front end web application, we can add to the kubectl get command a label selector using the -l or --selector option.

kubectl get deployment,service,ingress,secret,pvc -o name -l app=blog
In this case we have also provided a comma separated list of the resources we wish to query about in addition to searching based on the applied labels.

This should yield:

deployment.apps/blog
service/blog
ingress.extensions/blog
persistentvolumeclaim/blog-media
For the sample application you are deploying, all resources for the front end web application were given a label of app=blog.

One important use case for labels is when you want to delete an application. Where an application is defined by a set of resources, it is cumbersome and error prone to have to delete each resource one at a time. Using a label selector, you can delete them all at once.

The procedure for deleting an application would be to first use kubectl get with a label selector to determine that you have the correct set of resources, and then substitute kubectl get with the kubectl delete command.

Delete the front end web application now by running:

kubectl delete deployment,service,ingress,secret,pvc -l app=blog
This should output:

deployment.apps "blog" deleted
service "blog" deleted
ingress.extensions "blog" deleted
persistentvolumeclaim "blog-media" deleted
Having deleted the front end web application, we will now re-deploy it, but this time we will do it one step at a time so you can understand what each of the resources is for and what Kubernetes does in response to the resource definitions being created.

7: Deployment Resource
As already highlighted, a key resource created when deploying an application is the deployment resource. It specifies the name of the container image to be deployed for an application, how many instances should be started, and the strategy for how the deployment should be managed.

To view the deployment resource used for the front end web application, run:

cat frontend/deployment.yaml
This has various parts to it, as well as dependencies on other resources, so let's start over and create this from scratch.

To create a new deployment resource what often happens is that a developer will copy an existing one, be it one from an existing application, or a sample provided in documentation or a blog post.

An alternative is to have kubectl create it for you. For a deployment, the kubectl command provides two options for creating the resource definition for you.

The first option is using the kubectl create deployment command.

kubectl create deployment --help
For the deployment of the front end web application, the container image we want to use is quay.io/eduk8s-labs/app-k8s-fundamentals-frontend:latest.

To see what kubectl create deployment would create for us run:

kubectl create deployment blog --image quay.io/eduk8s-labs/app-k8s-fundamentals-frontend:latest --dry-run -o yaml
This should yield:

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: blog
  name: blog
spec:
  replicas: 1
  selector:
    matchLabels:
      app: blog
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: blog
    spec:
      containers:
      - image: quay.io/eduk8s-labs/app-k8s-fundamentals-frontend:latest
        name: app-k8s-fundamentals-frontend
        resources: {}
status: {}
Note that nothing has been created at this point as when we ran kubectl create deployment we used the --dry-run option, it therefore only showed what it would create.

Although kubectl create deployment could be used, it is a very bare bones skeleton for a deployment object.

A second option is to use kubectl run.

kubectl run --help
This command accepts numerous options for helping you fill out the resource definition with additional key configuration you might need.

To start to replicate the configuration for our sample application, run:

kubectl run blog --image quay.io/eduk8s-labs/app-k8s-fundamentals-frontend:latest --labels app=blog --replicas 2 --port 8080 --env BLOG_SITE_NAME="EduK8S Blog" --dry-run -o yaml
This should produce:

apiVersion: apps/v1beta1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: blog
  name: blog
spec:
  replicas: 2
  selector:
    matchLabels:
      app: blog
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: blog
    spec:
      containers:
      - env:
        - name: BLOG_SITE_NAME
          value: EduK8S Blog
        image: quay.io/eduk8s-labs/app-k8s-fundamentals-frontend:latest
        name: blog
        ports:
        - containerPort: 8080
        resources: {}
status: {}
The difference is that it has set the number of replicas of our application that we want to be 2 instead of 1. It includes the port number our application listens on, and includes one of the environment variables we want to be set. We have also ensured the label used is app=blog. This version of the deployment is still far from complete, but it is enough to get us started.

One could now re-run the command and leave off the --dry-run option and it would create the resource. To make subsequent changes to the resource definition, one could then edit it in place using the kubectl edit command.

Maintaining the master copy of the configuration in Kubernetes like this is fine in a development environment, but for production, it is better to keep the master copy as a local file, under revision control so you can track it, and when changes are needed, edit the local master copy and apply it to the Kubernetes cluster.

The Kubernetes documentation has a discussion about these two different approaches to managing configuration in Managing Kubernetes Objects Using Imperative Commands and Declarative Management of Kubernetes Objects Using Configuration Files.

As it is closer to what you would want to do for a production environment, we will use the latter approach.

For this first attempt towards replicating the front end web application, the output from kubectl run has been captured in the file frontend-v1/deployment.yaml. You can see the full contents of the directory by running:

ls -las frontend-v1
As is, it is only the one file. We could at this point run kubectl apply on just this file, but as we go along we will be adding additional files for other resources. We will therefore continue to use the ability of kubectl apply to be given a directory of files to process and apply them in one operation.

Create the deployment by running:

kubectl apply -f frontend-v1/
It should output:

deployment.apps/blog created
Monitor progress of the deployment so you know when it has completed.

kubectl rollout status deployment/blog
Note that although we used kubectl run as a basis for creating an initial version of the deployment resource, from Kubernetes 1.18 the ability to create a deployment using kubectl run has been removed, instead it creates a pod. This workshop still needs to be rewritten to cater for this change and is currently still using kubectl from Kubernetes 1.17. If you are using Kubernetes 1.18 or later, you will need to construct an initial deployment by hand using kubectl create deployment, but because it is missing the various parts explained above, they will need to be added.


8: ReplicaSets and Pods
Having created the deployment run:

kubectl get all -o name -l app=blog
The all value is not a resource type, but a short hand alias for the core Kubernetes resource types. It shows what we need here, but to be sure to get everything you want, you are usually better to list the resource types explicitly.

The output should be similar to:

pod/blog-6b8999855c-6jjhj
pod/blog-6b8999855c-zk8pg
deployment.apps/blog
replicaset.apps/blog-6b8999855c
Although you only created the deployment, this has resulted in the creation of additional resources for replicaset and pod.

This is because deployment acts as a template for the creation of a replicaset. A replicaset in turn acts as a template for the creation of the pods. It is the pods which represent the instances of your application. In this case, because the number of replicas has been set to 2, there are 2 pods.

To view the resource definition for the replicaset run:

kubectl get replicaset -l app=blog -o yaml
In this you will see that the spec section contains:

  spec:
    replicas: 2
    selector:
      matchLabels:
        app: blog
        pod-template-hash: "896156207"
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: blog
          pod-template-hash: "896156207"
      spec:
        containers:
        - env:
          - name: BLOG_SITE_NAME
            value: EduK8S Blog
          image: quay.io/eduk8s-labs/app-k8s-fundamentals-frontend:latest
          imagePullPolicy: Always
          name: blog
          ports:
          - containerPort: 8080
            protocol: TCP
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
        dnsPolicy: ClusterFirst
        restartPolicy: Always
        schedulerName: default-scheduler
        securityContext: {}
        terminationGracePeriodSeconds: 30
This has been filled out with what was provided in the spec portion of the deployment.

To view the resource definitions for the pods run:

kubectl get pod -l app=blog -o yaml
Here the fields from spec.template of the replicaset have been used in creating the pod resource definition. The spec.template of a deployment and replicaset is what is referred to as the pod template.

In both replicaset and pod you will see that a lot of additional fields have been added along the way. This is because they are being filled out with defaults for values which weren't specified in the original deployment. The resource definitions also contain fields which help track the status of whatever the resource represents.

In the case of the inherited defaults, these may be from the resource type, but also may be inherited in part from the global or namespace configuration. This is the case for the resource limits on CPU and memory, which have been inherited from limits set for the namespace you are working in.

In any case, if these defaults turn out not to be correct and you need to change them, or you need to add additional settings, the change should be made in the deployment. You should not edit replicaset or pod directly yourself. Update the deployment instead. The instances of replicaset and pod will be correspondingly updated for you.

When it comes to deleting an application, as we did previously for the front end web application, there is no need to explicitly delete either the replicaset or pods. This is because the pods are marked up as being owned by the replicaset they are created from, and the replicaset is marked as being owned by the deployment. When you delete the deployment, the replicaset, and the pods created from it, will be automatically deleted.

9: Replicas and Scaling
One example of updating the deployment is to change the number of replicas, or instances of your application which are running. To effect this change you would need to change the value of spec.replicas in the deployment.

The kubectl command provides an imperative command for updating the number of replicas. To increase the number of replicas to 3, run:

kubectl scale deployment/blog --replicas 3
This will output:

deployment.apps/blog scaled
and if you run:

kubectl get pods -l app=blog
you should now see that there are 3 pods running instead of the original 2.

If you ran the command fast enough, you may see a pod listed as being in Pending or ContainerCreating state. This is the new pod when it is starting up. Keep running kubectl get pods until you see 3 pods in the Running state.

The problem with having run kubectl scale is that the configuration in Kubernetes no longer matches what we used to originally create the deployment. This means we wouldn't be able to replicate the application deployment, as it existed in Kubernetes at that point, by deploying the original configuration alone.

To reset the configuration in Kubernetes back to what we had before running kubectl scale you can run kubectl apply with the original configuration we had in our local configuration file.

kubectl apply -f frontend-v1/
Run:

kubectl get pods -l app=blog
again and you will see that you are back to 2 replicas.

If you ran the command fast enough after applying the original configuration you may see a pod listed as being in Terminating state. This is the pod which is being shutdown in order to bring the number of replicas back to 2. Keep running kubectl get pods until you see the number in the Running state return back to 2.

This continual process whereby Kubernetes will ensure that the number of instances of your application, i.e., pods, matches the desired number of replicas specified in the deployment, comes into play in another way as well. This is that if an instance of your application was killed, Kubernetes will replace it automatically.

You can simulate this scenario by deleting one of the pods. To see what happens, first run in one terminal:

kubectl get pods -l app=blog --watch
The --watch option to kubectl get pods says to monitor the pods over time and show any changes.

Now from another terminal delete one of the pods.

kubectl delete `kubectl get pod -l app=blog -o name | head -1`
You should see the pod which was targeted being marked as Terminating and it will be removed. Because though the desired number of replicas is 2, a new instance of your application will be automatically started to replace it.

When complete, interrupt the kubectl get --watch command to stop it.

<ctrl+c>

10: Pods and Containers
It has already been mentioned that a pod represents an instance of your application.

To be more precise, a pod is an abstraction which represents a group of running containers, where the containers are to be managed and scaled as a unit.

An instance of your application runs in one container of each pod resulting from the deployment. Where there would only be multiple pods if the replica count on the deployment was greater than 1.

In most cases a pod will consist of only a single container. There are use cases for running multiple containers in one pod, but they are generally the exception rather than the rule.

Looking at the sample application being used in this workshop, there are two separate deployments. One for the front end web application, and the other for the database.

They are separated as it allows the front end web application to be scaled up to multiple instances independent of the database. If both components had been created as part of the one deployment, running in separate containers of a pod, it would not be possible to scale the application up to multiple instances. This is because scaling a database isn't as simple as increasing the replica count.

The sequence of events which results in your application being run is therefore, that the deployment is created, from which a replicaset is created. From the replicaset, pods are created corresponding to the replica count specified in the deployment. For each pod created, a container is run using the container image supplied in the deployment. This is the instance of your application.

11: Application Logging
Each instance of an application runs in it's own pod.

As you have already seen, you can list the pods for the front end web application using:

kubectl get pods -l app=blog -o name
To access the log output for a specific instance of your application, you can use the name of the pod with the kubectl logs command.

As we have multiple pods, we need to grab just one of the names.

POD=`kubectl get pod -l app=blog -o template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}' | head -1` && echo $POD
Then run kubectl logs:

kubectl logs $POD
Rather than identify a specific pod, you can also run kubectl logs against the deployment using:

kubectl logs deployment/blog
Where there are multiple pods, this will result in one of the pods associated with the deployment being randomly selected. So you will need to select the pod if you need to be sure about which one the logs are being retrieved from.

If there are multiple containers in the pod, you would need to name the container using the -c or --container option. Alternatively, you could use the --all-containers option to fetch logs from application process running in any of the containers.

If you wanted to follow the output of the running application, you can use the -f or --follow option.

When you have multiple replicas of your application, you would need to fetch the application logs from each pod. A Kubernetes cluster may optionally have a service deployed for aggregated logging, in which case you could access that service to view logs for all instances of the application at the same time.

12: Accessing Containers
To gain access to the container in which an instance of an application is running, and run a command, you can use kubectl exec.

As with logging, you need to specify the particular pod you want to access, and if there are multiple containers running in the pod, specify which container using the -c or --container option.

kubectl exec $POD env
If you want to run an interactive terminal session, you need to ensure you use the -i or --stdin option, and the -t or --tty option.

kubectl exec -it $POD bash
From the interactive shell, you can view files in the file system.

ls -las
and can interact with the application processes:

ps x
The only processes you will be able to see are those for the instance of your application running in that container. You cannot see processes running in other containers of the same pod, or operating system processes.

Run:

exit
to end the interactive terminal session for the container.

As with logs, you can use kubectl exec against the deployment, but you will not know which pod it will be run against if there is more than one.

13: Service Networking
The pod corresponding to each instance of your application is ephemeral. If it dies it is not resurrected. The replicaset ensures that a pod that has died is replaced with a new instance, as well as ensuring the correct number of pods exist when scaling the number of replicas up or down. When a new pod is created, it will always have a new name.

Each pod, as well as having a unique name, is also assigned it's own IP address. You can see the IP addresses assigned to each pod by running:

kubectl get pods -l app=blog -o wide
These IP addresses are only accessible within the Kubernetes cluster. Depending on the Kubernetes networking configuration, they may only be accessible from applications running in the same namespace.

Like with the names of pods, the IP addresses in use for an application will not stay the same over time. When a pod dies and is replaced, it can receive a completely different IP address. IP addresses cannot be relied upon for communicating between components in an application.

To add a stable IP address and hostname for an application, a service resource needs to be created. The IP address of a service will map to the set of pods which make up your application, with traffic to the IP address of the service being load balanced across the pods.

As with creating a deployment, the kubectl program provides methods for creating a service. These are kubectl expose and kubectl create service. We will skip these and use the resource definition. Run:

cat frontend-v2/service.yaml
to see the service definition. You should see:

apiVersion: v1
kind: Service
metadata:
  name: blog
  labels:
    app: blog
spec:
  type: ClusterIP
  selector:
    app: blog
  ports:
  - name: 8080-tcp
    port: 8080
    protocol: TCP
    targetPort: 8080
When used to create the resource object, this will result in a service named blog being created. The ports definition says that port 8080 will be exposed, with it mapping to port 8080 on the pods.

Update the current application configuration by running:

kubectl apply -f frontend-v2/
The output should be:

deployment.apps/blog unchanged
service/blog created
As the frontend-v2 directory also contains our original deployment.yaml file, this will also ensure that the current deployment is brought into line with what it defines.

To review details of the service created run:

kubectl get service --selector app=blog -o wide
This will display output similar to:

NAME      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE       SELECTOR
blog      ClusterIP   172.30.70.193   <none>        8080/TCP   1m        app=blog
You can see that the service has it's own IP address.

Another application running in the same namespace can connect on this IP address and port 8080 to talk to the front end web application.

The service will know what the corresponding pods are that traffic should be load balanced across by virtue of the label selector defined in the service. This is the spec.selector value:

  selector:
    app: blog
This means that the IP addresses for the pods resulting from a query of:

kubectl get pods -l app=blog -o name
will be registered as endpoints against the service.

You can see the IP addresses of the pods registered against the service by running:

kubectl get endpoints blog
Although the IP address will not change for the life of the service object, the IP should still not be used. Instead, a hostname corresponding to the service should be used. The hostname is the name of the service, is registered within an internal DNS in the Kubernetes cluster, and can be used by any application within the cluster.

For an application running in the same namespace, an un-qualifed hostname can be used. In this case blog would be the hostname. If networking in the cluster is configured to allow access across namespaces, an application in a different namespace can use the name with subdomain matching the name of the namespace, and the further domain of .svc.

For the front end web application you have deployed, the URL for accessing it would be:

http://blog.springone-w01-s1703.svc:8080
where springone-w01-s1703 is the subdomain added for the namespace.

You can test it works by running:

curl http://blog.springone-w01-s1703.svc:8080
Note that this still isn't accessible outside of the Kubernetes cluster, extra steps are required to expose a service outside of the cluster. The curl command only works because the terminal you are using is running as a pod in the same Kubernetes cluster.

14: Exposing the Service
In order to expose a service so that it is accessible outside of the Kubernetes cluster, you need to create an ingress resource object.

To see the definition for the ingress resource object we will use run:

cat frontend-v3/ingress.yaml
You should see output:

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: blog
  labels:
    app: blog
spec:
  rules:
  - host: blog-springone-w01-s1703.apps.asdf1.tanzu-devs.com
    http:
      paths:
      - path: "/"
        backend:
          serviceName: blog
          servicePort: 8080
The rules section in the ingress definition is what controls what should happen when traffic for your web application is received by the router for the Kubernetes cluster.

In this case the rule says that any HTTP requests received for the host blog-springone-w01-s1703.apps.asdf1.tanzu-devs.com should be directed to the application with service object named blog.

When an external user accesses the host name from their web browser, they will use the standard port 80 for HTTP traffic, the router will pass through that traffic to port 8080 of the service.

To update the configuration for the front end web application to add the ingress run:

kubectl apply -f frontend-v3/
This should output:

deployment.apps/blog unchanged
ingress.extensions/blog created
service/blog unchanged
You can review the ingress resource which was created by running:

kubectl get ingress -l app=blog
Now that the ingress has been created, you can access the front end web application using a web browser at:

http://blog-springone-w01-s1703.apps.asdf1.tanzu-devs.com
Visit the front end web application by clicking on this link. If it shows as not being available, keep refreshing the page until it is. This is necessary as it make take a few moments to reconfigure the ingress routing layer.

Note that this works because a wildcard CNAME has already been pre-configured in an external domain name server (DNS) to direct traffic for this host to the router for the Kubernetes cluster. If this was your own Kubernetes cluster, you would need to configure an appropriate CNAME in the DNS for the host you want to use.

15: Linking the Database
The front end web application is running and is accessible to the public internet. At this point it is using a file based SQLite database. This database is local to each instance, not shared between all (making it unsuited to a scaled application), and any changes to data will be lost whenever the pod restarts.

To provide persistence for data, and have all instances of the application using the same database, we will configure the front end web application to use the separate Postgresql database which is already running.

To view the resources for the database, run:

kubectl get deployment,service,pvc,secret -l app=blog-db -o name
This should yield:

deployment.apps/blog-db
service/blog-db
persistentvolumeclaim/blog-database
secret/blog-credentials
You should understand now the purpose of deployment and service. The persistentvolumeclaim is used to claim persistent storage for use by the database. The secret is used to hold the credentials for the database.

In order to link the database to the front end web application, we need to tell the front end web application the name of the host for the database, and what the login credentials are. To do this we need to add environment variable settings to the deployment for the front end web application.

You can see what environment variables are already set for the front end web application by running:

kubectl set env deployment/blog --list
This should display:

# Deployment blog, container blog
BLOG_SITE_NAME=EduK8S Blog
The way that the front end web application is implemented, it is expecting the following environment variables for a separate database.

DATABASE_HOST - The host name of the database.
DATABASE_USER - The user to login into the database.
DATABASE_PASSWORD - The password of the user for the database.
DATABASE_NAME - The name of the database.
To set environment variables, kubectl provides the command kubectl set env. We don't want to update the live configuration, but keep the configuration locally, but we can use it to work out the changes we need to make to that configuration.

For the database host, the host name will be the name of the database service object, which is blog-db.

To see what the deployment configuration would look like with that set, we can run:

kubectl set env deployment/blog DATABASE_HOST=blog-db --dry-run -o yaml
From the output, you can see that along side the existing BLOG_SITE_NAME environment variable, you now have the DATABASE_HOST environment variable. This is under the spec.template.spec.containers.env setting.

    spec:
      containers:
      - env:
        - name: BLOG_SITE_NAME
          value: EduK8S Blog
        - name: DATABASE_HOST
          value: blog-db
The database credentials could be added in a similar way, but for this application we already have those stored in the secret for the database. You can view the secret called blog-credentials:

kubectl get secret/blog-credentials -o yaml
Within the output you will see the data section holding values:

data:
  database-name: YmxvZw==
  database-password: dG9wLXNlY3JldA==
  database-user: YmxvZw==
What you see aren't the actual values as they have been obfuscated using base64 encoding.

In order to use the same values, but not actually have to copy them, you can configure the deployment to inject the environment variables from the secret. To see how the configuration should look for this you can run:

kubectl set env deployment/blog --from secret/blog-credentials --dry-run -o yaml
For these, the spec.template.spec.containers.env setting would need to be updated to:

    spec:
      containers:
      - env:
        - name: BLOG_SITE_NAME
          value: EduK8S Blog
        - name: DATABASE_NAME
          valueFrom:
            secretKeyRef:
              key: database-name
              name: blog-credentials
        - name: DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              key: database-password
              name: blog-credentials
        - name: DATABASE_USER
          valueFrom:
            secretKeyRef:
              key: database-user
              name: blog-credentials
Combining all of these we get:

cat frontend-v4/deployment.yaml
To apply this configuration run:

kubectl apply -f frontend-v4/
The database and the front end web application are now linked, but some extra steps are required to initalise the database.

16: Setting up Database
For the front end web application being used, the database will be initialised if required when the application first starts up. What hasn't yet been done is to setup an administrator password for the front end web application itself. You might also want to load in some initial data, such as some posts for our blog site.

To do this we need to execute some commands within the running container for one of the instances of the front end web application.

Grab the name of one of the pods which are running:

POD=`kubectl get pod -l app=blog -o template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}' | head -1` && echo $POD
and create an interactive terminal session.

kubectl exec -it $POD bash
To setup an administrator password run:

warpdrive setup
This will ensure that the database has been initialised and then prompt for the user information. Enter a user name for the administrator:

admin
Then an email address:

admin@example.com
A password:

this-is-secret
and confirm the password:

this-is-secret
A final thing this setup script will do is also load some initial posts.

Exit the interactive shell by running:

exit
You should now be able to visit the blog site at:

http://blog-springone-w01-s1703.apps.asdf1.tanzu-devs.com
and see the posts.

You can also if you want click on the person icon at the top right of the page, and login with the credentials entered. This will allow you to enter additional posts.

17: Persistent Volumes
One of the resources associated with the database was a persistent volume. We also need a persistent volume for the front end web application. This is required because although the blog post content is stored in the database, any image attached to the post is stored in the file system. As the container file system is ephemeral, the images would be lost when a pod is killed. The container file system is also not shared between the multiple instances of the application.

To add persistent storage to an application, the first step is that you need to create a persistent volume claim. This tells Kubernetes that you need storage, how big the volume needs to be and what type of storage is required.

The resource definition for the persistent volume claim we need to use can be seen by running:

cat frontend-v5/persistentvolumeclaim.yaml
You should see that it contains:

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: blog-media
  labels:
    app: blog
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
This says that the storage should be at least of size 1Gi and that the access mode should be ReadWriteOnce.

Kubernetes supports three different access modes for storage.

ReadWriteOnce (RWO) - The volume can be mounted as read/write by a single node.
ReadOnlyMany (ROX) - The volume can be mounted as read-only by many nodes.
ReadWriteMany (RWX) - The volume can be mounted as read/write by many nodes.
What access modes for storage are available will depend on the Kubernetes cluster.

For the front end web application we are using, because we want to be able to run multiple instances, we technically need storage with an access mode that allows it to be mounted on multiple nodes in the Kubernetes cluster. This is because instances of the application could run on different nodes. As such ReadWriteMany should be the requested storage access mode.

At this point you may be confused since the above persistent volume claim actually specifies ReadWriteOnce. For this workshop this is the case as we can't be sure a Kubernetes cluster will have storage of type ReadWriteMany. As a result we cheat. We request storage of type ReadWriteOnce and set a condition on the deployment for the front end to force all pods for the application to be scheduled to the same node, thus allowing us to use ReadWriteOnce. In a real system it is generally regarded as bad practice to force an application when scaled to run on a single node, but we don't have much choice here. For your own application, if it needs to be scaled or use rolling deployments, you should use ReadWriteMany.

In contrast, the database will only ever have one instance and so storage with access mode ReadWriteOnce is sufficient.

In addition to creating the persistent volume claim, the deployment needs to be updated.

At spec.template.spec.volumes in the deployment we need to add:

      volumes:
      - name: media
        persistentVolumeClaim:
          claimName: blog-media
This indicates the persistent volume claim is to be used.

At spec.template.spec.containers.volumeMounts, we neeed to add:

        volumeMounts:
        - name: media
          mountPath: "/opt/app-root/src/media"
This indicates that the persistent volume should be mounted at the path /opt/app-root/src/media in the container.

To see the updated deployment configuration run:

cat frontend-v5/deployment.yaml
To apply the configuration changes run:

kubectl apply -f frontend-v5/
This should output:

deployment.apps/blog configured
ingress.extensions/blog unchanged
persistentvolumeclaim/blog-media created
service/blog unchanged
Both components of the application, the database and the front end web application are fully deployed once again.

18: Workshop Summary
In this workshop you have had an opportunity to get hands on with Kubernetes. In the workshop you deployed an application consisting of a front end web application implementing a blog site, using a back end PostrgreSQL database for storage of posts. For storage of images attached to posts, a persistent volume was used.

For general information on Kubernetes see:

https://kubernetes.io/
If you want to try Kubernetes on your own computer, see Minikube:

https://kubernetes.io/docs/setup/minikube/
If you want to see further simple examples of using Kubernetes:

http://kubernetesbyexample.com/
