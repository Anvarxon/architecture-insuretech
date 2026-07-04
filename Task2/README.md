# Task2: динамическое масштабирование

## Запуск Minikube

```powershell
minikube start --driver=docker --memory=4096 --cpus=2
minikube addons enable metrics-server
```

## Запуск приложения

```powershell
kubectl apply -f Task2/deployment.yaml
kubectl apply -f Task2/service.yaml
minikube service scaletestapp --url
```

## Часть 1: HPA по memory

```powershell
kubectl apply -f Task2/hpa-memory.yaml
locust -f Task2/locustfile.py --host <minikube-service-url>
kubectl get hpa scaletestapp-memory -w
kubectl get deploy scaletestapp -w
```

Подтверждение масштабирования:

- `hpa-memory-scaling.log`
- `task-2-part-1-screenshot-hpa-memory-scaling.png`

## Часть 2: Prometheus и HPA по RPS

```powershell
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create namespace monitoring
helm install prometheus prometheus-community/prometheus -n monitoring `
  --set alertmanager.enabled=false `
  --set prometheus-pushgateway.enabled=false `
  --set kube-state-metrics.enabled=false `
  --set prometheus-node-exporter.enabled=false `
  --set server.persistentVolume.enabled=false
helm install prometheus-adapter prometheus-community/prometheus-adapter -n monitoring -f Task2/prometheus-adapter-values.yaml
```

Перед применением `hpa-rps.yaml` удалите HPA по memory, чтобы два HPA не управляли одним Deployment одновременно.

```powershell
kubectl delete hpa scaletestapp-memory
kubectl apply -f Task2/hpa-rps.yaml
kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1/namespaces/default/pods/*/http_requests_per_second"
locust -f Task2/locustfile.py --host <minikube-service-url>
kubectl get hpa scaletestapp-rps -w
```

Проверка Prometheus:

```powershell
kubectl -n monitoring port-forward svc/prometheus-server 9090:80
```

PromQL-запросы:

```promql
http_requests_total
sum(rate(http_requests_total[5m])) by (namespace,pod)
```

Подтверждение сбора метрик и масштабирования:

- `prometheus-metrics.log`
- `hpa-rps-scaling.log`
- `task-2-part-2-screenshot-prometheus-metrics.png`
- `task-2-part-2-screenshot-hpa-rps-scaling.png`
