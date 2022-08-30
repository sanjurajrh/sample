 kubectl run webserver \
--image=registry.access.redhat.com/ubi8/httpd-24:1-161

 probes-pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: probes
  labels:
    app: probes
  namespace: user-dev
spec:
  containers:
    - name: probes
      image: 'quay.io/redhattraining/do100-probes:latest'
      ports:
        - containerPort: 8080

kubectl run probes --image=quay.io/redhattraining/do100-probes:latest --dry-run=client -o yaml

kubectl create deployment do100-versioned-hello \
--image quay.io/redhattraining/do100-versioned-hello:v1.0


Procedure 4.1. Instructions

To illustrate how communication is handled in Kubernetes, you use two applications.

The name-generator app produces random names that can be consumed in the /random-name endpoint.

The email-generator app produces random emails that can be consumed in the /random-email endpoint.

The email-generator app consumes name-generator to include a random name in the emails that it generates.

Make sure your kubectl context uses the namespace username-dev. This allows you to execute kubectl commands directly into that namespace.

NOTE
This course uses the backslash character (\) to break long commands. On Linux and macOS, you can use the line breaks.

On Windows, use the backtick character (`) to break long commands. Alternatively, do not break long commands.

Refer to the section called “Orientation to the Classroom Environment” for more information about long commands.

[user@host DO100-apps]$ kubectl config set-context --current \
--namespace=username-dev
Deploy the name-generator app in the username-dev namespace.

Open a command-line terminal. In the DO100-apps repository, navigate to the name-generator folder.

Use the kubectl apply command to create a Deployment from the manifest located in the kubernetes directory. It creates three replicas of the name-generator app by using the quay.io/redhattraining/do100-name-generator:v1.0 image.

[user@host name-generator]$ kubectl apply -f kubernetes/deployment.yml
deployment.apps/name-generator created
List the deployments to verify it has been created successfully. Use the command kubectl get deployment.

[user@host name-generator]$ kubectl get deployment
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
name-generator   3/3     3            3           90m
Create a Service for the deployment of the name-generator app by using the kubectl expose command.

Using the deployment name, expose the service at port number 80. The following command creates a service that forwards requests on port 80 for the DNS name name-generator.namespace.local-domain to containers created by the name-generator deployment on port 8080.

[user@host name-generator]$ kubectl expose deployment name-generator \
--port 80 --target-port=8080
service/name-generator exposed
List the services to verify that the name-generator service has been created successfully. Use the command kubectl get service.

[user@host name-generator]$ kubectl get service
NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
name-generator   ClusterIP   10.98.55.248   <none>        80/TCP    31s
Review the code of the email-generator to see how the request to the name-generator is made. Deploy the app in the username-dev namespace.

In the DO100-apps repository, navigate to the email-generator folder.

In the app directory, open the server.js file. The server.js file is a NodeJS application, which exposes the endpoint /random-email on the 8081 port.

In the same folder, open the generate-email.js file. The generateEmail method generates a random email by making an HTTP request to the name-generator service.

The getNameFromExternalService method performs the actual HTTP request. The host, which is the name-generator service name, is defined in the NAME_GENERATOR_URL variable.

On the command line, return to the email-generator folder.

Apply the Deployment manifest in the username-dev namespace. It is located in the kubernetes folder and creates three replicas of the email-generator app by using the quay.io/redhattraining/do100-email-generator:v1.0 image.

[user@host email-generator]$ kubectl apply -f kubernetes/deployment.yml
deployment.apps/email-generator created
List the deployments in the namespace to verify it has been created successfully. Use the command kubectl get deployment.

[user@host email-generator]$ kubectl get deployment
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
email-generator   3/3     3            3           5s
name-generator    3/3     3            3           66m
Create a service for the deployment of the email-generator app by using a manifest.

Apply the Service manifest in the username-dev namespace. It is located in the kubernetes folder. Use the kubectl apply command.

This command exposes the service in the 80 port and targets port 8081, which is where the email-generator app serves.

[user@host email-generator]$ kubectl apply -f kubernetes/service.yml
service/email-generator created
List the services to verify that the email-generator service has been created successfully. Use the command kubectl get service.

[user@host email-generator]$ kubectl get service
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
email-generator   ClusterIP   10.108.68.139   <none>        80/TCP     3s
name-generator    ClusterIP   10.109.14.167   <none>        80/TCP     68m
Verify that everything works properly by making an HTTP request to the email-generator app from the username-stage namespace. The result should contain a name plus some numbers at the end.

To make a request to the email-generator app from another namespace, you use the Kubernetes DNS resolution pattern service-name.namespace. In this case, the host is email-generator.username-dev.

Create a temporary pod that enables you to make a request to the email-generator application. Run the following command, which provides you with a terminal to execute curl.

[user@host email-generator]$ kubectl run -n username-stage \
curl -it --rm \
--image=registry.access.redhat.com/ubi8/ubi-minimal -- sh
Note that:

The command creates a pod named curl in the username-stage namespace.

The pod contains one container that uses the registry.access.redhat.com/ubi8/ubi-minimal container image.

After Kubernetes creates the pod, you create an interactive remote shell session into the pod.

When you exit out of the interactive session, Kubernetes terminates the pod.

The command might take some time to execute. If you see the message If you don't see a command prompt, try pressing enter., then press Enter on your keyboard and the terminal opens.

In the terminal, make an HTTP request to the email-generator service by using curl. Because the service runs on the default HTTP port (80), you do not need to specify the port. You can also omit the local DNS domain.

[user@host email-generator]$ curl \
 http://email-generator .username-dev/random-email
You should see a response in JSON format similar to this:

{"email":"Susan487@host"}
Type exit to exit the terminal. The pod used to make the request is automatically deleted.

Finish

Remove all resources used in this exercise.

You can delete all resources in the namespace with the following command:

[user@host email-generator]$ kubectl delete all --all
pod "email-generator-ff5fdf658-bz8v2" deleted
pod "email-generator-ff5fdf658-k6ln6" deleted
pod "email-generator-ff5fdf658-pn466" deleted
pod "name-generator-9744675d-4kmp9" deleted
pod "name-generator-9744675d-grw9g" deleted
pod "name-generator-9744675d-tlpz9" deleted
service "email-generator" deleted
service "name-generator" deleted
deployment.apps "email-generator" deleted
deployment.apps "name-generator" deleted
Alternatively, you can delete the resources individually. Delete both the email-generator and name-generator services:

[user@host email-generator]$ kubectl delete service email-generator
service "email-generator" deleted
[user@host email-generator]$ kubectl delete service name-generator
service "name-generator" deleted
Delete both the email-generator and name-generator deployments:

[user@host email-generator]$ kubectl delete deployment email-generator
deployment.apps "email-generator" deleted
[user@host email-generator]$ kubectl delete deployment name-generator
deployment.apps "name-generator" deleted
This concludes the guided exercise.



==========================================================================================================================================================
Guided Exercise: Exposing Applications for External Access: Ingress Resources
In this exercise you will provide external access to a service running inside your Kubernetes cluster.

Outcomes

You should be able to:

Verify that the service IP address and the associated pod IP addresses for an application are not accessible outside of the cluster.

Create an ingress resource to provide external access to an application service.

Confirm that the ingress redirects traffic to the service.

You need a working Kubernetes cluster with the following:

Your kubectl command configured to communicate with the cluster.

An ingress controller enabled in your cluster and the associated domain name mapping.

Your Kubernetes context referring to your cluster and using the username-dev namespace.

Review the section called “Guided Exercise: Contrasting Kubernetes Distributions” for a comprehensive guide to install and enable ingress in your Kubernetes cluster.

Procedure 4.2. Instructions

Deploy a sample hello application. The hello app displays a greeting and its local IP address. When running under Kubernetes, this is the IP address assigned to its pod.

Create a new deployment named hello that uses the container image located at quay.io/redhattraining/do100-hello-ip:v1.0 in the username-dev namespace. Configure the deployment to use three pods.

Create the hello deployment with three replicas. Use the container image located at quay.io/redhattraining/do100-hello-ip:v1.0. This container image simply displays the IP address of its associated pod.

NOTE
This course uses the backslash character (\) to break long commands. On Linux and macOS, you can use the line breaks.

On Windows, use the backtick character (`) to break long commands. Alternatively, do not break long commands.

