1. I don't use Docker Desktop and minikube integration for immature for QEMU on Apple Silicone. 
That is why i'm using colima. Here is sample commands to spin up a cluster.
```
colima start --kubernetes --cpu 4 --memory 12 --disk 10
kubectl config set-context dagster --namespace dagster --cluster colima --user=colima
kubectl config use-context dagster
kubectl create ns dagster
```

2. To deploy dagster we need to run bellow commands:
```
helm repo add dagster https://dagster-io.github.io/helm
helm show values dagster/dagster --version 0.13.1 > values.yaml
helm upgrade --install dagster dagster/dagster -f values.yaml --version 0.13.1
export DAGIT_POD_NAME=$(kubectl get pods --namespace dagster -l "app.kubernetes.io/name=dagster,app.kubernetes.io/instance=dagster,component=dagit" -o jsonpath="{.items[0].metadata.name}")
echo "Visit http://127.0.0.1:8080 to open Dagit"
kubectl --namespace dagster port-forward $DAGIT_POD_NAME 8080:80
```

`values.yaml` is key for configuration. In this file you will be able to control all the config like Image wich will contain all you jobs, secrets and env variables that will be available for container in runtime.


3. Some extra step that nice to have to debug. K8s dashboard:
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.1/aio/deploy/recommended.yaml
kubectl apply -f ./sample_user.yaml
kubectl -n kubernetes-dashboard create token admin-user
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/workloads?namespace=default
```

https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html