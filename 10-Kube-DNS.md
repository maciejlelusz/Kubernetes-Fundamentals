# Kube DNS

DNS to jedna z fundamentalnych usług w Kubernetes. Dzięki niemu pody mogą odnajdywać się po nazwach, a serwisy otrzymują stałe adresy DNS.  
W tym module uruchomimy **Kube DNS** (historyczną implementację; w nowszych wersjach zastąpioną przez CoreDNS).

---

# Uruchomienie serwisu DNS

Aby zainstalować serwer DNS w naszym klastrze, wykonujemy:

```bash
kubectl create -f https://raw.githubusercontent.com/inleo-pl/Warsztat-Kubernetes-Fundamentals/master/kube-dns.yaml
```

Manifest tworzy m.in.:

- Deployment Kube-DNS,
- Service kube-dns w przestrzeni nazw `kube-system`,
- ConfigMap i role wymagane przez DNS.

---

# Weryfikacja działania

Po chwili sprawdzamy, czy pody Kube-DNS wystartowały:

```bash
kubectl get pods -l k8s-app=kube-dns -n kube-system
```

Status powinien być `Running`.

---

# Testowanie DNS w klastrze

Aby sprawdzić, czy DNS działa poprawnie, uruchomimy testowy pod:

```bash
kubectl run busybox --image=busybox --command -- sleep 3600
kubectl get pods -l run=busybox
```

Busybox działa jako prosty kontener diagnostyczny – w tym przypadku po prostu „śpi”.

---

## Pobranie nazwy poda busybox

Pobieramy pełną nazwę uruchomionego poda:

```bash
kubectl get pods -l run=busybox -o jsonpath="{.items[0].metadata.name}"
```

Zwrócona wartość będzie wyglądać mniej więcej tak:

```
busybox-5c9d88b4f7-x2q8m
```

---

## Test DNS wewnątrz poda

Podstaw **uzyskaną nazwę** w poniższym poleceniu i uruchom:

```bash
kubectl exec -ti <NAZWA_POD> -- nslookup kubernetes
```

Jeśli wszystko działa:

- pod powinien rozwiązać nazwę `kubernetes` (domyślna usługa API Servera),
- DNS zwróci adres IP serwisu API, np. `10.32.0.1`.

---

# Podsumowanie

Gratulacje — w Twoim klastrze działa już wewnętrzny DNS, dzięki któremu aplikacje mogą komunikować się nazwami, a nie adresami IP.

Ale… co to właściwie są te pody, deploymenty, serwisy?  
To świetne pytanie — pora przejść do kolejnego modułu - Pody.
