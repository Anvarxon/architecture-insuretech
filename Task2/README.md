# Task2: динамическое масштабирование

## Установка Prometheus и adapter

```powershell
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create namespace monitoring
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring
helm install prometheus-adapter prometheus-community/prometheus-adapter -n monitoring -f Task2/prometheus-adapter-values.yaml
```

## Запуск приложения

```powershell
minikube start --memory=4096 --cpus=2
minikube addons enable metrics-server
kubectl apply -f Task2/deployment.yaml
kubectl apply -f Task2/service.yaml
kubectl apply -f Task2/hpa-memory.yaml
kubectl apply -f Task2/service-monitor.yaml
minikube service scaletestapp --url
```

## Проверка HPA по memory

```powershell
locust -f Task2/locustfile.py --host <minikube-service-url>
kubectl get hpa scaletestapp-memory -w
kubectl get deploy scaletestapp -w
```

## Проверка HPA по RPS

Перед применением `hpa-rps.yaml` удалите HPA по memory, чтобы два HPA не управляли одним Deployment одновременно.

```powershell
kubectl delete hpa scaletestapp-memory
kubectl apply -f Task2/hpa-rps.yaml
kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1/namespaces/default/pods/*/http_requests_per_second" | jq
locust -f Task2/locustfile.py --host <minikube-service-url>
kubectl get hpa scaletestapp-rps -w
```

## Prometheus UI

```powershell
kubectl -n monitoring port-forward svc/prometheus-kube-prometheus-prometheus 9090:9090
```

Запросы для проверки:

```promql
up{service="scaletestapp"}
rate(http_requests_total[1m])
sum(rate(http_requests_total[1m])) by (pod)
```
