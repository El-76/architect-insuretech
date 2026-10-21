minikube delete

minikube start --addons=metrics-server --extra-config=controller-manager.horizontal-pod-autoscaler-sync-period=5s

minikube kubectl -- get deployment metrics-server -n kube-system

minikube kubectl -- patch deployment metrics-server -n kube-system \
  --type=json \
  -p='[{"op":"replace","path":"/spec/template/spec/containers/0/args/4","value":"--metric-resolution=12s"}]'

minikube kubectl -- create namespace scaletestapp

minikube kubectl -- apply -f deployment.yaml

minikube kubectl -- get pods -n scaletestapp

NAME                            READY   STATUS    RESTARTS   AGE
scaletestapp-67945c59fb-lqsdj   1/1     Running   0          28s

minikube kubectl -- apply -f service.yaml

minikube kubectl -- get services -n scaletestapp

NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
scaletestapp   ClusterIP   10.106.74.228   <none>        8080/TCP   24s

minikube kubectl -- apply -f hpa-memory.yaml

minikube kubectl -- get hpa -n scaletestapp

NAME              REFERENCE                 TARGETS          MINPODS   MAXPODS   REPLICAS   AGE
scaletestapphpa   Deployment/scaletestapp   memory: 8%/80%   1         10        1          59s

# minikube kubectl -- apply -f locust.yaml

# minikube kubectl -- port-forward --address 0.0.0.0 -n locust service/locust-master 8089:8089

minikube kubectl -- port-forward -n scaletestapp --address 127.0.0.1 svc/scaletestapp 31080:8080

minikube dashboard --port=18080 --url

locust

http://192.168.56.106:8089/

5000

5

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus-operator prometheus-community/kube-prometheus-stack

minikube kubectl -- apply -f monitor.yaml

helm install prometheus-adapter prometheus-community/prometheus-adapter -f adapter-values.yaml

minikube kubectl -- get --raw   "/apis/custom.metrics.k8s.io/v1beta1/namespaces/scaletestapp/pods/*/http_requests_per_second" | jq .
{
  "kind": "MetricValueList",
  "apiVersion": "custom.metrics.k8s.io/v1beta1",
  "metadata": {},
  "items": [
    {
      "describedObject": {
        "kind": "Pod",
        "namespace": "scaletestapp",
        "name": "scaletestapp-5d6cd674cd-2hxmf",
        "apiVersion": "/v1"
      },
      "metricName": "http_requests_per_second",
      "timestamp": "2026-10-21T20:53:10Z",
      "value": "0",
      "selector": null
    }
  ]
}

minikube kubectl -- port-forward --address 0.0.0.0 svc/prometheus-operator-kube-p-prometheus 9090:9090

minikube kubectl -- apply -f hpa-rps.yaml

minikube kubectl -- get hpa -n scaletestapp
