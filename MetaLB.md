# 1. Установка MetalLB.
```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.5/config/manifests/metallb-native.yaml
```

Проверяем, что поды поднялись:
```bash
kubectl get pods -n metallb-system
```

Ожидаешь что-то вроде:
```
controller-xxxx   Running
speaker-xxxx      Running
```

# 2. Включаем strictARP (ВАЖНО).

Открываешь configmap:
```bash
kubectl edit configmap kube-proxy -n kube-system
```

Находишь:
```
strictARP: false
```

Меняешь на:
```
strictARP: true
```

Затем перезапускаешь kube-proxy:
```bash
kubectl rollout restart daemonset kube-proxy -n kube-system
```

# 3. Настройка IP-пула.

Пример (подставь свою сеть!)
Создаём файл:
```bash
nano metallb-config.yaml
```

Вставь:
```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.1.240-192.168.1.250
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: l2-advert
  namespace: metallb-system
```

Применяем:
```bash
kubectl apply -f metallb-config.yaml
```

# 4. Проверка.

Создадим тестовый сервис:
```bash
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=LoadBalancer
```

Смотрим:
```bash
kubectl get svc
```

Ты должен увидеть:
```
nginx   LoadBalancer   10.x.x.x   192.168.1.240   80:xxxxx/TCP
```

# 5. Проверка доступа.
```
http://192.168.1.240
```

Должен открыться nginx.
