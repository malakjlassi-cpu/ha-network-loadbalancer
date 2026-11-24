# HAProxy Load Balancing


## Présentation

Ce module porte sur la mise en œuvre d'un mécanisme d'équilibrage de charge basé sur **HAProxy**, permettant de répartir le trafic entrant entre plusieurs serveurs Web.

Il vise à améliorer l'utilisation des ressources disponibles, à limiter la surcharge d'un serveur unique et à maintenir la continuité du service en cas d'indisponibilité d'un serveur backend.

## Objectifs

- Mettre en place une architecture composée d'un répartiteur de charge et de plusieurs serveurs backend.
- Implémenter et comparer différentes stratégies d'équilibrage.
- Assurer la surveillance de la disponibilité des serveurs backend.
- Gérer automatiquement l'exclusion et la réintégration des serveurs défaillants.
- Évaluer les performances et le comportement du système dans différents scénarios.

## Périmètre fonctionnel

### 1. Stratégies d'équilibrage

Étude et mise en œuvre des stratégies suivantes :

| Algorithme | Principe |
|---|---|
| Round Robin | Répartition cyclique des connexions entre les serveurs. |
| Least Connections | Sélection du serveur ayant le moins de connexions actives. |
| Source | Sélection du serveur en fonction de l'adresse IP source du client. |
| Weighted Round Robin | Répartition proportionnelle aux poids attribués aux serveurs. |

### 2. Surveillance des serveurs backend

- Configuration des vérifications de disponibilité (*Health Checks*).
- Détection des serveurs indisponibles.
- Exclusion automatique des serveurs défaillants de la distribution du trafic.
- Réintégration automatique après rétablissement.

### 3. Tests de résilience

Validation du comportement de HAProxy dans les scénarios suivants :

- Fonctionnement normal de l'ensemble des serveurs.
- Indisponibilité d'un serveur backend.
- Rétablissement d'un serveur backend.
- Continuité du service pendant les défaillances.

### 4. Évaluation des performances

Comparaison expérimentale des stratégies selon les indicateurs suivants :

- Débit (*Throughput*).
- Temps de réponse (*Latency*).
- Taux d'erreur (*Error Rate*).
- Répartition du trafic entre les serveurs.

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
          |Nginx |  |Nginx |  |Nginx |
          +------+  +------+  +------+
```

Les composants seront déployés dans un environnement conteneurisé afin de faciliter la configuration, la reproductibilité des tests et la simulation des scénarios de défaillance.

## Technologies

| Technologie | Utilisation |
|---|---|
| HAProxy | Répartition du trafic et surveillance des backends |
| Docker | Conteneurisation des services |
| Docker Compose | Orchestration de l'environnement expérimental |
| Nginx | Serveurs Web backend |
| curl | Vérification des réponses HTTP |
| Wireshark | Capture et analyse du trafic réseau |
| Bash | Automatisation des tests |

## Organisation du module

| Répertoire | Description |
|---|---|
| `docs/` | Conception, périmètre et protocole expérimental |
| `docker/` | Définition de l'environnement conteneurisé |
| `haproxy/` | Configurations du répartiteur de charge |
| `scripts/` | Scripts de lancement et de test |
| `results/` | Données expérimentales, tableaux et graphiques |
| `captures/` | Captures et analyses Wireshark |

## Livrables

- Configurations HAProxy et Docker.
- Environnement expérimental reproductible.
- Scénarios et scripts de validation.
- Résultats des tests de disponibilité et de récupération.
- Analyse comparative des stratégies d'équilibrage.
- Mesures de performance et visualisations.
- Documentation technique et contribution au rapport final.

