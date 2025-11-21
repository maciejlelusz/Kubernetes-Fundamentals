# Serwisy

**Service** w Kubernetes zapewnia stabilny punkt dostępu do aplikacji działających w podach.  
W tym module:

- sprawdzisz istniejące serwisy,
- wdrożysz Kubernetes Dashboard,
- utworzysz użytkownika z uprawnieniami administracyjnymi,
- wystawisz Dashboard przez NodePort,
- połączysz się do niego przez HAProxy.

---

# Podstawowe operacje na serwisach

Wyświetlenie listy serwisów w klastrze:

```bash
kubectl get services
```

Na tym etapie powinieneś widzieć m.in. `kubernetes` oraz `kube-dns`.

---

# Instalacja Kubernetes Dashboard

Wdrażamy Dashboard za pomocą oficjalnego manifestu:

```bash
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```

Po chwili serwis i pody powinny pojawić się w przestrzeni `kube-system`.

---

# Tworzenie użytkownika administracyjnego

Aby połączyć się z Dashboardem, potrzebny jest użytkownik z odpowiednimi uprawnieniami.

Utwórz plik `kubernetes-dashboard-admin.yaml`:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
```

Zastosuj manifest:

```bash
kubectl create -f kubernetes-dashboard-admin.yaml
```

---

# Pobranie tokenu logowania

Token logowania dla konta admin-user:

```bash
kubectl -n kube-system describe secret \
$(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```

Zachowaj token — będziesz go potrzebować podczas logowania do Dashboardu.

---

# Wystawienie Dashboardu przez NodePort

Domyślnie Dashboard działa jako `ClusterIP`, więc nie jest dostępny spoza klastra.  
Zmodyfikujemy go, aby był wystawiony jako `NodePort`.

Edytujemy usługę:

```bash
kubectl -n kube-system edit service kubernetes-dashboard
```

Odnajdź sekcję:

```yaml
spec:
  type: ClusterIP
```

i zmień na:

```yaml
spec:
  type: NodePort
```

Zapisz i zamknij edytor.

Następnie sprawdź, jaki NodePort został przydzielony:

```bash
kubectl -n kube-system get service kubernetes-dashboard
```

Przykład:

```
443:31234/TCP
```

To oznacza, że Dashboard będzie dostępny pod portem `31234` na każdym węźle.

---

# Dostęp przez HAProxy

Zaktualizuj konfigurację HAProxy, aby przepuścić ruch do nowo utworzonego NodePorta.

Po aktualizacji wejdziesz na Dashboard pod adresem:

```
https://zewnetrzny.adres.twojego.haproxy:8080
```

Pamiętaj — Dashboard działa *tylko* przez HTTPS.

Podczas logowania użyj tokenu pobranego wcześniej.

---

# Podsumowanie

Poznałeś:

- jak działają serwisy w Kubernetes,
- jak zainstalować Dashboard,
- jak utworzyć użytkownika z uprawnieniami administracyjnymi,
- jak wystawić Dashboard do świata,
- jak skorzystać z HAProxy, aby do niego dotrzeć.

Czas na to, co najważniejsze w Kubernetes — **deploymenty**.
