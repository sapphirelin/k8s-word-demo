original version [here](https://github.com/gyj0825/k8s-word-demo)

# Kubernetes Words Demo

This demo app included three containers:

- [db](db/Dockerfile) - a Postgres database which stores words （名詞、副詞/動詞、形容詞）

- [words](words/Dockerfile) - a Java REST API which serves words read from the database

- [web](web/Dockerfile) - a Go web application which calls the API and builds words into sentences:


## Before deploy the app to k8s, build images and push to your Registry

Create a Registry Project `k8s-word-demo`

```
$ docker build -t k8s-words-db:v1.0
$ docker build -t k8s-words-api:v1.0
$ docker build -t k8s-words-web:v1.0
$ docker tag k8s-words-db:v1.0 REGISTRY_URL/k8s-word-demo/k8s-words-db:v1.0
$ docker tag k8s-words-api:v1.0 REGISTRY_URL/k8s-word-demo/k8s-words-api:v1.0
$ docker tag k8s-words-web:v1.0 REGISTRY_URL/k8s-word-demo/k8s-words-web:v1.0
$ docker push REGISTRY_URL/k8s-word-demo/k8s-words-db:v1.0
$ docker push REGISTRY_URL/k8s-word-demo/k8s-words-api:v1.0
$ docker push REGISTRY_URL/k8s-word-demo/k8s-words-web:v1.0
```
Remember to change REGISTRY_URL as yours.


## Deploy Using a Kubernetes Manifest

You can deploy this app to Kubernetes by using the [Kubernetes manifest](kube-deployment.yml). It describes the same application in terms of Kubernetes deployments, services and pod specifications.

First, change Registry URL as yours e.g. harbor.example.com
```
$ sed -i 's#REGISTRY_URL#harbor.example.com#g' minio-template.yaml
```

Apply the manifest using `kubectl`:

```shell
$ kubectl apply -f kube-deployment.yml
```

Now browse to http://<Web-External-IP> and you will see the site.

Use `kubectl get svc` to check if the services are up.
you would see the output similar as:
```
$ kubectl get svc
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
words-api    ClusterIP      172.31.0.65      <none>        8080/TCP         4s
words-db     ClusterIP      172.31.0.243     <none>        5432/TCP         4s
words-web    LoadBalancer   172.31.0.58      192.168.0.9   80:30220/TCP     3s
```

Use `kubectl get pods` to check if the pods are running.
You whould see one pod each for the database and web components, and five pods for the words API - which is specified as the replica count in the compose file:

```
$ kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
words-api-9787944d6-f544j    1/1     Running   0          6s
words-api-9787944d6-fdczp    1/1     Running   0          6s
words-api-9787944d6-q4rdc    1/1     Running   0          6s
words-api-9787944d6-qqzb5    1/1     Running   0          6s
words-api-9787944d6-vbcn4    1/1     Running   0          6s
words-db-cc96d879d-fmzqk     1/1     Running   0          6s
words-web-69dbfc57cb-rlprp   1/1     Running   0          6s
```

Open your browser and enter `http://<Web-External-IP>` to see the site. Each time you refresh the page, you'll have a different sentence generated by the API calls.
