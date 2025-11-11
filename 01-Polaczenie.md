# Połączenie z laboratorium Kubernetes Fundamentals

Ten moduł pomoże Ci nawiązać połączenie z Twoim środowiskiem laboratoryjnym w ramach kursu **Kubernetes Fundamentals**.
Dowiesz się, jak uzyskać dostęp do serwerów, skonfigurować klucze SSH oraz jak przygotować środowisko do dalszych ćwiczeń.

---

## 🔍 Znalezienie adresów laboratorium

Aby odnaleźć dane dostępowe do swojego laboratorium (adresy hostów i ich role), odwiedź poniższą stronę:

👉 [Adresy laboratoriów](https://github.com/maciejlelusz/Kubernetes-Fundamentals/blob/master/00-Adresy-LAB.md)

Znajdziesz tam listę środowisk oznaczonych jako **LAB01–LAB05**, wraz z adresami serwerów:
- `haproxy` — serwer odpowiedzialny za równoważenie ruchu i przekierowania do masterów,
- `manager01` (master) — główny serwer zarządzający klastrem Kubernetes,
- `worker01` oraz `worker02` — węzły robocze obsługujące kontenery.

---

## 🔑 Pobranie i konfiguracja klucza SSH

Aby połączyć się z serwerami w laboratorium, musisz pobrać i skonfigurować klucz prywatny SSH.
Wykonaj poniższe kroki na swoim komputerze:

### 1. Pobierz klucz dostępu:
```bash
wget https://raw.githubusercontent.com/inleo-pl/Warsztat-Kubernetes-Fundamentals/master/Kubernetes_Fundamentals.pem
```

### 2. Ustaw odpowiednie uprawnienia:
```bash
chmod 600 /ścieżka/do/pliku/Kubernetes_Fundamentals.pem
```

---

## 🖥️ Połączenie z hostami

Po skonfigurowaniu klucza możesz połączyć się z poszczególnymi serwerami swojego laboratorium za pomocą poleceń:

```bash
ssh -i /ścieżka/do/pliku/Kubernetes_Fundamentals.pem ubuntu@haproxy
ssh -i /ścieżka/do/pliku/Kubernetes_Fundamentals.pem ubuntu@manager01
ssh -i /ścieżka/do/pliku/Kubernetes_Fundamentals.pem ubuntu@worker01
ssh -i /ścieżka/do/pliku/Kubernetes_Fundamentals.pem ubuntu@worker02
```

💡 **Wskazówka:**
Jeśli pojawi się ostrzeżenie o kluczu hosta (`The authenticity of host can't be established...`), wpisz `yes`, aby kontynuować połączenie.

---

## 🌐 Architektura środowiska

Na tym etapie warto:
- sprawdzić adresy IP wszystkich serwerów (`haproxy`, `manager01`, `worker01`, `worker02`),
- zapisać je, aby ułatwić późniejsze konfiguracje,
- pobrać również certyfikat `.pem` na serwer **HAProxy** w celu autoryzacji wewnętrznych połączeń.

---

## 📥 Pobranie klucza PEM na HAProxy

Zaloguj się na serwer **HAProxy** i wykonaj:

```bash
wget https://raw.githubusercontent.com/inleo-pl/Warsztat-Kubernetes-Fundamentals/master/Kubernetes_Fundamentals.pem
chmod 600 Kubernetes_Fundamentals.pem
```

To zapewni Ci możliwość dalszej komunikacji między węzłami przy użyciu bezpiecznych połączeń SSH.

---

## ✅ Podsumowanie

- Odczytałeś adresy serwerów swojego laboratorium.
- Skonfigurowałeś połączenie SSH przy użyciu klucza PEM.
- Znasz strukturę środowiska Kubernetes, które będziesz konfigurować.
