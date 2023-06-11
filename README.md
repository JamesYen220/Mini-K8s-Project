**1. 部署自己之前的作业到Kubernetes上，需要编写一个deployment的yaml文件**

1. Create a Java Spring Boot application
2. Containerize the application using Docker
3. Push the Docker image to a registry
4. Install and start Minikube
    - brew install minikube
5. Write a Kubernetes deployment YAML file
    - brew install kubectl
6. Deploy the application on Minikube

Let's break down each step:

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
./mvnw package
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

You should see your `hello-world` listed.

Now we need to expose the deployment as a service so we can access it. Run the following command:

```bash
kubectl expose deployment helloworld-deployment --type=NodePort --port=8080
```

This will create a service that exposes our application to external traffic. By specifying `type=NodePort`, Kubernetes will allocate a port on each node for our service.

You can view the service with the following command:

```bash
kubectl get services
```

You should see your `helloworld-deployment` service listed, along with the port it's been assigned on the node.

Finally, to access the application, you can ask Minikube to give you the URL of the service:

```bash
minikube service hello-world --url
```

When you navigate to this URL in a web browser, you should see your "Hello, world!" message.

That's it! You've successfully deployed your Spring Boot "Hello World" application to a Kubernetes cluster using Minikube and Docker. Please let me know if you have any questions or run into any issues!


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
