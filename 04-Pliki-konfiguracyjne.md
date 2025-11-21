# Pliki konfiguracyjne

W tym module przygotujemy zestaw plików konfiguracyjnych (`kubeconfig`) wymaganych przez poszczególne komponenty klastra Kubernetes. Pliki te określają m.in. sposób uwierzytelnienia, adres API Servera oraz kontekst używany przez daną usługę.

---

## Kubelet – pliki konfiguracyjne

Każdy węzeł typu *worker* musi otrzymać własny plik konfiguracyjny dla usługi **kubelet**. Zaczynamy od `worker01`:

```bash
{
  kubectl config set-cluster kubernetes \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${EXTERNAL_IP}:6443 \
    --kubeconfig=worker01.kubeconfig

  kubectl config set-credentials system:node:worker01 \
    --client-certificate=worker01.pem \
    --client-key=worker01-key.pem \
    --embed-certs=true \
    --kubeconfig=worker01.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes \
    --user=system:node:worker01 \
    --kubeconfig=worker01.kubeconfig

  kubectl config use-context default --kubeconfig=worker01.kubeconfig
}
```

Konfiguracja dla `worker02`:

```bash
{
  kubectl config set-cluster kubernetes \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${EXTERNAL_IP}:6443 \
    --kubeconfig=worker02.kubeconfig

  kubectl config set-credentials system:node:worker02 \
    --client-certificate=worker02.pem \
    --client-key=worker02-key.pem \
    --embed-certs=true \
    --kubeconfig=worker02.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes \
    --user=system:node:worker02 \
    --kubeconfig=worker02.kubeconfig

  kubectl config use-context default --kubeconfig=worker02.kubeconfig
}
```

---

## Kube-proxy – pliki konfiguracyjne

```bash
{
  kubectl config set-cluster kubernetes \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${EXTERNAL_IP}:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.pem \
    --client-key=kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
}
```

---

## Kube-controller-manager – pliki konfiguracyjne

```bash
{
  kubectl config set-cluster kubernetes \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.pem \
    --client-key=kube-controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
}
```

---

## Kube-scheduler – pliki konfiguracyjne

```bash
{
  kubectl config set-cluster kubernetes \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.pem \
    --client-key=kube-scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
}
```

---

## Admin – pliki konfiguracyjne

```bash
{
  kubectl config set-cluster kubernetes \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default --kubeconfig=admin.kubeconfig
}
```

---

## Przesyłanie plików na węzły

```bash
scp -i ../Kubernetes_Fundamentals.pem worker01.kubeconfig kube-proxy.kubeconfig ubuntu@${WORKER01_IP}:~
scp -i ../Kubernetes_Fundamentals.pem worker02.kubeconfig kube-proxy.kubeconfig ubuntu@${WORKER02_IP}:~
scp -i ../Kubernetes_Fundamentals.pem admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig ubuntu@${MASTER01_IP}:~
```

---

## Podsumowanie

Wszystkie wymagane pliki konfiguracyjne zostały przygotowane i przesłane.  
Możemy przejść do kolejnego modułu – **Szyfrowanie**.
