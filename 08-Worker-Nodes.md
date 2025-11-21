# Węzły worker

W tym module przygotujemy **węzły robocze (worker nodes)**, czyli maszyny, na których uruchamiane są kontenery z aplikacjami.  
Zainstalujemy i skonfigurujemy:

- `runc` – domyślny runtime kontenerów,
- `gVisor` – sandboxowany runtime dla zaufania ograniczonego,
- wtyczki **CNI** (Container Networking Interface),
- `containerd` – demon odpowiedzialny za zarządzanie kontenerami,
- `kubelet` – agent Kubernetes na każdym węźle,
- `kube-proxy` – komponent odpowiedzialny za routing i reguły sieciowe.

Wszystkie kroki należy wykonać **na każdym węźle typu worker** – w naszym labie: `worker01` i `worker02`.  
Jeśli dodasz kolejne węzły, wykonaj te same instrukcje również na nich.

---

## Punkt wyjścia

Na tym etapie Control Plane i etcd są już uruchomione. Czas dołożyć do tego prawdziwe mięso – węzły, które będą uruchamiały nasze workloady.

---

# Zależności i ustawienia systemu

Poniższe polecenia wykonujemy na **każdym** węźle worker.

---

## 1. Pakiety systemowe

Doinstaluj brakujące pakiety:

```bash
sudo apt-get update
sudo apt-get -y install socat conntrack ipset
```

- `socat` – pomocny przy tunelowaniu ruchu sieciowego,
- `conntrack` i `ipset` – narzędzia wymagane do obsługi zaawansowanych reguł sieciowych (iptables, kube-proxy).

---

## 2. Wyłączenie swap

Kubelet wymaga wyłączonego swapa (stabilność i przewidywalność planowania zasobów).

Na każdym workerze:

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

Drugie polecenie komentuje wpis swapa w `/etc/fstab`, aby nie włączał się przy restarcie systemu.

---

# Instalacja komponentów

## 1. Pobranie binariów

```bash
wget -q --show-progress --https-only --timestamping \
  https://github.com/kubernetes-incubator/cri-tools/releases/download/v1.0.0-beta.0/crictl-v1.0.0-beta.0-linux-amd64.tar.gz \
  https://storage.googleapis.com/kubernetes-the-hard-way/runsc \
  https://github.com/opencontainers/runc/releases/download/v1.0.0-rc5/runc.amd64 \
  https://github.com/containernetworking/plugins/releases/download/v0.6.0/cni-plugins-amd64-v0.6.0.tgz \
  https://github.com/containerd/containerd/releases/download/v1.1.0/containerd-1.1.0.linux-amd64.tar.gz \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kubelet
```

---

## 2. Tworzenie katalogów

Przygotowujemy katalogi dla CNI, kubeleta, kube-proxy i Kubernetes:

```bash
sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

---

## 3. Instalacja binariów

```bash
chmod +x kubectl kube-proxy kubelet runc.amd64 runsc
sudo mv runc.amd64 runc
sudo mv kubectl kube-proxy kubelet runc runsc /usr/local/bin/
sudo tar -xvf crictl-v1.0.0-beta.0-linux-amd64.tar.gz -C /usr/local/bin/
sudo tar -xvf cni-plugins-amd64-v0.6.0.tgz -C /opt/cni/bin/
sudo tar -xvf containerd-1.1.0.linux-amd64.tar.gz -C /
```

Po tym kroku:

- `containerd` jest zainstalowany w systemie,
- wtyczki CNI są w `/opt/cni/bin`,
- `kubectl`, `kubelet`, `kube-proxy`, `runc` i `runsc` są dostępne w `PATH`.

---

# CNI Networking

CNI odpowiada za konfigurację interfejsów sieciowych oraz podsieć dla podów na węzłach.

## 1. Konfiguracja bridge CNI

```bash
cat <<EOF | sudo tee /etc/cni/net.d/10-bridge.conf
{
    "cniVersion": "0.3.1",
    "name": "bridge",
    "type": "bridge",
    "bridge": "cnio0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "ranges": [
          [{"subnet": "10.20.0.0/24"}]
        ],
        "routes": [{"dst": "0.0.0.0/0"}]
    }
}
EOF
```

Ta konfiguracja:

- tworzy most (`bridge`) o nazwie `cnio0`,
- przydziela podom adresy z podsieci `10.20.0.0/24`,
- ustawia bramkę i trasę domyślną.

---

## 2. Konfiguracja loopback

```bash
cat <<EOF | sudo tee /etc/cni/net.d/99-loopback.conf
{
    "cniVersion": "0.3.1",
    "type": "loopback"
}
EOF
```

Loopback CNI zapewnia, że pody mają poprawnie skonfigurowany interfejs `lo`.

---

# Containerd

`containerd` jest demonem odpowiedzialnym za zarządzanie cyklem życia kontenerów – kubelet komunikuje się z nim poprzez CRI.

## 1. Katalog konfiguracyjny

```bash
sudo mkdir -p /etc/containerd/
```

---

## 2. Konfiguracja containerd

```bash
cat << EOF | sudo tee /etc/containerd/config.toml
[plugins]
  [plugins.cri.containerd]
    snapshotter = "overlayfs"
    [plugins.cri.containerd.default_runtime]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runc"
      runtime_root = ""
    [plugins.cri.containerd.untrusted_workload_runtime]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runsc"
      runtime_root = "/run/containerd/runsc"
