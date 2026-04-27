1. Установка Calico:
```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

2. Проверка:
```bash
kubectl get pods -n kube-system
```

3. Проверка сети:
```bash
kubectl get nodes
```
