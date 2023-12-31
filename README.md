**1. 部署自己之前的作业到Kubernetes上，需要编写一个deployment的yaml文件**

1. Create a Java Spring Boot application
2. Containerize the application using Docker
3. Push the Docker image to a registry
4. Install and start Minikube
    - brew install minikube
5. Write a Kubernetes deployment YAML file
    - brew install kubectl
6. Deploy the application on Minikube

**Some basic concepts**

Kubectl, Kubernetes, and Minikube are all related to the management and deployment of containerized applications in a Kubernetes environment. Here's an overview of their relationship:

1. Kubernetes: Kubernetes is an open-source container orchestration platform developed by Google. It provides a framework for automating the deployment, scaling, and management of containerized applications across a cluster of machines. Kubernetes abstracts the underlying infrastructure and provides a consistent way to manage and control containers.

2. Minikube: Minikube is a lightweight tool that enables local development and testing of Kubernetes applications. It creates a single-node Kubernetes cluster on a local machine, providing an environment that mimics a full-scale Kubernetes cluster. Minikube is useful for developers who want to experiment with Kubernetes features, test their applications, or learn Kubernetes without the need for a full production cluster.

3. Kubectl: Kubectl is the official command-line tool for interacting with Kubernetes clusters. It acts as a client to the Kubernetes API server and allows users to deploy and manage applications, inspect and modify cluster resources, and view logs and events. Kubectl enables administrators and developers to control and monitor the Kubernetes cluster from the command line or through scripts. Kubectl can be used to monitor and interact with the activity in a Minikube cluster. When you start a Minikube cluster, it creates a single-node Kubernetes cluster on your local machine. You can then use kubectl commands to manage and monitor the resources within that Minikube cluster.

Docker plays a crucial role in the relationship between Kubectl, Kubernetes, and Minikube. Here's how Docker fits into the picture:

1. Docker: Docker is a popular containerization platform that allows you to package applications and their dependencies into containers. Containers are lightweight, isolated environments that encapsulate an application and all its necessary dependencies, enabling consistent deployment and execution across different environments.

2. Kubernetes and Containers: Kubernetes works with containers to manage and orchestrate application deployments. It leverages Docker (or other container runtimes) as the default container runtime to create, run, and manage containers on individual nodes within a Kubernetes cluster. Kubernetes treats containers as the basic units of deployment and scaling.

3. Minikube and Docker: Minikube uses Docker as the default container runtime to create and manage containers within the Minikube cluster. When you start a Minikube cluster, it provisions a single-node Kubernetes cluster, and Docker is used by Minikube to run containers on that node. Minikube interacts with Docker to create and manage the containers required for running Kubernetes applications locally.

4. Kubectl and Containers: Kubectl, as the command-line interface to Kubernetes, interacts with containers indirectly through the Kubernetes API server. When you use kubectl commands to deploy or manage applications, Kubernetes translates those commands into actions on the underlying containers managed by Docker (or the container runtime in use).

In summary, Docker provides the containerization technology that Kubernetes leverages to deploy and manage applications. Minikube, as a local Kubernetes cluster, uses Docker as the container runtime. Kubectl interacts with Kubernetes, which in turn manages the containers using Docker. Docker, therefore, plays a crucial role in containerization, while Kubernetes and Minikube provide the orchestration and management layers for running containers in a scalable and distributed manner.

Learn More: https://www.bilibili.com/video/BV13Q4y1C7hS?p=14&spm_id_from=pageDriver&vd_source=e89b2dd5994f0e82f6841422969e2b5e


**Step 1: Create a Java Spring Boot application**

First, we will create a simple "Hello World" application using Spring Boot and IntelliJ IDEA.

1. Open IntelliJ IDEA, go to "File -> New -> Project".
2. In the new project wizard, select "Spring Initializr". Click "Next".
3. Fill in the "Group" and "Artifact" details as per your requirement. Click "Next".
4. Choose the "Spring Web" dependency, then click "Next" and "Finish".
5. In the generated project, navigate to `src/main/java/<YourProjectName>/DemoApplication.java`.
6. Create a new controller class in the same package. The class should look like this:

```java
package com.james.bookstore.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/")
    public String helloWorld() {
        return "Hello, world!";
    }
}
```

**Step 2: Containerize the application using Docker**

1. Make sure Docker is installed on your machine. You can verify it by running `docker version` in your terminal.
2. In your project directory, create a Dockerfile with the following content:

```Dockerfile
FROM openjdk:11
EXPOSE 8080
ADD target/bookstore-0.0.1-SNAPSHOT.jar bookstore.jar
ENTRYPOINT ["java","-jar","/bookstore.jar"]
```

