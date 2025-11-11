# Konfiguracja HAProxy w środowisku Kubernetes Fundamentals

Ten moduł opisuje proces konfiguracji i uruchomienia **HAProxy** w środowisku laboratoryjnym Kubernetes.  
HAProxy będzie pełnił rolę **load balancera**, przekierowując ruch do serwerów master klastra Kubernetes.

---

## 🔧 Wstęp

Na początku nasze środowisko jest całkowicie puste. Twoim zadaniem będzie przygotowanie serwera **HAProxy**, który umożliwi równoważenie ruchu między serwerami **master** w klastrze.

---

## 🔄 Aktualizacja systemu

Zaktualizuj system operacyjny serwera HAProxy, aby upewnić się, że korzysta on z najnowszych pakietów:

```bash
sudo apt-get update
sudo apt-get -y upgrade
```

---

## 🧰 Instalacja narzędzi pomocniczych

Na serwerze HAProxy zainstalujemy narzędzia potrzebne do generowania certyfikatów SSL oraz do komunikacji z Kubernetesem.

### 1. Instalacja CloudFlare SSL (CFSSL)

Pobierz i zainstaluj pakiety **cfssl** oraz **cfssljson**:

```bash
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
chmod +x cfssl*
sudo mv cfssl_linux-amd64 /usr/local/bin/cfssl
sudo mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
```

Sprawdź poprawność instalacji:
```bash
cfssl version
```

---

### 2. Instalacja klienta Kubernetes (kubectl)

Pobierz i zainstaluj klienta `kubectl`, który umożliwi Ci komunikację z klastrem Kubernetes:

```bash
wget https://storage.googleapis.com/kubernetes-release/release/v1.9.0/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin
kubectl version
```

---

## ⚙️ Instalacja i konfiguracja HAProxy

Zainstaluj serwer HAProxy:

```bash
sudo apt-get -y install haproxy
```

Otwórz plik konfiguracyjny:
```bash
sudo vi /etc/haproxy/haproxy.cfg
```

Na końcu pliku dodaj następującą konfigurację:

```bash
global

defaults

frontend kubernetes
    bind        [HAProxy-IP]:6443
    option      tcplog
    mode        tcp
    default_backend kubernetes-master-nodes

backend kubernetes-master-nodes
    mode    tcp
    balance roundrobin
    option  tcp-check
    server  master01 [master01-IP]:6443 check fall 3 rise 2
```

📘 **Objaśnienie konfiguracji:**
- `frontend kubernetes` — definiuje punkt wejściowy ruchu przychodzącego z zewnątrz.  
- `backend kubernetes-master-nodes` — lista serwerów master, do których przekierowywany jest ruch.  
- `balance roundrobin` — równoważenie ruchu metodą rotacyjną.  
- `tcp-check` — weryfikacja dostępności portu 6443.  

Jeśli w środowisku znajdowałyby się dodatkowe serwery master, należałoby dodać kolejne linie w sekcji `backend`, np.:
```bash
server  master02 [master02-IP]:6443 check fall 3 rise 2
server  master03 [master03-IP]:6443 check fall 3 rise 2
```

---

## ▶️ Uruchomienie HAProxy

Zrestartuj usługę, aby zastosować zmiany:

```bash
sudo systemctl restart haproxy
```

Sprawdź, czy HAProxy działa poprawnie:
```bash
sudo systemctl status haproxy
```

---

## ✅ Podsumowanie

Na tym etapie serwer **HAProxy** został poprawnie skonfigurowany i pełni funkcję równoważenia ruchu do serwera master Kubernetes.
