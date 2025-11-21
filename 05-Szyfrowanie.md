# Szyfrowanie

Kubernetes przechowuje w etcd wiele kluczowych informacji – definicje obiektów, konfiguracje aplikacji, a przede wszystkim **sekrety**.  
Domyślnie dane te *nie są szyfrowane*, dlatego w tym module zajmiemy się wdrożeniem szyfrowania w warstwie etcd, co znacząco poprawia bezpieczeństwo klastra.

Celem jest przygotowanie pliku konfiguracyjnego `EncryptionConfig`, który poinformuje API Server, w jaki sposób ma szyfrować dane przechowywane w etcd.

---

## Generowanie klucza szyfrowania

Pierwszym krokiem jest wygenerowanie 32-bajtowego klucza, który będzie wykorzystywany przez algorytm **AES-CBC** do szyfrowania danych (głównie secretów).

```bash
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

Polecenie:

- pobiera 32 bajty losowych danych z `/dev/urandom`,
- koduje je w Base64 (wymagany format),
- zapisuje w zmiennej `ENCRYPTION_KEY`.

---

## Tworzenie manifestu `EncryptionConfig`

Następnie budujemy plik `encryption-config.yaml`.  
Jest to plik, który wskaże kube-apiserverowi, **jakiego algorytmu użyć**, **dla jakich zasobów** oraz **z jakim kluczem**.

```bash
cat > encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```

### Omówienie kluczowych elementów:

| Pole            | Znaczenie                                                                                  |
|-----------------|--------------------------------------------------------------------------------------------|
| `resources`     | Określa, jakie obiekty mają być szyfrowane. Tutaj: tylko `secrets`.                        |
| `aescbc`        | Algorytm szyfrowania, zalecany przez Kubernetes dla środowisk produkcyjnych.              |
| `keys`          | Lista kluczy — dzięki temu można rotować klucze w przyszłości.                             |
| `identity: {}`  | Ostatni krok łańcucha — oznacza „brak szyfrowania”. Pozostaje jako fallback.              |

Dzięki temu API Server *szyfruje* secret podczas zapisu i *odszyfrowuje* go podczas odczytu, zachowując pełną kompatybilność.

---

## Przesłanie pliku na master

Po wygenerowaniu manifestu należy przesłać go na węzeł kontrolny `master01`, tak aby kube-apiserver mógł go odczytać:

```bash
scp -i ../Kubernetes_Fundamentals.pem encryption-config.yaml ubuntu@${MASTER01_IP}:~
```

Na serwerze `master01` plik ten zostanie później umieszczony w lokalizacji wskazywanej parametrem:

```
--encryption-provider-config=/path/to/encryption-config.yaml
```

(wykonamy to w kolejnym module dotyczącym uruchamiania API Servera i etcd).

---

## Co dalej? Weryfikacja działania szyfrowania

Po pełnym wdrożeniu (w kolejnym module) warto zweryfikować czy dane rzeczywiście są szyfrowane.  
Przykład:

1. Tworzymy testowego secreta:
    ```bash
    kubectl create secret generic test-secret --from-literal=msg="tajne"
    ```

2. Podglądamy jego zawartość w etcd (nie przez kubectl, ale bezpośrednio w bazie):
    ```bash
    sudo ETCDCTL_API=3 etcdctl \
      --cacert=/etc/kubernetes/pki/ca.pem \
      --cert=/etc/kubernetes/pki/kubernetes.pem \
      --key=/etc/kubernetes/pki/kubernetes-key.pem \
      get /registry/secrets/default/test-secret | hexdump -C
    ```

Zaszyfrowany secret **nie powinien** zawierać oryginalnego tekstu `"tajne"`, a zamiast tego dane binarne lub Base64.

---

## Podsumowanie

- Wygenerowaliśmy klucz szyfrowania dla sekretnych danych Kubernetes.
- Przygotowaliśmy manifest `EncryptionConfig`, który konfiguruje sposób szyfrowania.
- Plik został wysłany na serwer master i będzie używany przez kube-apiserver.
- W kolejnym module uruchomimy bazę danych klastra — **etcd** — i podłączymy do niej mechanizm szyfrowania.