3. To create the Docker image, first you need to build your application. Navigate to the root directory of your application in terminal and run the following command: 

```bash
mvn package
```

4. After the build is successful, create the Docker image using the following command:

```bash
docker build -t hello-world .
```

**Step 3: Push the Docker image to a registry**

For Minikube to pull the Docker image, it needs to be hosted on a Docker registry. For this guide, we'll use Docker's public registry, Docker Hub.

1. Login to Docker Hub using the command: `docker login`. Provide your Docker Hub username and password.
2. Tag the image with your Docker Hub username: `docker tag hello-world:latest <your-username>/hello-world:latest`.
3. Push the image to Docker Hub: `docker push <your-username>/hello-world:latest`.

**Step 4: Install and start Minikube**

1. Install Minikube based on your operating system. 
2. Start Minikube with the command: `minikube start`.

**Step 5: Write a Kubernetes deployment YAML file**

In your project directory, create a file named `deployment.yaml` with the following content:

The YAML content should look like this:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-world
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
      - name: hello-world
        image: <your-username>/hello-world:latest
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: hello-world-service
spec:
  type: LoadBalancer
  ports:
  - port: 8080
    targetPort: 8080
  selector:
    app: hello-world
```

This YAML file includes a Deployment and a Service. The Deployment specifies that we want one replica of our app running, and it should use the Docker image we pushed to Docker Hub. The Service will expose our application on a LoadBalancer type, which means it will be accessible through a public IP address in a real cloud environment. In Minikube, this will be the IP address of the Minikube virtual machine itself.

**Step 6: Deploy the application on Minikube**

Before we can deploy our application, we need to make sure Minikube can pull the Docker image we created. We can do this by pointing Docker's context to Minikube's Docker daemon with the following command:

```bash
eval $(minikube docker-env)
```

Now we can apply our deployment configuration:

```bash
kubectl apply -f deployment.yaml
```

This will create the deployment in our Kubernetes cluster. You can check the status of the deployment with the following command:

```bash
kubectl get deployments
```
<img width="466" alt="image" src="https://github.com/JamesYen220/K86/assets/100248639/842857eb-1f6f-4e5e-a98b-2dcde940e048">

You should see your `hello-world` listed.


**Deployment or Pods not working**

```bash
kubectl describe deployment helloworld-deployment
```

This will create the deployment in our Kubernetes cluster. You can check the status of the deployment with the following command:

```bash
kubectl get deployments
```
<img width="486" alt="image" src="https://github.com/JamesYen220/K86/assets/100248639/e0d0a4e9-1ac3-4084-b375-45c916324d88">


This command will show you the events and configuration related to your `helloworld-deployment`. Look for any warning or error messages in the output.

You can also check the logs of the pods themselves. First, get the pod names:

```bash
kubectl get pods
```
<img width="767" alt="image" src="https://github.com/JamesYen220/K86/assets/100248639/ef04c585-4432-4e3c-acf0-4224211fc707">


Then, you can get the logs of a specific pod by using its name:

```bash
kubectl logs <pod-name>
```

Replace `<pod-name>` with the name of one of the pods that's not running properly. This should give you more insight into what might be going wrong.


Now we need to expose the deployment as a service so we can access it. Run the following command:

```bash
kubectl expose deployment hello-world --type=NodePort --port=8080
```

This will create a service that exposes our application to external traffic. By specifying `type=NodePort`, Kubernetes will allocate a port on each node for our service.

You can view the service with the following command:

```bash
kubectl get services
```

You should see your `hello-world` service listed, along with the port it's been assigned on the node.
<img width="708" alt="image" src="https://github.com/JamesYen220/K86/assets/100248639/dd4de4c9-6722-49a4-b251-442b2fc7c5d1">


Finally, to access the application, you can ask Minikube to give you the URL of the service:

```bash
minikube service hello-world --url
```

When you navigate to this URL in a web browser, you should see your "Hello, world!" message.

That's it! You've successfully deployed your Spring Boot "Hello World" application to a Kubernetes cluster using Minikube and Docker. Please let me know if you have any questions or run into any issues!

<img width="745" alt="image" src="https://github.com/JamesYen220/K86/assets/100248639/1b7f1219-8015-4969-a17f-21273cf9e6cc">
<img width="1405" alt="image" src="https://github.com/JamesYen220/K86/assets/100248639/5ffdb18f-c039-4978-9719-a50a4524f1c7">


-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


**2. 能够动态扩缩容，在yaml文件里写好相关配置后就能实现**
The requirement you mentioned is to dynamically scale the application, and you want the related configurations to be written in the YAML file.

Kubernetes provides a feature called the Horizontal Pod Autoscaler (HPA) that automatically scales the number of pods in a replication controller, deployment, replica set or stateful set based on observed CPU utilization.

Here's how you can set it up:

1. First, you need to make sure that the metrics server is running in your cluster. The Horizontal Pod Autoscaler uses the metrics server to fetch metrics like CPU utilization. In Minikube, you can enable it with the command `minikube addons enable metrics-server`.

2. Next, you'll need to define a HorizontalPodAutoscaler resource. This can be done in the same YAML file as your deployment, or in a separate file. Here's an example configuration:

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: hello-world-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hello-world
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```