Refer to the section called “Orientation to the Classroom Environment” for more information about long commands.

[user@host ~]$ kubectl create deployment hello \
--image quay.io/redhattraining/do100-hello-ip:v1.0 \
--replicas 3
deployment.apps/hello created
Run the kubectl get pods -w command to verify that three pods are running. Press Ctrl+C to exit the kubectl command after all three hello pods display the Running status.

[user@host ~]$ kubectl get pods -w
NAME                     READY   STATUS              RESTARTS   ...
hello-5f87ddc987-76hxn   0/1     ContainerCreating   0          ...
hello-5f87ddc987-j8bbv   0/1     ContainerCreating   0          ...
hello-5f87ddc987-ndbk5   0/1     ContainerCreating   0          ...
hello-5f87ddc987-j8bbv   1/1     Running             0          ...
hello-5f87ddc987-ndbk5   1/1     Running             0          ...
hello-5f87ddc987-76hxn   1/1     Running             0          ...
Create a service for the hello deployment that redirects to pod port 8080.

Run the kubectl expose command to create a service that redirects to the hello deployment. Configure the service to listen on port 8080 and redirect to port 8080 within the pod.

[user@host ~]$ kubectl expose deployment/hello --port 8080
service/hello exposed
Verify that Kubernetes created the service:

