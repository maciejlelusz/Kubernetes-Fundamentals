# Uruchomienie etcd

W Kubernetesie wszystkie komponenty kontrolne są stateless – ich konfiguracja i stan klastra są przechowywane w jednym, kluczowym miejscu: **w etcd**.  
To rozproszona, wysoko dostępna baza kluczy-wartości, na której opiera się cały Control Plane.

W środowisku produkcyjnym etcd uruchamia się na kilku węzłach master, aby zapewnić wysoką dostępność (HA).  
W naszym labie dysponujemy jednym masterem, jednak proces wdrożenia pozostaje identyczny dla kolejnych węzłów.

---

## Aktualna architektura

Do tej pory przygotowaliśmy certyfikaty, kubeconfigi oraz szyfrowanie, ale Control Plane wciąż nie działa — brakuje etcd, czyli serca klastra.

---

## Instalacja etcd na węźle master01

Logujemy się na węzeł `master01`, gdzie uruchomimy instancję etcd.

---

### 1. Pobranie binariów etcd

Pobieramy stabilną wersję etcd:

```bash
wget -q --show-progress --https-only --timestamping \
"https://github.com/coreos/etcd/releases/download/v3.3.5/etcd-v3.3.5-linux-amd64.tar.gz"
```

---

### 2. Rozpakowanie archiwum

```bash
tar xvzf etcd-v3.3.5-linux-amd64.tar.gz
```

---

### 3. Instalacja binariów

Przenosimy pliki wykonywalne do standardowej lokalizacji:

```bash
sudo mv etcd-v3.3.5-linux-amd64/etcd* /usr/local/bin/
```

---

### 4. Przygotowanie katalogów

Tworzymy katalogi konfiguracyjne i dane:

```bash
sudo mkdir -p /etc/etcd /var/lib/etcd
```

---

### 5. Kopiowanie certyfikatów

Przenosimy certyfikaty potrzebne do szyfrowanej komunikacji w klastrze etcd:

```bash
cd /home/ubuntu/
sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
```

To zapewnia:

- **TLS między klientami a serwerem etcd**,  
- **TLS między węzłami klastra etcd (peer-to-peer)**.

---

### 6. Definiowanie zmiennych

Ustawiamy podstawowe zmienne identyfikujące instancję:

```bash
ETCD_NAME=$(hostname -s)
INTERNAL_IP=$(hostname -i)
```

- `ETCD_NAME` — nazwa nodu (np. `master01`)
- `INTERNAL_IP` — adres IP nodu w sieci wewnętrznej

---

### 7. Tworzenie usługi systemd dla etcd

Tworzymy plik jednostki `systemd`, który kontroluje pracę etcd:

```bash
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/kubernetes.pem \\
  --peer-key-file=/etc/etcd/kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster ${ETCD_NAME}=https://${INTERNAL_IP}:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### Omówienie kluczowych parametrów:

| Parametr | Znaczenie |
|---------|-----------|
| `--peer-*` | Ustawienia komunikacji w klastrze etcd (HA) |
| `--listen-client-urls` | Adresy, na których etcd przyjmuje zapytania klientów |
| `--advertise-client-urls` | Adresy ogłaszane pozostałym komponentom |
| `--data-dir` | Katalog przechowywania danych etcd |
| `--initial-cluster` | Definicja wszystkich węzłów etcd w klastrze |

---

## 8. Uruchomienie usługi

Ładujemy nową jednostkę systemd, włączamy autostart i uruchamiamy etcd:

```bash
{
  sudo systemctl daemon-reload
  sudo systemctl enable etcd
  sudo systemctl start etcd
}
```

---

## 9. Weryfikacja działania etcd

Sprawdzamy, czy etcd poprawnie działa i jest dostępny:

```bash
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
```

Jeśli konfiguracja została wykonana prawidłowo, powinniśmy zobaczyć listę członków klastra (w naszym labie — jednego).

---

## Podsumowanie

Świetnie! Etcd działa, a nasz klaster zyskuje centralne, bezpieczne miejsce przechowywania stanu.  

Czas przejść do kolejnego kroku i rozpocząć konfigurację **Control Plane**:  
