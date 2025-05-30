# Advanced Programming Rust 
by Andriyo Averill Fahrezi, NPM of 2306172325

## Module 11 - Deployment - Deployment on Kubernetes

### Reflection on Hello Minikube
- Logs Before : 

![Kubectl Logs Before Accessing](src/images/image.png)

- Logs After : 

![Kubectl Logs After Accessing](src/images/image2.png)

**1. Application Logs Comparison**

Before the application was exposed as a service, its logs only showed **initial startup messages**, confirming that its HTTP and UDP servers were listening on their assigned ports.
    
Once the application was exposed as a service and accessed via the proxy, the logs immediately began displaying HTTP **GET requests** to the root path (`/`). Each subsequent interaction, such as reloading or reopening the application, generated additional GET requests. For instance, initial access resulted in two GET requests, and two more reloads added four more, bringing the total to six.
    
This clearly demonstrates that the service successfully **routes external traffic to the application's pod**, with each user interaction accurately reflected in the application's logs.

**2. Two versions of `kubectl get`**

When using `kubectl get`, the `-n` option is crucial for specifying which namespace we want to query.

**How Namespaces Affect `kubectl get`**

-   If we run `kubectl get` without the `-n` option, it will display resources residing in the **default namespace**. This is typically where our pods and services are created if we don't explicitly define a namespace during their creation.

-   However, if we use `kubectl get -n kube-system`, we'll see resources specifically within the `kube-system namespace`. This namespace is home to essential Kubernetes components, such as DNS services, proxies, and other control plane elements that are vital for the cluster's operation. we won't find our own applications' pods or services listed here because they exist in different namespaces. This separation highlights how namespaces provide vital isolation and organization for resources within a Kubernetes cluster.

Commands like `kubectl get deployments` or `kubectl get pods` provide **real-time status** of our resources. To get even more specific information, we can always include the `-n <namespace>` option to target a particular namespace or add `-o wide` for additional details about the resources.

### Reflection on Rolling Update & Kubernetes Manifest File

**1. Differences between Rolling Update and Recreate deployment strategy**

**Rolling Update (Default Strategy)**

-    **Gradual Replacement:** Instead of taking everything down, a rolling update replaces your old pods with new ones gradually, either one by one or in small, controlled batches.
-    **Zero Downtime:** The beauty of this approach is that old pods are only terminated after the new ones are up, running, and confirmed healthy. This ensures your users never experience an interruption in service.
-    **Controlled Pace:** You can fine-tune the speed and risk of the update using parameters like maxSurge (how many new pods can be added above the desired count) and maxUnavailable (how many pods can be unavailable during the update).
-    **Ideal for Production:** This strategy is the go-to for production environments where uptime is absolutely critical, even if it means a slightly slower deployment process.

**Recreate Strategy**

-    **Temporary Downtime:** The most significant difference is that all existing pods are terminated before new ones are created. This inevitably leads to a temporary period of downtime for your application.
-    **Faster, Simpler:** Because it doesn't involve the complex orchestration of a gradual rollout, the recreate strategy is generally faster and simpler to execute.
-    **Resource Efficiency:** It uses fewer resources during the deployment because you're not running both the old and new versions of your application simultaneously.
-    **Best for Development or Tolerant Apps:** This strategy is often preferred in development environments or for applications that can tolerate brief periods of unavailability, where the simplicity and speed outweigh the need for continuous uptime.

**2. Deploying Spring Petclinic REST using Recreate deployment strategy**

I made a new deployment manifest for both deployment and service. I executed it with:
```bash
kubectl apply -f .\deployment_recreate.yaml
kubectl apply -f .\service_recreate.yaml
```
This is the output from the terminal

![image](https://github.com/user-attachments/assets/befd7cf7-a6d7-4ceb-a6f0-1692481df591)

In a recreate deployment, I noticed a distinct sequence of events:

First, all the existing pods were terminated at the same time. This led to a brief period with no pods running, which meant the service was temporarily unavailable during this transition. After the old pods were gone, new pods were then created and started up. When I actually first execute it, it says that only 2 pods are running with 0 deployment is up, this is due to the Container is basically creating and still running. In the end, all pods are up and running.

**3. Recreate Manifest Files**

Here are the manifest files that is different from the ones I used before, this is for the Recreate deployment strategy:

-    `deployment_recreate.yaml`

```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        app: spring-petclinic-rest
      name: spring-petclinic-rest-recreate
    spec:
      replicas: 4
      selector:
        matchLabels:
          app: spring-petclinic-rest
      strategy:
        type: Recreate
      template:
        metadata:
          labels:
            app: spring-petclinic-rest
        spec:
          containers:
          - image: docker.io/springcommunity/spring-petclinic-rest:3.2.1
            name: spring-petclinic-rest
            ports:
            - containerPort: 2325
```

-    `service_recreate.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: spring-petclinic-rest
  name: spring-petclinic-rest-recreate
spec:
  ports:
  - port: 2325
    protocol: TCP
    targetPort: 2325
  selector:
    app: spring-petclinic-rest
  type: LoadBalancer
```

The difference that I made is I changed the port number to the last 4 digits of my NPM.

**4. Why manifest files outperform manual deployment**

**a. Reproducibility & Version Control**

Manifest files enable version control (like Git) for our Kubernetes configurations. This lets us track changes, revert to previous versions, and ensures consistent deployments across all environments, eliminating manual errors.

**b. Automation & CI/CD Integration**

Manifest files are key to automating deployment pipelines. They significantly reduce human error associated with manual `kubectl commands` and are fundamental to GitOps workflows.

**c. Documentation & Infrastructure as Code**

They provide a transparent and auditable record of your deployment configurations. This adherence to Infrastructure as Code (IaC) principles makes it far easier for teams to collaborate and share knowledge, as everyone can see exactly how the application is configured.

**d. Efficiency & Speed**

Deploying complex applications becomes remarkably efficient with manifest files. A single command, `kubectl apply -f`, can deploy an entire application stack. This eliminates the need to remember and type out multiple `kubectl` commands and their various parameters, saving time and preventing typos.

**e. Configuration Management**

Manifest files offer centralized configuration in a declarative format. This makes it incredibly easy to modify and update application settings. They also promote better organization of related resources.

    