[user@host ~]$ kubectl get service/hello
NAME    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    ...
hello   ClusterIP   10.103.208.46   <none>        8080/TCP   ...
Note that the IP associated to the service is private to the Kubernetes cluster, You can not access that IP directly.

Create an ingress resource that directs external traffic to the hello service.

Use a text editor to create a file in your current directory named ingress-hello.yml.

Create the ingress-hello.yml file with the following content. Ensure correct indentation (using spaces rather than tabs) and then save the file.

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello
  labels:
    app: hello
spec:
  rules:
    - host: INGRESS-HOST
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: hello
                port:
                  number: 8080
Replace INGRESS-HOST by the host name associated to your Kubernetes cluster, such as hello.example.com or hello-username-dev.apps.sandbox.x8i5.p1.openshiftapps.com. If you are unsure of the host name to use then refer to the section called “Guided Exercise: Contrasting Kubernetes Distributions” to find the appropriate value.

The file at https://github.com/RedHatTraining/DO100-apps/blob/main/network/ingress-hello.yml contains the correct content for the ingress-hello.yml file. You can download the file and use it for comparison.

Use the kubectl create command to create the ingress resource.

[user@host ~]$ kubectl create -f ingress-hello.yml
ingress.networking.k8s.io/hello created
Display information about the hello ingress. If the command does not display an IP address, then wait up to a minute and try running the command again.

[user@host ~]$ kubectl get ingress/hello
NAME    CLASS    HOSTS               ADDRESS        PORTS   ...
hello   <none>   hello.example.com   192.168.64.7   80      ...
The value in the HOST column matches the host line specified in your ingress-hello.yml file. Your IP address is likely different from the one displayed here.

Verify that the ingress resource successfully provides access to the hello service and the pods associated with the service.

Access the domain name specified by your ingress resource.

Use the curl command to access the domain name associated to your ingress controller. Repeat the same command multiple times and note the IP of the responding pod replica varies:

[user@host ~]$ curl hello.example.com
Hello from IP: 172.17.0.5
[user@host ~]$ curl hello.example.com
Hello from IP: 172.17.0.5
[user@host ~]$ curl hello.example.com
Hello from IP: 172.17.0.6
[user@host ~]$ curl hello.example.com
Hello from IP: 172.17.0.6
[user@host ~]$ curl hello.example.com
Hello from IP: 172.17.0.4
[user@host ~]$ curl hello.example.com
Hello from IP: 172.17.0.4
The hello ingress queries the hello service to identify the IP addresses of the pod endpoints. The hello ingress then uses round robin load balancing to spread the requests among the available pods, and each pod responds to the curl command with the pod IP address.

Optionally, open a web browser and navigate to the wildcard domain name. The web browser displays a message similar to the following.

Hello from IP: 172.17.0.6
Refresh you browser window to repeat the request and see different responses.

NOTE
Because load balancers frequently create an association between a web client and a server (one of the hello pods in this case), reloading the web page is unlikely to display a different IP address. This association, sometimes referred to as a sticky session, does not apply to the curl command.

Finish

Delete the created resources that have the app=hello label.

[user@host ~]$ kubectl delete deployment,service,ingress -l app=hello
deployment.apps "hello" deleted
service "hello" deleted
ingress.networking.k8s.io "hello" deleted
Verify that no resources with the app=hello label exist in the current namespace.

[user@host ~]$ kubectl get deployment,service,ingress -l app=hello
No resources found in username-dev namespace.
This concludes the guided exercise.

