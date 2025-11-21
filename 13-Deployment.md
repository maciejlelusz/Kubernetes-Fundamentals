# Deployment

**Deployment** to jeden z najważniejszych obiektów w Kubernetes.  
Odpowiada za:

- deklaratywne zarządzanie aplikacją,
- rollouty (wdrożenia) i rollbacki (wycofania),
- skalowanie manualne i automatyczne,
- utrzymanie pożądanego stanu replik.

W tym module utworzymy Deployment dla Nginxa, zaktualizujemy go, przeskalujemy i wykonamy rollback.

---

# Tworzenie Deploymentu

Przygotuj plik o nazwie `nginx-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

Plik definiuje:

- **3 repliki** Nginxa,
- selector oparty o label `app: nginx`,
- kontener z obrazem `nginx:1.7.9`,
- port kontenera `80`.

---

## Zastosowanie deploymentu

```bash
kubectl create -f https://k8s.io/examples/controllers/nginx-deployment.yaml
```

Możesz też użyć lokalnego pliku, jeśli go stworzyłeś:

```bash
kubectl apply -f nginx-deployment.yaml
```

---

# Podstawowe operacje

## Lista deploymentów

```bash
kubectl get deployments
```

---

## Status rollout’u (wdrożenia)

```bash
kubectl rollout status deployment/nginx-deployment
```

---

## Podgląd labeli

Zobaczmy, jakie etykiety mają pody:

```bash
kubectl get pods --show-labels
```

Deploymenty ściśle opierają się na labelach, dlatego to polecenie jest bardzo przydatne.

---

# Aktualizacja Deploymentu

Zmiana obrazu:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1
```

To spowoduje **rolling update**, czyli stopniowe zastępowanie starych podów nowymi.

---

## Podgląd szczegółów deploymentu

```bash
kubectl describe deployments
```

Znajdziesz tam informacje o rolloutach, replikach i zdarzeniach.

---

# Rollback — cofnięcie zmian

Najpierw sprawdzamy historię wersji:

```bash
kubectl rollout history deployment/nginx-deployment
```

Podgląd konkretnej rewizji:

```bash
kubectl rollout history deployment/nginx-deployment --revision=2
```

Wycofanie deploymentu do ostatniej stabilnej wersji:

```bash
kubectl rollout undo deployment/nginx-deployment
```

Możliwe jest również wycofanie do konkretnej rewizji:

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```

---

# Skalowanie

## Skalowanie ręczne

Zwiększenie liczby replik do 10:

```bash
kubectl scale deployment nginx-deployment --replicas=10
```

---

## Autoskalowanie (HPA)

Automatyczne skalowanie na podstawie użycia CPU:

```bash
kubectl autoscale deployment nginx-deployment --min=10 --max=15 --cpu-percent=80
```

To polecenie tworzy HorizontalPodAutoscaler (HPA), który monitoruje zużycie CPU i skaluję liczbę replik w zakresie 10–15.

---

# Podsumowanie

Wiesz już, jak:

- tworzyć i stosować Deploymenty,
- analizować rollouty,
- aktualizować aplikacje,
- cofać zmiany,
- skalować aplikacje ręcznie i automatycznie.

Deployment to fundament pracy z Kubernetesem — od teraz możesz wdrażać prawdziwe aplikacje produkcyjne.
