# Konfiguracja klienta

Na koniec skonfigurujemy **klienta kubectl**, aby wygodnie zarządzać klastrem z maszyny z HAProxy.  
HAProxy pełni tutaj rolę **jednego punktu dostępowego** do Control Plane (load balancer / reverse proxy przed API Serverami).

---

# Przygotowanie maszyny klienckiej (HAProxy)

Połącz się z maszyną z HAProxy (np. przez SSH) i przejdź do katalogu z certyfikatami oraz konfiguracją, np. `conf/`.

```bash
EXTERNAL_IP=$(hostname -i)
cd conf/
```

- `EXTERNAL_IP` – adres IP HAProxy, który wystawia port 6443 i przekazuje ruch do API Servera,
- katalog `conf/` zakładamy jako miejsce, gdzie znajdują się pliki `ca.pem`, `admin.pem`, `admin-key.pem` itd.

---

# Konfiguracja kubectl

Teraz skonfigurujemy kubectl tak, aby:

1. znał klaster (`set-cluster`),
2. posiadał dane użytkownika (`set-credentials`),
3. miał zdefiniowany kontekst (`set-context`),
4. używał tego kontekstu domyślnie (`use-context`).

---

## 1. Konfiguracja klastra

```bash
kubectl config set-cluster kubernetes \
  --certificate-authority=ca.pem \
  --embed-certs=true \
  --server=https://${EXTERNAL_IP}:6443
```

- `--certificate-authority=ca.pem` – zaufany CA, który podpisał certyfikaty API Servera,
- `--embed-certs=true` – certyfikat CA zostanie osadzony bezpośrednio w pliku kubeconfig,
- `--server` – adres endpointu API (za HAProxy).

---

## 2. Dane uwierzytelniające użytkownika (admin)

```bash
kubectl config set-credentials admin \
  --client-certificate=admin.pem \
  --client-key=admin-key.pem
```

Jest to użytkownik administracyjny, którego certyfikat został wygenerowany w jednym z poprzednich modułów.

---

## 3. Kontekst kubectl

Tworzymy kontekst `kubernetes`, który łączy klaster `kubernetes` z użytkownikiem `admin`:

```bash
kubectl config set-context kubernetes --cluster=kubernetes --user=admin
```

---

## 4. Ustawienie domyślnego kontekstu

```bash
kubectl config use-context kubernetes
```

Od tego momentu wszelkie polecenia `kubectl` będą domyślnie używały:

- klastra: `kubernetes`,
- użytkownika: `admin`,
- endpointu: `https://${EXTERNAL_IP}:6443`.

---

# Test połączenia z klastrem

Sprawdźmy, czy klient poprawnie łączy się z API Serverem oraz czy klaster widzi węzły worker:

```bash
kubectl get nodes
```

Jeśli wszystko działa poprawnie, zobaczysz listę węzłów (`master`, `worker01`, `worker02`) wraz z ich statusem, np. `Ready`.

---

# Uprawnienia dla użytkownika „kubernetes”

Jeżeli chcesz zezwolić użytkownikowi `kubernetes` (podpisanemu odpowiednim certyfikatem) na dostęp do kubelet API na węzłach, możesz dodać odpowiednie powiązanie ról:

```bash
kubectl create clusterrolebinding apiserver-kubelet-api-admin \
  --clusterrole system:kubelet-api-admin \
  --user kubernetes
```

To przyzna użytkownikowi `kubernetes` uprawnienia do wywoływania kubelet API na węzłach (np. do debugowania podów).

---

# Podsumowanie

Masz swój **pierwszy w pełni działający klaster Kubernetes**, który postawiłeś od początku do końca sam – od certyfikatów, przez etcd, Control Plane, workery, aż po klienta. Gratulacje!

Kolejnym krokiem będzie uruchomienie podstawowych usług w klastrze, zaczynając od **KubeDNS**.
