 Load Balancing avec HAProxy

## Présentation

Ce module porte sur la mise en œuvre d'un mécanisme d'équilibrage de charge basé sur **HAProxy**, permettant de répartir le trafic entrant entre plusieurs serveurs Web.

Il vise à améliorer l'utilisation des ressources disponibles, à limiter la surcharge d'un serveur unique et à maintenir la disponibilité du service en cas d'indisponibilité d'un serveur backend.

## Objectifs

- Mettre en place une architecture composée d'un répartiteur de charge et de plusieurs serveurs backend.
- Implémenter et comparer quatre stratégies d'équilibrage.
- Permettre le changement de stratégie à travers un script dédié.
- Assurer la surveillance de la disponibilité des serveurs backend.
- Gérer automatiquement l'exclusion et la réintégration des serveurs défaillants.
- Tester le comportement du répartiteur face aux erreurs et aux serveurs lents.
- Évaluer les performances du système dans différents scénarios.

## Périmètre fonctionnel

### 1. Stratégies d'équilibrage

Étude et mise en œuvre des stratégies suivantes :

| **Algorithme** | **Principe** |
|---|---|
| Round Robin | Répartition cyclique des connexions entre les serveurs. |
| Least Connections | Sélection du serveur ayant le moins de connexions actives. |
| Source | Sélection du serveur en fonction de l'adresse IP source du client. |
| Weighted Round Robin | Répartition proportionnelle aux poids attribués aux serveurs. |

La stratégie utilisée sera paramétrable à l'aide du script `set_strategy.sh`.

Exemple :

```bash
./set_strategy.sh leastconn
```

### 2. Surveillance des serveurs backend

- Configuration des vérifications de disponibilité (*Health Checks*) à l'aide de `GET /health`.
- Définition des paramètres de vérification : `inter 2s`, `fall 3`, `rise 2`.
- Détection des serveurs indisponibles.
- Exclusion automatique des serveurs défaillants de la distribution du trafic.
- Réintégration automatique après rétablissement.

### 3. Gestion des erreurs et tests de résilience

Validation du comportement de HAProxy dans les scénarios suivants :

- Fonctionnement normal de l'ensemble des serveurs.
- Indisponibilité d'un serveur backend.
- Retrait d'un serveur après trois échecs de vérification.
- Réintégration après deux vérifications réussies.
- Indisponibilité de tous les serveurs backend et observation de l'erreur HTTP 503.
- Comportement face à un serveur lent.
- Vérification des mécanismes `retries` et `redispatch`.

### 4. Évaluation des performances

Comparaison expérimentale des quatre stratégies à l'aide d'un script de génération de charge développé dans ce module, avec `wrk` ou `curl`.

Les stratégies seront comparées selon les indicateurs suivants :

- Débit (*Throughput*).
- Temps de réponse (*Latency*).
- Taux d'erreur (*Error Rate*).
- Répartition du trafic entre les serveurs.

Les résultats seront présentés sous forme de tableaux et de graphiques.

### 5. Analyse du trafic réseau

Utilisation de Wireshark pour :

- Capturer les échanges HTTP.
- Observer les requêtes périodiques de health checks.
- Analyser le comportement du trafic lors d'une défaillance ou d'une récupération.
- Annoter et interpréter les captures obtenues.

---

## Architecture expérimentale

L'environnement de test repose sur une architecture centralisée autour de HAProxy, connecté à trois serveurs Web backend.

```text
                    Clients
                       |
                       v
                  +----------+
                  | HAProxy  |
                  +----------+
                       |
             +---------+---------+
             |         |         |
             v         v         v
          +------+  +------+  +------+
          | WEB1 |  | WEB2 |  | WEB3 |
          | Nginx|  | Nginx|  | Nginx|
          +------+  +------+  +------+
```

Les composants seront déployés dans un environnement conteneurisé afin de faciliter la configuration, la reproductibilité des tests et la simulation des scénarios de défaillance.

## Technologies

| **Technologie** | **Utilisation** |
|---|---|
| HAProxy | Répartition du trafic et surveillance des backends. |
| Docker | Conteneurisation des services et simulation de capacités différentes. |
| Docker Compose | Orchestration de l'environnement expérimental. |
| Nginx | Serveurs Web backend. |
| Bash | Automatisation du changement de stratégie et des tests. |
| curl | Vérification des réponses HTTP et de la distribution. |
| wrk | Génération de charge et mesure des performances. |
| Wireshark | Capture et analyse du trafic réseau. |

## Organisation du module

| **Répertoire** | **Description** |
|---|---|
| `docs/` | Documentation technique et protocole expérimental. |
| `docker/` | Définition de l'environnement conteneurisé. |
| `lb/` | Configuration principale HAProxy, variantes et script de changement de stratégie. |
| `scripts/` | Scripts de tests fonctionnels et de génération de charge. |
| `results/` | Données expérimentales, tableaux et graphiques. |
| `captures/` | Captures et analyses Wireshark. |

## Livrables

- Fichier `haproxy.cfg` comprenant la configuration du frontend, du backend, des health checks et des mécanismes de gestion des erreurs.
- Quatre variantes de configuration : Round Robin, Least Connections, Source et Weighted Round Robin.
- Script `set_strategy.sh` pour le changement de stratégie.
- Scripts de tests fonctionnels et de génération de charge.
- Tableau et graphique de comparaison des quatre stratégies.
- Captures Wireshark annotées et interprétées.
- Documentation technique et contribution au rapport final, notamment les sections :
  - Configuration du répartiteur.
  - Comparaison des stratégies.
  - Gestion des erreurs.

## Limites du module

Ce module est consacré à la configuration et à l'évaluation du répartiteur HAProxy.

La gestion de la haute disponibilité entre les load balancers avec Keepalived/VRRP et la gestion de la VIP sont traitées dans le module correspondant du projet global.

