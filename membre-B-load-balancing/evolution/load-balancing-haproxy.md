# Étape réalisée par le Membre B : Load Balancing avec HAProxy

> **Statut :** environnement fonctionnel, tests de base validés
> **Périmètre :** répartition de charge HTTP entre trois serveurs Nginx

---

## 1. Objectif

Mettre en place un système de répartition de charge (*load balancing*) qui distribue les requêtes HTTP entre trois serveurs web Nginx, au lieu de les envoyer à un serveur unique. Cette approche améliore la disponibilité et la répartition de la charge.

---

## 2. Architecture mise en œuvre

```text
Client HTTP (curl / navigateur)
          │
          ▼
  HAProxy – port 8080 (Round Robin)
          │
   ┌──────┼──────┐
   ▼      ▼      ▼
 WEB1    WEB2    WEB3
Nginx:80 Nginx:80 Nginx:80
```

Les quatre conteneurs (HAProxy, WEB1, WEB2, WEB3) communiquent via le réseau Docker `lb-network` (driver `bridge`).

| Composant | Image | Rôle |
|---|---|---|
| `haproxy` | `haproxy:alpine` | Répartiteur de charge, exposé sur le port 8080 |
| `web1`, `web2`, `web3` | `nginx:alpine` | Serveurs web de backend |

---

## 3. Structure du projet

```text
membre-B-load-balancing/
├── captures/                  # Preuves (captures d'écran)
├── docker/
│   ├── docker-compose.yml
│   └── nginx/
│       ├── web1/              # index.html + health/index.html
│       ├── web2/
│       └── web3/
├── docs/
│   ├── 01-mission-et-perimetre.md
│   ├── 02-conception-architecturale.md
│   └── 04-plan-experimental.md
├── haproxy/
│   ├── haproxy.cfg
│   └── strategies/            # Futures stratégies de répartition
├── results/
├── scripts/
├── tests/
└── README.md
```

---

## 4. Réalisations techniques

### 4.1 Serveurs Nginx

Chaque serveur affiche une page distincte (« Bienvenue sur WEB1 / WEB2 / WEB3 »), ce qui permet d'identifier immédiatement le serveur ayant traité une requête.

### 4.2 Endpoints de santé

Chaque serveur expose `/health/`, qui renvoie `OK` avec un statut HTTP 200. HAProxy s'en sert pour surveiller l'état des serveurs.

### 4.3 Orchestration Docker Compose

Le fichier `docker/docker-compose.yml` déclare :

- les quatre services ;
- le réseau `lb-network` ;
- les volumes montés en lecture seule (`:ro`) ;
- la dépendance de HAProxy vis-à-vis des trois serveurs web (`depends_on`).

Validation de la configuration :

```bash
docker compose -f docker/docker-compose.yml config
```

Résultat : configuration valide.

### 4.4 Configuration HAProxy

Fichier : `haproxy/haproxy.cfg`

```cfg
global
    log stdout format raw local0
    maxconn 2000

defaults
    log global
    mode http
    option httplog
    option dontlognull

    timeout connect 5s
    timeout client 30s
    timeout server 30s

frontend http_front
    bind *:8080
    default_backend web_servers

backend web_servers
    balance roundrobin

    option httpchk GET /health/
    http-check expect status 200

    default-server inter 2s fall 3 rise 2

    server web1 web1:80 check
    server web2 web2:80 check
    server web3 web3:80 check
```

| Élément | Valeur | Rôle |
|---|---|---|
| Frontend | `bind *:8080` | Point d'entrée des clients |
| Stratégie | `balance roundrobin` | Distribution cyclique |
| Health check | `option httpchk GET /health/` + `http-check expect status 200` | Vérification HTTP de chaque serveur |
| `inter 2s` | Vérification toutes les 2 s | Détection rapide |
| `fall 3` | 3 échecs consécutifs | Serveur déclaré indisponible |
| `rise 2` | 2 succès consécutifs | Serveur réintégré |
| Timeouts | connect 5 s, client 30 s, server 30 s | Limites de connexion |

---

## 5. Incident rencontré et correction

| Rubrique | Détail |
|---|---|
| **Symptôme** | Erreur `503 Service Unavailable` renvoyée par HAProxy |
| **Cause** | Le health check ciblait `/health` ; Nginx redirige `/health` vers `/health/` (HTTP 301). HAProxy attendait un statut 200 et considérait donc les serveurs comme indisponibles. |
| **Correction** | `option httpchk GET /health/` (avec le slash final) |
| **Résultat** | Les trois serveurs sont détectés comme opérationnels, le service répond normalement |

---

## 6. Commandes principales

| Commande | Utilité |
|---|---|
| `docker compose -f docker/docker-compose.yml up -d` | Démarrer l'environnement en arrière-plan |
| `docker ps` | Afficher les conteneurs actifs |
| `docker exec <conteneur> <commande>` | Exécuter une commande dans un conteneur |
| `docker restart haproxy` | Redémarrer HAProxy |
| `docker logs haproxy` | Consulter les logs HAProxy |

---

## 7. Tests et résultats

| # | Test | Commande | Résultat |
|---|---|---|---|
| 1 | Conteneurs actifs | `docker ps` | 4 conteneurs en statut *Up* |
| 2 | Endpoints de santé | `docker exec <srv> wget -qO- http://localhost/health/` | `OK` sur WEB1, WEB2, WEB3 |
| 3 | Accès via HAProxy | `curl -i http://localhost:8080` | `HTTP/1.1 200 OK`, page servie par un backend |
| 4 | Répartition de charge | 9 requêtes `curl` successives | 3 requêtes par serveur |

### Test de répartition

```bash
for i in {1..9}; do
    echo "===== Requête $i ====="
    curl -s http://localhost:8080 | grep "Bienvenue"
done
```

Ordre de passage observé : WEB1, WEB3, WEB2, WEB3, WEB1, WEB2, WEB1, WEB3, WEB2.

| Serveur | Requêtes | Part |
|---|---|---|
| WEB1 | 3 | 33,3 % |
| WEB2 | 3 | 33,3 % |
| WEB3 | 3 | 33,3 % |

### Analyse

La répartition globale est équitable (3 / 3 / 3). L'ordre de passage n'est toutefois pas strictement cyclique, ce qui peut s'expliquer par le fonctionnement multi-thread de HAProxy. Cela n'affecte pas l'équité globale, mais il est préférable de le mentionner. Un test sur un volume plus important (100 ou 1000 requêtes) permettra de confirmer la tendance.

---

## 8. Livrables et état d'avancement

### Terminé

- [x] Environnement Docker fonctionnel (4 conteneurs)
- [x] Configuration HAProxy en Round Robin avec health checks
- [x] Correction de l'erreur 503
- [x] Tests de base validés

### Captures à produire (dossier `captures/`)

| Fichier | Contenu |
|---|---|
| `01-docker-containers.png` | Les quatre conteneurs actifs |
| `02-health-check.png` | Les trois réponses `OK` |
| `03-first-request.png` | Réponse HTTP 200 via HAProxy |
| `04-round-robin.png` | Les neuf requêtes distribuées |
| *(optionnel)* capture du 503 | Section « Diagnostic » (ne remplace pas la preuve du fonctionnement final) |

### Prochaines étapes

- [ ] Compléter `scripts/`, `tests/` et `results/`
- [ ] Implémenter d'autres stratégies dans `haproxy/strategies/` (par ex. `leastconn`, `source`)
- [ ] Tester la tolérance aux pannes (`docker stop web2`) pour vérifier la bascule automatique
- [ ] Lancer un test de charge sur un volume plus important (100 à 1000 requêtes)
