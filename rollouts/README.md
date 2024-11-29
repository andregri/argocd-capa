# Argocd Rollouts
https://argo-rollouts.readthedocs.io/en/stable/

A Kubernetes controller and a set of CRDs to perform advanced deployments (blue-green, canary).

Kubernetes provides rolling updates but they have some disadvantages:
- no controls over the speed
- only basic health checks, no stress test checks
- no possibility to rollback

ArgoCD rollouts features:
- blue-green
- canary
- traffic shifting
- automatic rollback and promotion
- manual approval
- customizable metrics

## Get started
https://argo-rollouts.readthedocs.io/en/stable/getting-started/

Install rollouts:
```
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
```

Apply a Rollout resource:
````
kubectl apply -f - <<EOF 
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: rollouts-demo
spec:
  replicas: 5
  strategy:
    canary:
      steps:
      - setWeight: 20
      - pause: {}
      - setWeight: 40
      - pause: {duration: 10}
      - setWeight: 60
      - pause: {duration: 10}
      - setWeight: 80
      - pause: {duration: 10}
  revisionHistoryLimit: 2
  selector:
    matchLabels:
      app: rollouts-demo
  template:
    metadata:
      labels:
        app: rollouts-demo
    spec:
      containers:
      - name: rollouts-demo
        image: argoproj/rollouts-demo:blue
        ports:
        - name: http
          containerPort: 8080
          protocol: TCP
        resources:
          requests:
            memory: 32Mi
            cpu: 5m
EOF
```

Get the rollout:
```
kubectl get rollout rollouts-demo --watch 
```
Output:
```
NAME            DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
rollouts-demo   5         5         5            5           22s
```

Apply a service for the rollout:
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: rollouts-demo
spec:
  ports:
  - port: 80
    targetPort: http
    protocol: TCP
    name: http
  selector:
    app: rollouts-demo
EOF
```

Update the deployment to trigger the rollout:
```
kubectl patch rollout rollouts-demo --type='merge' -p '{"spec":{"template":{"spec":{"containers":[{"name":"rollouts-demo","image":"argoproj/rollouts-demo:yellow"}]}}}}'
```

Get the rollout:
```
kubectl get rollout rollouts-demo --watch 
```
Output:
```
NAME            DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
rollouts-demo   5         5         1            5           10m
```