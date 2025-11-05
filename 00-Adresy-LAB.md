# Kubernetes Fundamentals — Adresy środowisk laboratoryjnych

W tym dokumencie znajdują się szczegółowe informacje o adresach hostów dla środowisk laboratoryjnych wykorzystywanych w ramach kursu **Kubernetes Fundamentals**.  
Każde laboratorium składa się z trzech podstawowych typów węzłów:
- **HAProxy** — odpowiedzialny za równoważenie obciążenia między serwerami master,  
- **Master** — zarządza klastrem Kubernetes,  
- **Worker** — realizuje zadania uruchamiania i zarządzania kontenerami (workload).

---

## 🔹 LAB01 — Środowisko testowe 1
| Komponent | Adres |
|------------|------------------------------------------------|
| HAProxy    | `haproxy-k8s01-noduwzp7.srv.ravcloud.com` |
| Master     | `master01-k8s01-tyudmqhi.srv.ravcloud.com` |
| Worker01   | `worker01-k8s01-vdu5eurl.srv.ravcloud.com` |
| Worker02   | `worker02-k8s01-dixhjhcy.srv.ravcloud.com` |

---

## 🔹 LAB02 — Środowisko testowe 2
| Komponent | Adres |
|------------|------------------------------------------------|
| HAProxy    | `haproxy-k8s02-p7wbb5pm.srv.ravcloud.com` |
| Master     | `master01-k8s02-p4nchzxv.srv.ravcloud.com` |
| Worker01   | `worker01-k8s02-mnbtelky.srv.ravcloud.com` |
| Worker02   | `worker02-k8s02-5c7wfjvh.srv.ravcloud.com` |

---

## 🔹 LAB03 — Środowisko testowe 3
| Komponent | Adres |
|------------|------------------------------------------------|
| HAProxy    | `haproxy-k8s03-qdgbslfk.srv.ravcloud.com` |
| Master     | `master01-k8s03-ubijdtkn.srv.ravcloud.com` |
| Worker01   | `worker01-k8s03-smfz2aid.srv.ravcloud.com` |
| Worker02   | `worker02-k8s03-jsmokpey.srv.ravcloud.com` |

---

## 🔹 LAB04 — Środowisko testowe 4
| Komponent | Adres |
|------------|------------------------------------------------|
| HAProxy    | `haproxy-k8s04-eiwwwl8q.srv.ravcloud.com` |
| Master     | `master01-k8s04-ewji9phd.srv.ravcloud.com` |
| Worker01   | `worker01-k8s04-mmk1tais.srv.ravcloud.com` |
| Worker02   | `worker02-k8s04-rxpwacpn.srv.ravcloud.com` |

---

## 🔹 LAB05 — Środowisko testowe 5
| Komponent | Adres |
|------------|------------------------------------------------|
| HAProxy    | `haproxy-k8s05-2rli44fs.srv.ravcloud.com` |
| Master     | `master01-k8s05-rt77amqp.oud.com` |
| Worker01   | `worker01-k8s05-m90plo6r.srv.ravcloud.com` |
| Worker02   | `worker02-k8s05-cy9u5vsq.srv.ravcloud.com` |

---

## ℹ️ Uwagi końcowe
- Wszystkie powyższe adresy dotyczą środowisk testowych dostarczonych przez platformę **RavCloud**.  
- Dostęp SSH i certyfikaty są konfigurowane indywidualnie dla każdego laboratorium.  
- Jeśli napotkasz problemy z połączeniem, upewnij się, że masz dostęp do odpowiedniej sieci VPN lub kluczy SSH.  
- W razie potrzeby zgłoś problem do prowadzącego lub administratora środowiska labowego.
