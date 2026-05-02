# Установка.
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/headlamp/main/kubernetes-headlamp.yaml
```

# Вариант 1 — создать отдельного пользователя.
```bash
kubectl create serviceaccount headlamp-admin -n kube-system
```

Дать ему права:
```bash
kubectl create clusterrolebinding headlamp-admin \
  --clusterrole=cluster-admin \
  --serviceaccount=kube-system:headlamp-admin
```

Получить токен:
```bash
kubectl create token headlamp-admin -n kube-system
```

# Вариант 2 — если уже есть secret.
```bash
kubectl get secret -n kube-system
```

Потом:
```bash
kubectl describe secret <name> -n kube-system
```

или:
```bash
kubectl get secret <name> -n kube-system -o jsonpath="{.data.token}" | base64 -d
```
