REST to SOAP Transformation with Apache Camel and Gloo Gateway
==========================================

This example demonstrates how to use Apache Camel to frontend an exisiting legacy SOAP service with a RESTFUL interface.  This can be deployed to any flavor of Kubernetes, in this case, EKS.

## Prerequisites

1. Java 11+
2. Maven 3.8+
3. A running EKS instace

## Build

You can build this example using

```
    mvn package
```

## Run the example

Using the shell:

 1. Start the springboot service:

```
  $ mvn spring-boot:run
```

## Test the example:

Using HTTPie, import the Swagger spec using the following url:

```
     http://localhost:8080/camel/api-doc
```

You can test the transformation using any number you like in the URL:

```bash
curl -X GET "http://localhost:8080/camel/calculator/2/201" -H "accept: application/json" | jq
```

     
## Deploying to Kubernetes

To deploy to AWS EKS, we need to follow these steps.

## Prerequisites

1. Java 11+
2. Docker
3. DockerHub account
4. An AWS account, with
5. aws CLI
6. kubectl

## Method

1. Via the CLI (`/gloo-rest-soap-transformation` directory), run the following commands to build the Docker image locally:

```bash
docker login
mvn clean spring-boot:build-image
```

2. Login to your public DockerHub repo (using the push commands popup), then tag the docker image and push it to the the repo using your correct repo ID:

```bash
docker tag docker.io/library/gloo-rest-soap-transformation:1.0.0 simongreen02/calypso:gloo-rest-soap-transformation
docker push simongreen02/calypso:gloo-rest-soap-transformation
```

3. Update the repo ID in `src/k8s/deployment.yaml` to point to the correct ECR repo:

```yaml
    spec:
      containers:
      - image: simongreen02/calypso:gloo-rest-soap-transformation
        name: gloo-rest-soap-transformation
        resources: {}
status: {}
```

4. Deploy the image and service to gke:

```bash
kubectl apply -f src/k8s/deployment.yaml 
```

5. Run the following command and copy the `LoadBalancer Ingress` IP (or hostname):

```bash
kubectl describe service gloo-rest-soap-transformation
```

6.   Using the above hostname, test the service with the following command:

```bash
curl http://35.226.32.108/camel/api-doc
```

7.  You can import the above URL into Insomnia and test the endpoint using the OpenAPI spec.

You can test the transformation using any number you like in the URL:

```bash
curl -X GET "http://35.226.32.108:8080/camel/numbertowords/122" -H "accept: application/json"
```

8.  If the request is successful, you should receive the following response:

```json
{
  "AddResponse": {
    "AddResult": 203
  }
}
```