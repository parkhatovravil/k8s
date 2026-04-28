# Общая стратегия:
1. Проверка кластера
2. Бэкап control-plane
3. Обновление control-plane
4. Обновление worker-нод по одной (rolling upgrade)
5. Проверка сети (Calico)
6. Финальная валидация

# Pre-check:

На master:
```bash
kubectl get nodes -o wide
kubectl get pods -A
kubectl get componentstatuses
```
Всё должно быть Ready / Running.

Проверка Calico:
```bash
kubectl get pods -n kube-system | grep calico
```

# 1. Бэкап.

Бэкап etcd:
```bash
ETCDCTL_API=3 etcdctl snapshot save /root/etcd-backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

Проверка:
```bash
etcdctl snapshot status /root/etcd-backup.db
```

# 2. Обновление control-plane.

2.1 Обновляем kubeadm:
```bash
apt-mark unhold kubeadm
apt-get update
apt-get install -y kubeadm=1.36.x-00
apt-mark hold kubeadm
```

2.2 Проверяем план:
```bash
kubeadm upgrade plan
```

Убедись, что видит нужную версию, в нашем случае 1.36.

2.3 Применяем апгрейд:
```bash
kubeadm upgrade apply v1.36.x
```

Это обновит:
1. API Server
2. Controller Manager
3. Scheduler
4. etcd (если нужно)

2.4 Обновляем kubelet + kubectl:
```bash
apt-mark unhold kubelet kubectl
apt-get install -y kubelet=1.36.x-00 kubectl=1.36.x-00
apt-mark hold kubelet kubectl
```

```bash
systemctl daemon-reexec
systemctl restart kubelet
```

2.5 Проверка master:
```bash
kubectl get nodes
kubectl get pods -A
```

master должен быть Ready, но версия уже та на которую вы обновляли, в нашем случае 1.36.

3. # Обновление worker-нод (по одной).

3.1 Drain:
```bash
kubectl drain <worker-node> --ignore-daemonsets --delete-emptydir-data
```

3.2 Обновление kubeadm:
```bash
apt-mark unhold kubeadm
apt-get update
apt-get install -y kubeadm=1.36.x-00
apt-mark hold kubeadm
```

3.3 Обновление ноды:
```bash
kubeadm upgrade node
```

3.4 kubelet + kubectl:
```bash
apt-mark unhold kubelet kubectl
apt-get install -y kubelet=1.36.x-00 kubectl=1.36.x-00
apt-mark hold kubelet kubectl
```

```bash
systemctl restart kubelet
```

3.5 Возвращаем ноду:
```bash
kubectl uncordon <worker-node>
```

# 4. Проверка Calico.

После обновления:
```bash
kubectl get pods -n kube-system | grep calico
```

# 5. Финальная проверка

```bash
kubectl get nodes
kubectl version
kubectl get pods -A
```

Дополнительно:
```bash
kubectl get events -A | grep -i error
```