============================================================================================================================================================================

 kubectl create deployment hello-limit \
  --image quay.io/redhattraining/hello-world-nginx:v1.0 \
  --dry-run=client -o yaml > hello-limit.yaml

Edit the file hello-limit.yaml to replace the resources: {} line with the highlighted lines below. Ensure that you have proper indentation before saving the file.

...output omitted...
    spec:
      containers:
      - image: quay.io/redhattraining/hello-world-nginx:v1.0
        name: hello-world-nginx
        resources:
          requests:
            cpu: "8"
            memory: 20Mi
status: {}

 kubectl create --save-config -f hello-limit.yaml

kubectl get events --field-selector type=Warning

Edit the hello-limit.yaml file to request 1.2 CPUs for the container. Change the cpu: "8" line to match the highlighted line below.

...output omitted...
        resources:
          requests:
            cpu: "1200m"
            memory: 20Mi


 kubectl apply -f hello-limit.yaml
==================================================================================================================================================================================


Guided Exercise: Proving Liveness, Readiness and Startup
In this exercise, you will configure liveness and readiness probes to monitor the health of an application deployed to your Kubernetes cluster.

The application you deploy in this exercise exposes two HTTP GET endpoints:

The /healthz endpoint responds with a 200 HTTP status code when the application pod can receive requests.

The endpoint indicates that the application pod is healthy and reachable. It does not indicate that the application is ready to serve requests.

The /ready endpoint responds with a 200 HTTP status code if the overall application works.

The endpoint indicates that the application is ready to serve requests.

In this exercise, the /ready endpoint responds with the 200 HTTP status code when the application pod starts. The /ready endpoint responds with the 503 HTTP status code for the first 30 seconds after deployment to simulate slow application startup.

You will configure the /healthz endpoint for the liveness probe, and the /ready endpoint for the readiness probe.

You will simulate network failures in your Kubernetes cluster and observe behavior in the following scenarios:

The application is not available.

The application is available but cannot reach the database. Consequently, it cannot serve requests.

Outcomes

You should be able to:

Configure readiness and liveness probes for an application from the command line.

Locate probe failure messages in the event log.

You need a working Kubernetes cluster, and your kubectl command must be configured to communicate with the cluster.

Make sure your kubectl context refers to a namespace where you have enough permissions, usually username-dev or username-stage. Use the kubectl config set-context --current --namespace=namespace command to switch to the appropriate namespace.

Procedure 5.2. Instructions

Deploy the do100-probes sample application to the Kubernetes cluster and expose the application.

Create a new deployment by using kubectl.

NOTE
This course uses the backslash character (\) to break long commands. On Linux and macOS, you can use the line breaks.