EOF
```

Powyżej:

- `default_runtime` używa `runc` do zwykłych kontenerów,
- `untrusted_workload_runtime` wykorzystuje `runsc` (gVisor) dla mniej zaufanych obciążeń.

---

## 3. Jednostka systemd dla containerd

```bash
cat <<EOF | sudo tee /etc/systemd/system/containerd.service
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target

[Service]
ExecStartPre=/sbin/modprobe overlay
ExecStart=/bin/containerd
Restart=always
RestartSec=5
Delegate=yes
KillMode=process
OOMScoreAdjust=-999
LimitNOFILE=1048576
LimitNPROC=infinity
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF
```

---

# Kubelet

`kubelet` to agent działający na każdym węźle worker, odpowiedzialny za uruchamianie podów oraz raportowanie stanu węzła do API Servera.

## 1. Kopiowanie certyfikatów i kubeconfigu

Na każdym workerze:

```bash
cd /home/ubuntu/
sudo mv ${HOSTNAME}-key.pem ${HOSTNAME}.pem /var/lib/kubelet/
sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
sudo mv ca.pem /var/lib/kubernetes/
```

Zakładamy, że nazwa hosta (`${HOSTNAME}`) odpowiada nazwie węzła (np. `worker01`).

---

## 2. Konfiguracja kubeleta

```bash
cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.32.0.10"
podCIDR: "10.0.0.0/24"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/${HOSTNAME}.pem"
tlsPrivateKeyFile: "/var/lib/kubelet/${HOSTNAME}-key.pem"
EOF
```

---

## 3. Jednostka systemd dla kubelet

```bash
cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --container-runtime=remote \\
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

---

# Kubernetes Proxy

`kube-proxy` zarządza regułami sieciowymi (np. iptables), dzięki którym ruch do usług Kubernetes (Services) trafia do właściwych podów.

## 1. Przekazanie kubeconfigu

```bash
cd /home/ubuntu
sudo mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

---

## 2. Konfiguracja kube-proxy

```bash
cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "10.0.0.0/24"
EOF
```

---

## 3. Jednostka systemd dla kube-proxy

```bash
cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

---

# Uruchomienie serwisów

Na każdym węźle worker:

```bash
sudo systemctl daemon-reload
sudo systemctl enable containerd kubelet kube-proxy
sudo systemctl start containerd kubelet kube-proxy
```

---

# Weryfikacja

Na węźle `master01` sprawdzamy, czy węzły poprawnie zarejestrowały się w klastrze:

```bash
kubectl get nodes --kubeconfig admin.kubeconfig
```

Jeśli wszystko się udało, zobaczysz `worker01`, `worker02` ze statusem `Ready` (po krótkiej chwili od startu).

---

# Podsumowanie

To był solidny kawał roboty – ale w tej chwili mamy już kompletne środowisko: Control Plane, etcd i węzły worker.

Kolejny krok to przygotowanie **klienta** (wygodnej konfiguracji `kubectl`) na Twojej stacji roboczej:  
https://github.com/inleo-pl/Warsztat-Kubernetes-Fundamentals/blob/master/09-Konfiguracja-klienta.md