This configuration will create a Horizontal Pod Autoscaler that manages the number of pods in the `hello-world` deployment. The number of pods will be between `minReplicas` and `maxReplicas`, scaling based on CPU utilization. In this case, if the average CPU utilization across all pods exceeds 50%, Kubernetes will start creating new pods. If CPU utilization drops below 50%, it will start removing pods, down to a minimum of 1.

3. You can apply this configuration with `kubectl apply -f hpa.yaml`, if you put it in a separate file named `hpa.yaml`.

4. To check the status of the HPA, you can use the command `kubectl get hpa`.

Remember that HPA is based on the metrics available in your cluster. The example above uses CPU utilization, but Kubernetes can also scale based on memory usage and custom metrics, provided that the metrics are available in your cluster.

Also note that this is a basic example and actual values for `minReplicas`, `maxReplicas` and `averageUtilization` should be chosen based on the requirements of your specific application and environment.

<img width="770" alt="image" src="https://github.com/JamesYen220/K86/assets/100248639/eae039a5-80be-48b5-b5ad-56234b37a12c">

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


**3. 利用Prometheus监控Kubernete以及其上的应用**
Monitoring your Kubernetes cluster and the applications running on it with Prometheus is a great choice. Prometheus, a CNCF project, is a systems and service monitoring system. It collects metrics from configured targets at given intervals, evaluates rule expressions, displays the results, and can trigger alerts if some condition is observed to be true. 

To use Prometheus for monitoring, you'll need to install Prometheus itself, as well as node_exporter for machine metrics, and kube-state-metrics for cluster metrics. You'll also need to configure your applications to expose Prometheus metrics.

Here are the steps to setup Prometheus for Kubernetes:

**Step 1: Install Prometheus**

To install Prometheus, you can create a new namespace and apply the Prometheus operator manifest:


**Create monitoring namespace**

1. **Install Prometheus Operator**: 
    
    ```bash
    kubectl create namespace monitoring

    # Add the prometheus-community Helm repo
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

    # Update the Helm repo
    helm repo update

    # Install Prometheus
    helm install prometheus prometheus-community/prometheus --namespace monitoring
    ```

2. **Verify the installation**: 

    After a while, verify that the Prometheus Operator components are deployed:

    ```
    kubectl get pods -n monitoring
    ```

You should see several pods starting with `prometheus-operator-...` in the "monitoring" namespace. If this works, then you can proceed with the rest of the Prometheus setup.


**Step 2: Create a Prometheus instance**

You can create a Prometheus instance by applying a YAML file with a `Prometheus` resource. Here's an example:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: prometheus
  namespace: monitoring
spec:
  serviceAccountName: prometheus
  serviceMonitorSelector:
    matchLabels:
      team: frontend
  resources:
    requests:
      memory: 400Mi
  enableAdminAPI: false
```

This will create a Prometheus instance that will monitor services labeled with `team: frontend`. You can adjust the `matchLabels` to match your environment.

```shell
kubectl apply -f prometheus-instance.yaml
```

**Step 3: Install node_exporter**

Node_exporter is a Prometheus exporter for machine metrics. You can install it with the following commands:

```bash
brew install helm
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install [RELEASE_NAME] prometheus-community/prometheus-node-exporter
```

In this command, replace `[RELEASE_NAME]` with the name you want to give to this release of the application. If you're running these commands for the first time, Helm will download the chart from the Prometheus community Helm chart repository, update the chart with the latest information from the repository, and then install the application with the name you've specified.

**Step 4: Install kube-state-metrics**

Kube-state-metrics is a service that listens to the Kubernetes API server and generates metrics about the state of the objects. It's not focused on the health of the individual Kubernetes components, but rather on the health of the various objects inside, such as deployments, nodes and pods.

You can install it with the following command:

```yaml
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  labels:  
    app.kubernetes.io/component: exporter  
    app.kubernetes.io/name: kube-state-metrics  
    app.kubernetes.io/version: 2.9.2  
  name: kube-state-metrics  
  namespace: kube-system  