On Windows, use the backtick character (`) to break long commands. Alternatively, do not break long commands.

Refer to the section called “Orientation to the Classroom Environment” for more information about long commands.

[user@host ~]$ kubectl create deployment do100-probes \
--image quay.io/redhattraining/do100-probes:latest
deployment.apps/do100-probes created
Expose the deployment on port 8080.

[user@host ~]$ kubectl expose deployment/do100-probes --port 8080
service/do100-probes exposed
Use a text editor to create a file in your current directory called probes-ingress.yml.

Create the probes-ingress.yml file with the following content. Ensure correct indentation (using spaces rather than tabs) and then save the file.

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: do100-probes
  labels:
    app: do100-probes
spec:
  rules:
    - host: INGRESS-HOST
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: do100-probes
                port:
                  number: 8080
Replace INGRESS-HOST with the hostname associated with your Kubernetes cluster, such as hello.example.com or do100-probes-USER-dev.apps.sandbox.x8i5.p1.openshiftapps.com. If you are unsure of the hostname to use then refer to the section called “Guided Exercise: Contrasting Kubernetes Distributions” to find the appropriate value.

The file at https://github.com/RedHatTraining/DO100-apps/blob/main/probes/probes-ingress.yml contains the correct content for the probes-ingress.yml file. You can download the file and use it for comparison.

Use the kubectl create command to create the ingress resource.

[user@host ~]$ kubectl create -f probes-ingress.yml
ingress.networking.k8s.io/do100-probes created
Manually test the application's /ready and /healthz endpoints.

Display information about the do100-probes ingress. If the command does not display an IP address, then wait up to a minute and try running the command again.

[user@host ~]$ kubectl get ingress/do100-probes
NAME           CLASS   HOSTS               ADDRESS        PORTS   ...
do100-probes   nginx   hello.example.com   192.168.49.2   80      ...
The value in the HOST column matches the host line specified in your probes-ingress.yml file. Your IP address is likely different from the one displayed here.

Test the /ready endpoint:

[user@host ~]$ curl -i hello.example.com/ready
On Windows, remove -i flag:

[user@host ~]$ curl hello.example.com/ready
The /ready endpoint simulates a slow startup of the application, and so for the first 30 seconds after the application starts, it returns an HTTP status code of 503, and the following response:

HTTP/1.1 503 Service Unavailable
...output omitted...
Error! Service not ready for requests...
After the application has been running for 30 seconds, it returns:

HTTP/1.1 200 OK
...output omitted...
Ready for service requests...
Test the /healthz endpoint of the application:

[user@host ~]$ curl -i hello.example.com/healthz
HTTP/1.1 200 OK
...output omitted...
OK
On Windows, remove the -i flag:

[user@host ~]$ curl hello.example.com/healthz
Test the application response:

[user@host ~]$ curl hello.example.com
Hello! This is the index page for the app.
Activate readiness and liveness probes for the application.

Use the kubectl edit command to edit the deployment definition and add readiness and liveness probes.

For the liveness probe, use the /healthz endpoint on the port 8080.

For the readiness probe, use the /ready endpoint on the port 8080.

For both probes:

Configure an initial delay of 2 seconds.

Configure the timeout as 2 seconds.

[user@host ~] kubectl edit deployment/do100-probes
...output omitted...
This command opens your default system editor. Make changes to the definition so that it displays as follows.

...output omitted...
spec:
  ...output omitted...
  template:
    ...output omitted...
    spec:
      containers:
      - image: quay.io/redhattraining/do100-probes:latest
        ...output omitted...
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 2
          timeoutSeconds: 2
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 2
          timeoutSeconds: 2
WARNING
The YAML resource is space sensitive. Use spaces to preserve the spacing.

Do not use the tab character to edit the preceding deployment.

Save and exit the editor to apply your changes.

Verify the value in the livenessProbe and readinessProbe entries:

[user@host DO288-apps]$ kubectl describe deployment do100-probes
...output omitted...
    Liveness:     http-get  http://:8080/healthz  delay=2s timeout=2s period=10s #success=1 #failure=3
    Readiness:    http-get  http://:8080/ready  delay=2s timeout=2s period=10s #success=1 #failure=3
...output omitted...
Wait for the application pod to redeploy and change into the READY state:

[user@host ~]$ kubectl get pods
NAME                            READY   STATUS    RESTARTS      AGE
...output omitted...
do100-probes-7794c5cb4f-vwl4x   0/1     Running   0             6s
The READY status shows 0/1 if the AGE value is less than approximately 30 seconds. After that, the READY status is 1/1. Note the pod name for the following steps.

[user@host ~]$ kubectl get pods
NAME                            READY   STATUS    RESTARTS      AGE
...output omitted...
do100-probes-7794c5cb4f-vwl4x   1/1     Running   0             62s
Use the kubectl logs command to see the results of the liveness and readiness probes. Use the pod name from the previous step.

[user@host ~]$ kubectl logs -f do100-probes-7794c5cb4f-vwl4x
...output omitted...
nodejs server running on  http://0.0.0.0:8080 
ping /healthz => pong [healthy]
ping /ready => pong [notready]
ping /healthz => pong [healthy]
ping /ready => pong [notready]
ping /healthz => pong [healthy]
ping /ready => pong [ready]
...output omitted...
Observe that the readiness probe fails for about 30 seconds after redeployment, and then succeeds. Recall that the application simulates a slow initialization of the application by forcibly setting a 30-second delay before it responds with a status of ready.

Do not terminate this command. You will continue to monitor the output of this command in the next step.

Simulate a network failure.

In case of a network failure, a service becomes unresponsive. This means both the liveness and readiness probes fail.

Kubernetes can resolve the issue by recreating the container on a different node.

In a different terminal window or tab, run the following commands to simulate a liveness probe failure:

[user@host ~]$ curl  http://hello.example.com/flip?op=kill-health
Switched app state to unhealthy...
[user@host ~]$ curl  http://hello.example.com/flip?op=kill-ready
Switched app state to not ready...
Return to the terminal where you are monitoring the application deployment:

[user@host ~]$ kubectl logs -f do100-probes-7794c5cb4f-vwl4x
...output omitted...
Received kill request for health probe.
Received kill request for readiness probe.
...output omitted...
ping /ready => pong [notready]
ping /healthz => pong [unhealthy]
...output omitted...
Received kill request for health probe.
...output omitted...
Received kill request for readiness probe.
...output omitted...
Kubernetes restarts the pod when the liveness probe fails repeatedly (three consecutive failures by default). This means Kubernetes restarts the application on an available node not affected by the network failure.

You see this log output only when you immediately check the application logs after you issue the kill request. If you check the logs after Kubernetes restarts the pod, then the logs are cleared and you only see the output shown in the next step.

Verify that Kubernetes restarts the unhealthy pod. Keep checking the output of the kubectl get pods command. Observe the RESTARTS column and verify that the count is greater than zero. Note the name of the new pod.

[user@host ~]$ kubectl get pods
NAME                           READY   STATUS    RESTARTS      AGE
do100-probes-95758759b-4cm2j   1/1     Running   1 (11s ago)   62s
Review the application logs. The liveness probe succeeds and the application reports a healthy state.

[user@host ~]$ kubectl logs -f do100-probes-95758759b-4cm2j
...output omitted...
ping /ready => pong [ready]
ping /healthz => pong [healthy]
...output omitted...
Finish

Delete the deployment, ingress, and service resources to clean your cluster. Kubernetes automatically deletes the associated pods.

[user@host ~]$ kubectl delete deployment do100-probes
deployment.apps "do100-versioned-hello" deleted
[user@host ~]$ kubectl delete service do100-probes
service "do100-probes" deleted
[user@host ~]$ kubectl delete ingress do100-probes
ingress.networking.k8s.io "do100-probes" deleted
This concludes the guided exercise.

=============================================================================================================================================================================
Guided Exercise: Configuring Cloud Applications
In this exercise, you will use configuration maps and secrets to externalize the configuration for a containerized application.

Outcomes

You should be able to:

Deploy a simple Node.js-based application that prints configuration details from environment variables and files.

Inject configuration data into the container using configuration maps and secrets.

Change the data in the configuration map and verify that the application picks up the changed values.

Ensure that:

Minikube and kubectl are running on your machine

You have cloned the DO100-apps repository

You have executed the setup script located at DO100-apps/setup/operating-system/setup.sh

Make sure your kubectl context uses the namespace username-dev. This allows you to execute kubectl commands directly into that namespace.

[user@host ~]$ kubectl config set-context --current --namespace=username-dev
Procedure 5.3. Instructions

Review the application source code and deploy the application.

Enter your local clone of the DO100-apps Git repository.

[user@host ~]$ cd DO100-apps
Inspect the DO100-apps/app-config/app/app.js file.

The application reads the value of the APP_MSG environment variable and prints the contents of the /opt/app-root/secure/myapp.sec file:

const response = Value in the APP_MSG env var is => ${process.env.APP_MSG}\n;
...output omitted...
// Read in the secret file
fs.readFile('/opt/app-root/secure/myapp.sec', 'utf8', function (secerr,secdata) {
...output omitted...
Create a new deployment called app-config using the DO100-apps/app-config/kubernetes/deployment.yml file.

[user@host DO100-apps]$ kubectl apply -f app-config/kubernetes/deployment.yml
deployment.apps/app-config created
Test the application.

Expose the deployment on port 8080:

[user@host DO100-apps]$ kubectl expose deployment/app-config --port 8080
service/app-config exposed
Modify the app-config/kubernetes/ingress.yml file to contain correct host value for your Kubernetes environment:

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-config
  labels:
    app: app-config
spec:
  rules:
    - host: _INGRESS-HOST_
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app-config
                port:
                  number: 8080
Replace _INGRESS-HOST_ with the hostname associated with your Kubernetes cluster, such as hello.example.com or app-config-USER-dev.apps.sandbox.x8i5.p1.openshiftapps.com. If you are unsure of the hostname to use then refer to the section called “Guided Exercise: Contrasting Kubernetes Distributions” to find the appropriate value.

Create the ingress resource to be able to invoke the service just exposed:

[user@host DO100-apps]$ kubectl create -f app-config/kubernetes/ingress.yml
ingress.networking.k8s.io/app-config created
Invoke the host URL by using the curl command:

[user@host DO100-apps]$ curl hello.example.com
Value in the APP_MSG env var is => undefined
Error: ENOENT: no such file or directory, open '/opt/app-root/secure/myapp.sec'
The undefined value for the environment variable and the ENOENT: no such file or directory error are shown because neither the environment variable nor the file exists in the container.

Create the configuration map and secret resources.

Create a configuration map resource to hold configuration variables that store plain text data.

Create a new configuration map resource called appconfmap. Store a key called APP_MSG with the value Test Message in this configuration map:

NOTE
This course uses the backslash character (\) to break long commands. On Linux and macOS, you can use the line breaks.

On Windows, use the backtick character (`) to break long commands. Alternatively, do not break long commands.

