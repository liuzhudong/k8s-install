# kubernetes 安装文档

```js

cat apt-key.gpg | apt-key add -

sudo add-apt-repository "deb https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial main"

sudo apt-get update

sudo apt-get install -y kubelet kubeadm kubectl

sudo sysctl net.bridge.bridge-nf-call-iptables=1

sed -i "s,ExecStart=$,Environment=\"KUBELET_EXTRA_ARGS=--pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.1\"\nExecStart=,g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

systemctl daemon-reload
systemctl restart kubelet

kubeadm init --config /data/k8s-install/kubeadm.conf

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml


kubeadm join 172.18.173.191:6443 --token 7m97j4.gcc9rxpdne7ul89t --discovery-token-ca-cert-hash sha256:7645bef1ea78a5959b7d673c1b731811d9926b9a946022e26a614f4477adce9b


kubectl drain izwz96d3kxt5vxmdm9439xz --delete-local-data --force --ignore-daemonsets
kubectl delete node izwz96d3kxt5vxmdm9439xz

kubectl drain izwz9h5gdxq9t3kxkamk25z --delete-local-data --force --ignore-daemonsets
kubectl delete node izwz9h5gdxq9t3kxkamk25z



// 查看节点状态
kubectl get nodes

// 查看pods状态
kubectl get pods --all-namespaces
kubectl get pod --namespace=kube-system

// 查看K8S集群状态
kubectl get cs

// 测试 DNS 解析是否正常
kubectl run curl --image=radial/busyboxplus:curl -i --tty
nslookup kubernetes.default

// dashboard YAML
// 官网版
https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml


// https://github.com/gh-Devin/kubernetes-dashboard/blob/master/kubernetes-dashboard.yaml

kubectl create -f kubernetes-dashboard.yaml

http://119.23.74.36:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/

kubectl describe -n kube-system secret/kubernetes-dashboard-certs

wget https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/rbac/heapster-rbac.yaml

wget https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/influxdb/grafana.yaml

wget https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/influxdb/heapster.yaml

wget https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/influxdb/influxdb.yaml

kubernetes-dashboard-admin.rbac.yaml

kubectl create -f kubernetes-dashboard-admin.rbac.yaml

eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbi10b2tlbi1kZHdxdyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjJmZDkyNjVkLTU5YWMtMTFlOC1hNGEyLTAwMTYzZTAyYWE3MiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTprdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbiJ9.non5lmejFdWbKYjmxpgqeA-A1oqBm2jmF0heXKCjCz19wJco03hlplttGb6JNqRquYBmz206V7pOS5CQjQDDaQFJew8_ZyuE_olA8B0nQmOjpqO9WXrQftGe1aqSPcLOvgfF2vPU0XcK8BOC2mMXJkqqj25ocimZNRL7zSSWXNENyx2JQFukcoLTA20YgVCVaX809fbfrBEqZ6JNpZgR0vM7SupvaPe_rG_Jg_ei5vU-2h9ryOhz6hc0-jaFKV68nPVnZCaSOt1xpudeTC-5GjS_ho9lAFqOp1bFrAfkRGUPwODSMlukVjMNlawzz8lnVHNqCxfbUpucTDoS3J7uKA


https://119.23.74.36:6443/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/

http://119.23.74.36:30090



# 生成client-certificate-data
grep 'client-certificate-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d >> kubecfg.crt

# 生成client-key-data
grep 'client-key-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d >> kubecfg.key

# 生成p12
openssl pkcs12 -export -clcerts -inkey kubecfg.key -in kubecfg.crt -out kubecfg.p12 -name "kubernetes-client"

```


k8s 部署：

```js
// Initializing your master
kubeadm init
kubeadm init --config /data/k8s-install/kubeadm.conf

// Tear down
kubectl drain <node name> --delete-local-data --force --ignore-daemonsets
kubectl delete node <node name>

kubeadm reset

// non-root user(非root用户执行)
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

// root user
export KUBECONFIG=/etc/kubernetes/admin.conf

// make a record(记录下来)
// kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
kubeadm join 172.18.173.191:6443 --token ild2t0.iv0cg9d2pe42z7if --discovery-token-ca-cert-hash sha256:6326fbb11696022c92aabf1467ceacc243f53bef1e51db46b756f0077f0604b7


// Installing a pod network(安装网络)
// flannel
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.10.0/Documentation/kube-flannel.yml

// Canal
kubectl apply -f https://raw.githubusercontent.com/projectcalico/canal/master/k8s-install/1.7/rbac.yaml
kubectl apply -f https://raw.githubusercontent.com/projectcalico/canal/master/k8s-install/1.7/canal.yaml

// 查看pods状态
kubectl get pods --all-namespaces

// 测试 DNS 解析是否正常
kubectl run curl --image=radial/busyboxplus:curl -i --tty
nslookup kubernetes.default

// master node参与工作负载
kubectl taint nodes --all node-role.kubernetes.io/master-

// Joining your nodes (在节点机上执行)
// kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>

// 查看节点状态 (master 节点上)
kubectl get nodes

// 查看K8S集群状态
kubectl get cs

```

k8s dashboard 安装：

```js
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml


```

```js

apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.7.9
    ports:
    - containerPort: 80


```