spec:  
  replicas: 1  
  selector:  
    matchLabels:  
      app.kubernetes.io/name: kube-state-metrics  
  template:  
    metadata:  
      labels:  
        app.kubernetes.io/component: exporter  
        app.kubernetes.io/name: kube-state-metrics  
        app.kubernetes.io/version: 2.9.2  
    spec:  
      automountServiceAccountToken: true  
      containers:  
      - image: registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.9.2  
        livenessProbe:  
          httpGet:  
            path: /healthz  
            port: 8080  
          initialDelaySeconds: 5  
          timeoutSeconds: 5  
        name: kube-state-metrics  
        ports:  
        - containerPort: 8080  
          name: http-metrics  
        - containerPort: 8081  
          name: telemetry  
        readinessProbe:  
          httpGet:  
            path: /  
            port: 8081  
          initialDelaySeconds: 5  
          timeoutSeconds: 5  
        securityContext:  
          allowPrivilegeEscalation: false  
          capabilities:  
            drop:  
            - ALL  
          readOnlyRootFilesystem: true  
          runAsNonRoot: true  
          runAsUser: 65534  
          seccompProfile:  
            type: RuntimeDefault  
      nodeSelector:  
        kubernetes.io/os: linux  
      serviceAccountName: kube-state-metrics
```

You can save this content into a file (let's call it `kube-state-metrics.yaml`) and then apply it using the following command:

```shell
kubectl apply -f kube-state-metrics.yaml
```

**Step 5: Expose application metrics**

To monitor your applications with Prometheus, they need to expose a `/metrics` HTTP endpoint that returns current metrics in Prometheus format. Spring Boot applications can use the Actuator's `prometheus` micrometer to expose these metrics. You can add these to your application's properties:

```properties
management.endpoints.web.exposure.include=*
management.endpoint.metrics.enabled=true
management.endpoint.prometheus.enabled=true
management.metrics.export.prometheus.enabled=true
```

You'll also need to add the micrometer dependency to your `pom.xml`:

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-registry-prometheus</artifactId>
            <version>1.7.3</version>
        </dependency>
```

Once your application is exposing metrics, you can add a `ServiceMonitor` to tell Prometheus to scrape metrics from your application:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: hello-world-monitor
  labels:
    team: frontend
spec:
  selector:
    matchLabels:
      app: hello-world
  endpoints:
  - port: web
    path: /actuator/prometheus
```
```shell
kubectl apply -f servicemonitor.yaml
```


**Verifying Your Prometheus Setup**

After setting up Prometheus and configuring your application to expose metrics, you can verify that Prometheus is successfully scraping metrics from your application by accessing the Prometheus dashboard:

1. Run the following command to set up port forwarding:

```bash
kubectl port-forward svc/prometheus-operator-prometheus -n monitoring 9090
```

To check the service name of your Prometheus instance in Kubernetes, you can use the following command:

```
kubectl get services -n monitoring
```

This command will list all the services in the "monitoring" namespace. Look for the Prometheus service in the output. The service name should be displayed in the "NAME" column.

<img width="815" alt="image" src="https://github.com/JamesYen220/K86/assets/100248639/b09b46a7-7cc0-44fb-bf74-5ada4516d49a">

2. Open a web browser and navigate to `http://localhost:9090`.
<img width="1728" alt="image" src="https://github.com/JamesYen220/K86/assets/100248639/47d977a2-f028-4485-80a1-49da8ef441f6">

3. Click on the "Status" dropdown menu and select "Targets". You should see your application listed as a target, with the status "UP".
<img width="1728" alt="image" src="https://github.com/JamesYen220/K86/assets/100248639/fbbc39d0-945e-4461-b640-f270927a7548">
<img width="1721" alt="image" src="https://github.com/JamesYen220/K86/assets/100248639/43650dfc-f414-4349-94a1-c0088109344c">
<img width="1720" alt="image" src="https://github.com/JamesYen220/K86/assets/100248639/311aaa4d-1021-4788-aaf0-ccb39bb228bc">
<img width="1711" alt="image" src="https://github.com/JamesYen220/K86/assets/100248639/911578c9-9ded-40f7-8c61-69f1773b1379">


4. You can also query your application's metrics: type the metric's name into the "Expression" input field and click "Execute".

Remember to replace `prometheus-operator-prometheus` with the service name of your Prometheus instance if you named it differently.