Refer to the section called “Orientation to the Classroom Environment” for more information about long commands.

[user@host DO100-apps]$ kubectl create configmap appconfmap \
--from-literal APP_MSG="Test Message"
configmap/appconfmap created
Verify that the configuration map contains the configuration data:

[user@host DO100-apps]$ kubectl describe cm/appconfmap
Name:		appconfmap
...output omitted...
Data
====
APP_MSG:
---
Test Message
...output omitted...
Review the contents of the DO100-apps/app-config/app/myapp.sec file:

username=user1
password=pass1
salt=xyz123
Create a new secret to store the contents of the myapp.sec file.

[user@host DO100-apps]$ kubectl create secret generic appconffilesec \
--from-file app-config/app/myapp.sec
secret/appconffilesec created
Verify the contents of the secret. Note that the contents are stored in base64-encoded format:

[user@host DO100-apps]$ kubectl get secret/appconffilesec -o json
{
    "apiVersion": "v1",
    "data": {
        "myapp.sec": "dXNlcm5hbWU9dXNlcjEKcGFzc3dvcmQ9cGFzczEKc2...
    },
    "kind": "Secret",
    "metadata": {
        ...output omitted...
        "name": "appconffilesec",
        ...output omitted...
    },
    "type": "Opaque"
}
Inject the configuration map and the secret into the application container.

