# Pody

**Pod** to najmniejsza jednostka uruchomieniowa w Kubernetes — zawiera jeden lub więcej kontenerów współdzielących sieć i przestrzeń procesów.  
W tym module uruchomimy prosty pod z Nginxem, sprawdzimy jego działanie i wejdziemy do środka kontenera.

---

# Tworzenie poda

Najprostszym sposobem na uruchomienie poda jest użycie polecenia:

```bash
kubectl run nginx --image=nginx
```

To polecenie tworzy jednocześnie **Deployment**, który zarządza pody.  
Sam pod może mieć dynamiczną nazwę, dlatego odczytamy ją później.

---

## Wyświetlenie listy podów

```bash
kubectl get pods -l run=nginx
```

Selektor `-l run=nginx` filtruje pody uruchomione przez powyższy deployment.

---

## Pobranie nazwy poda

Zapisujemy nazwę do zmiennej:

```bash
POD_NAME=$(kubectl get pods -l run=nginx -o jsonpath="{.items[0].metadata.name}")
```

Możesz to sprawdzić:

```bash
echo $POD_NAME
```

---

# Port-forwarding — testowanie lokalne

Forwardujemy port z lokalnej maszyny do poda:

```bash
kubectl port-forward $POD_NAME 8080:80
```

W innym terminalu testujemy:

```bash
curl --head http://127.0.0.1:8080
```

Powinieneś zobaczyć nagłówki odpowiedzi HTTP z serwera nginx.

---

# Logowanie i dostęp do kontenera

## Odczytywanie logów

```bash
kubectl logs $POD_NAME
```

## Wykonywanie poleceń w kontenerze

Sprawdzenie wersji nginx:

```bash
kubectl exec -ti $POD_NAME -- nginx -v
```

Wejście do środka kontenera:

```bash
kubectl exec -ti $POD_NAME -- bash
```

(W niektórych obrazach zamiast `bash` dostępny jest tylko `sh`.)

---

# Wystawienie aplikacji — Service

Aby wystawić aplikację poza pod, tworzymy serwis typu NodePort:

```bash
kubectl expose deployment nginx --port 80 --type NodePort
```

To polecenie tworzy obiekt:

- Service,
- wskazujący na Deployment nginx,
- przydzielający port z zakresu 30000–32767 na każdym węźle.

Po stworzeniu serwisu możesz zaktualizować HAProxy, aby przepuszczać ruch z zewnątrz do aplikacji.  
Tym jednak zajmiemy się później — najpierw czas poznać dokładniej *serwisy*.

---

# Podsumowanie

Teraz rozumiesz:

- czym są pody,
- jak je uruchamiać,
- jak wykonuje się port-forwarding,
- jak przeglądać logi i uruchamiać polecenia wewnątrz kontenera,
- jak wystawiać aplikacje za pomocą serwisu.

Kolejny krok to poznanie **Service**, czyli sposobu na ekspozycję aplikacji w Kubernetesie.
