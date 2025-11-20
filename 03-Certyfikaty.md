# Certyfikaty – Kubernetes Fundamentals

W tym labie stworzymy infrastrukturę PKI używając CloudFlare's PKI toolkit, cfssl, następnie machniemy Certificate Authority i wygenerujemy certyfikaty TLS dla następujących komponentów: etcd, kube-apiserver, kube-controller-manager, kube-scheduler, kubelet i kube-proxy.

## Stworzenie katalogu na certyfikaty
```
mkdir conf/
cd conf
```

## Zmienne środowiskowe
```
WORKER01_IP='[worker01-IP]'
WORKER02_IP='[worker02-IP]'
MASTER01_IP='[master01-IP]'
EXTERNAL_IP='[HAProxy-IP]'
```

## Certificate Authority
```
{
cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF

cat > ca-csr.json <<EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca
}
```

## Certyfikaty Client i Server – admin
```
{
cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:masters",
      "OU": "Kubernetes",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert   -ca=ca.pem   -ca-key=ca-key.pem   -config=ca-config.json   -profile=kubernetes   admin-csr.json | cfssljson -bare admin
}
```

## Certyfikaty kubelet – worker01
```
cat > worker01-csr.json <<EOF
{
  "CN": "system:node:worker01",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:nodes",
      "OU": "Kubernetes",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert   -ca=ca.pem   -ca-key=ca-key.pem   -config=ca-config.json   -hostname=worker01,${WORKER01_IP},${WORKER01_IP}   -profile=kubernetes   worker01-csr.json | cfssljson -bare worker01
```

## Certyfikaty kubelet – worker02
```
cat > worker02-csr.json <<EOF
{
  "CN": "system:node:worker02",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:nodes",
      "OU": "Kubernetes",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert   -ca=ca.pem   -ca-key=ca-key.pem   -config=ca-config.json   -hostname=worker02,${WORKER02_IP},${WORKER02_IP}   -profile=kubernetes   worker02-csr.json | cfssljson -bare worker02
```

## Certyfikat kube-controller-manager
```
{
cat > kube-controller-manager-csr.json <<EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-controller-manager",
      "OU": "Kubernetes",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert   -ca=ca.pem   -ca-key=ca-key.pem   -config=ca-config.json   -profile=kubernetes   kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager
}
```

## Certyfikat kube-proxy
```
{
cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:node-proxier",
      "OU": "Kubernetes",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert   -ca=ca.pem   -ca-key=ca-key.pem   -config=ca-config.json   -profile=kubernetes   kube-proxy-csr.json | cfssljson -bare kube-proxy
}
```

## Certyfikat kube-scheduler
```
{
cat > kube-scheduler-csr.json <<EOF
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-scheduler",
      "OU": "Kubernetes",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert   -ca=ca.pem   -ca-key=ca-key.pem   -config=ca-config.json   -profile=kubernetes   kube-scheduler-csr.json | cfssljson -bare kube-scheduler
}
```

## Certyfikat Kubernetes API Server
```
{
cat > kubernetes-csr.json <<EOF
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert   -ca=ca.pem   -ca-key=ca-key.pem   -config=ca-config.json   -hostname=10.32.0.1,${EXTERNAL_IP},${MASTER01_IP},127.0.0.1,haproxy,master01,kubernetes.default   -profile=kubernetes   kubernetes-csr.json | cfssljson -bare kubernetes
}
```

## Service Account Key Pair
```
{
cat > service-account-csr.json <<EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert   -ca=ca.pem   -ca-key=ca-key.pem   -config=ca-config.json   -profile=kubernetes   service-account-csr.json | cfssljson -bare service-account
}
```

## Przesyłanie certyfikatów na nody
```
scp -i ../Kubernetes_Fundamentals.pem ca.pem worker01-key.pem worker01.pem ubuntu@${WORKER01_IP}:~
scp -i ../Kubernetes_Fundamentals.pem ca.pem worker02-key.pem worker02.pem ubuntu@${WORKER02_IP}:~
scp -i ../Kubernetes_Fundamentals.pem ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem service-account-key.pem service-account.pem ubuntu@${MASTER01_IP}:~
```