Use the kubectl set env command to add the configuration map to the deployment configuration:

[user@host DO100-apps]$ kubectl set env deployment/app-config \
--from configmap/appconfmap
deployment.apps/app-config env updated
Use the kubectl patch command to add the secret to the deployment configuration:

Patch the app-config deployment using the following patch code. You can find this content in the DO100-apps/app-config/kubernetes/secret.yml file.

[user@host DO100-apps]$ kubectl patch deployment app-config \
--patch-file app-config/kubernetes/secret.yml
deployment.apps/app-config patched
Verify that the application is redeployed and uses the data from the configuration map and the secret.

Verify that the configuration map and secret were injected into the container. Retest the application using the route URL:

[user@host DO100-apps]$ curl hello.example.com
Value in the APP_MSG env var is => Test Message
The secret is => username=user1
password=pass1
salt=xyz123
Kubernetes injects the configuration map as an environment variable and mounts the secret as a file into the container. The application reads the environment variable and file and then displays its data.

Finish

Delete the created resources to clean your cluster. Kubernetes automatically deletes the associated pods.

[user@host ~]$ kubectl delete all,ingress -l app=app-config
pod "app-config-5cb9674bc5-wktrj" deleted
service "app-config" deleted
deployment.apps "app-config" deleted
ingress.networking.k8s.io "app-config" deleted
[user@host ~]$ kubectl delete cm appconfmap
configmap "appconfmap" deleted
[user@host ~]$ kubectl delete secret appconffilesec
secret "appconffilesec" deleted
This concludes the guided exercise
====================================================================================================================================================================================================

 strategy:
    type: RollingUpdate               1
    rollingUpdate:                    2
      maxSurge: 50%                   3
      maxUnavailable: 10%             4

1 Defining the RollingUpdate strategy for the hello deployment.

2 The RollingUpdate strategy accepts the rollingUpdate object to configure further strategy parameters.

3 The maxSurge parameter sets the maximum number of pods that can be scheduled above the desired number of pods. This deployment configures 4 pods. Consequently, 2 new pods can be created at a time.

4 The maxUnavailable parameter sets the maximum number of pods that can be unavailable during the update. Kubernetes calculates the absolute number from the configured percentage by rounding down. Consequently, maxUnavailable is set to 0 with the current deployment parameters.


====================================================================================================================================================================================================
Guided Exercise: Implementing Cloud Deployment Strategies
In this exercise you will deploy a managed containerized application in your Kubernetes cluster. You will observe how some of Kubernetes' automatic deployments and high availability features work.

Outcomes

You should be able to:

Deploy an application container with several replicas.

Review the structure of the Deployment resource manifest.

Update the application to a new version without losing availability.

You need a working Kubernetes cluster, and your kubectl command must be configured to communicate with the cluster.

Make sure your kubectl context refers to a namespace where you have enough permissions, usually username-dev or username-stage. Use the kubectl config set-context --current --namespace=namespace command to switch to the appropriate namespace.

Procedure 6.1. Instructions

Deploy a Node.js application container to your Kubernetes cluster.

Use the kubectl create command to create a new application with the following parameters:

NOTE
This course uses the backslash character (\) to break long commands. On Linux and macOS, you can use the line breaks.

On Windows, use the backtick character (`) to break long commands. Alternatively, do not break long commands.

Refer to the section called “Orientation to the Classroom Environment” for more information about long commands.

[user@host ~]$ kubectl create deployment do100-multi-version \
--replicas=5 \
--image quay.io/redhattraining/do100-multi-version:v1
deployment.apps/do100-multi-version created
Wait until the pod is deployed. The pod should be in the READY state.

[user@host ~]$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS      AGE
do100-multi-version-788cb59f94-54lcz   1/1     Running   0               4s
do100-multi-version-788cb59f94-cv4dd   1/1     Running   0               4s
do100-multi-version-788cb59f94-nt2lh   1/1     Running   0               4s
do100-multi-version-788cb59f94-snx4f   1/1     Running   0               4s
do100-multi-version-788cb59f94-x7k7n   1/1     Running   0               4s
Note that the exact names of your pods will likely differ from the previous example.

Review the application logs to see the running version.

[user@host ~] kubectl logs deploy/do100-multi-version
Found 5 pods, using pod/do100-multi-version-788cb59f94-54lcz

> multi-version@1.0.0 start /opt/app-root/src
> node app.js

do100-multi-version server running version 1.0 on  http://0.0.0.0:8080
Note the version number that the application logs.

Edit the deployment to change the application version and add a readiness probe.

Verify that the deployment strategy for the application is RollingUpdate:

[user@host ~]$ kubectl describe deploy/do100-multi-version
...output omitted...
StrategyType:           RollingUpdate
...output omitted...
Use kubectl edit to modify the deployment resource.

[user@host ~] kubectl edit deployment/do100-multi-version
...output omitted...
Update the version of the image to v2. Additionally, configure a readiness probe so that you can watch the new deployment as it happens.

Your deployment resource should look like the following:

...output omitted...
spec:
  ...output omitted...
  template:
    ...output omitted...
    spec:
      containers:
      - image: quay.io/redhattraining/do100-multi-version: v2
        ...output omitted...
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 2
          timeoutSeconds: 2
When you are done, save your changes and close the editor.

Verify that the new version of the application is deployed via the rolling deployment strategy.

Watch the pods as Kubernetes redeploys the application.

[user@host ~]$ kubectl get pods -w
NAME                                   READY   STATUS              RESTARTS   AGE
do100-multi-version-788cb59f94-54lcz   1/1     Running             0          5m28s
...output omitted...
do100-multi-version-8477f6f4bb-lnlcz   0/1     ContainerCreating   0          3s
...output omitted...
do100-multi-version-8477f6f4bb-lnlcz   0/1     Running             0          5s
do100-multi-version-8477f6f4bb-lnlcz   1/1     Running             0          40s
...output omitted...
do100-multi-version-788cb59f94-54lcz   0/1     Terminating         0          6m8s
...output omitted...
As the new application pods start and become ready, pods running the older version are terminated. Note that the application takes about thirty seconds to enter the ready state.

Press Ctrl+c to stop the command.

View the logs of the new version of the application.

[user@host ~] kubectl logs deploy/do100-multi-version
Found 5 pods, using pod/do100-multi-version-8477f6f4bb-9h5xl

> multi-version@1.0.0 start /opt/app-root/src
> node app.js

do100-multi-version server running version 2.0 on  http://0.0.0.0:8080
Finish

Delete the deployment to clean your cluster. Kubernetes automatically deletes the associated pods.

[user@host ~]$ kubectl delete deploy/do100-multi-version
deployment.apps "do100-multi-version" deleted
This concludes the guided exercise.
==================================================================================================================================================================================================

