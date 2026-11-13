# Veille technique — TD 1

Pour chaque sujet : comparer deux options, leurs limites et justifier
la recommandation. Ajouter les liens des sources consultees.

## 1. Récupérer les publications de presse

### Objectif
Récupérer les titres, extraits, liens et dates des publications
d'au moins quatre PQR et quatre PQN, dans le respect des conditions
des éditeurs, sans collecter le texte intégral.

### Option A — Flux RSS/Atom des éditeurs

Principe :
Lire les flux de syndication directement publiés par les journaux.

Avantages :
- Publications accessibles sous une forme structurée.
- Accès direct aux sources sélectionnées.
- Possibilité de ne conserver que les champs autorisés par le projet.

Limites :
- Disponibilité des flux à vérifier pour chaque journal.
- Dates ou extraits parfois absents.
- Fréquence de mise à jour et profondeur du flux à vérifier.
- Les conditions de réutilisation restent à examiner.
- Un flux peut contenir du texte intégral : ce contenu ne doit pas
  être conservé, y compris dans le stockage brut ou les journaux.

Décision :
Retenir les flux RSS des éditeurs comme mode de collecte.
Les huit sources ont été inspectées manuellement et les résultats
sont consignés dans sources.csv. Les droits de réutilisation restent
à clarifier avant exploitation.

### Option B — API officielle d'un éditeur

Principe :
Utiliser une interface documentée proposée directement par un journal.

Avantages :
- Accès structuré lorsqu'une API est disponible.
- Documentation permettant d'identifier les champs et modalités d'accès.

Limites :
- Existence et accessibilité à vérifier pour chaque journal.
- Authentification, quotas ou coûts possibles.
- Intégrations potentiellement différentes selon les éditeurs.

Décision :
Non retenue comme solution principale à ce stade, faute de confirmation
d'un accès adapté aux huit sources et au budget nul.

### Option C — API d'un agrégateur de presse

Principe :
Interroger un service centralisant les publications de plusieurs médias.

Avantages :
- Point d'accès commun à plusieurs sources.
- Recherche et filtrage selon les possibilités du fournisseur.

Limites :
- Couverture des quatre PQR et quatre PQN à confirmer.
- Présence des extraits et dates de publication à vérifier.
- Quotas, fraîcheur et conditions d'utilisation variables.
- Dépendance à un intermédiaire supplémentaire.
- Gratuité en développement ne signifie pas gratuité en production.

Décision :
Alternative écartée à ce stade : son adéquation à la couverture,
à la fraîcheur et au budget du projet n'est pas établie.

### Option D — Extraction des pages HTML

Principe :
Extraire les métadonnées directement depuis les pages web des journaux.

Limites :
- Approche hors du périmètre retenu pour ce TP, qui exclut le scraping.
- Aucun texte intégral ne doit être collecté.

Décision :
Option écartée pour respecter les consignes du projet.

### Choix retenu et justification

Les flux RSS des éditeurs sont retenus pour récupérer les métadonnées
nécessaires sans extraire les pages des articles.

Les huit flux sélectionnés ont été inspectés manuellement :
titres, extraits, liens, dates et publications du jour ont été observés.
Les adresses et les résultats sont documentés dans sources.csv.

Cette vérification technique ne vaut pas autorisation de réutilisation.
Les restrictions connues et les conditions restant à clarifier sont
consignées dans sources.csv. Une source incompatible avec l'usage prévu
devra être remplacée ou faire l'objet d'une autorisation adaptée.

### Sources de la veille

- Spécification RSS :
  https://cyber.harvard.edu/rss/rss.html
- Spécification Atom :
  https://datatracker.ietf.org/doc/html/rfc4287
- Exemple d'API d'agrégation — GDELT DOC :
  https://blog.gdeltproject.org/gdelt-doc-2-0-api-debuts/
- Exemple de conditions commerciales d'une API — NewsAPI :
  https://newsapi.org/pricing

La disponibilité d'API officielles propres aux journaux n'a pas été
confirmée. Les vérifications techniques des huit flux RSS sont terminées ;
les vérifications des conditions de réutilisation restent à finaliser.

## 2. Exécuter le pipeline chaque matin

Responsable : Darrel.
Relecture : Yannick.
Statut : choix du timer systemd validé par Darrel pour la cible
de production ; compatibilité de l'infrastructure fournie à confirmer.

### Besoin

Déclencher automatiquement les traitements assez tôt pour rendre
l'édition disponible à 7 h, suivre leur résultat et permettre une relance
contrôlée en cas d'échec.

### Option A — Cron sur Linux

Avantages :
- Déclenchement léger d'une commande à heure fixe.
- Configuration simple à versionner.

Limites :
- Cron standard ne rattrape pas une exécution manquée pendant l'arrêt
  de la machine.
- Les reprises, journaux, alertes et dépendances entre étapes
  doivent être organisés séparément.

Décision proposée :
Alternative simple, non privilégiée face aux possibilités de suivi
et de rattrapage offertes par systemd.

### Option B — Timer systemd sur Linux

Avantages :
- Planification dans une unité timer associée à une unité service.
- Rattrapage possible d'un déclenchement manqué avec Persistent=true.
- Configuration déclarative pouvant être versionnée.

Limites :
- Nécessite une machine Linux utilisant systemd.
- Ne fournit pas à lui seul un suivi détaillé des étapes du pipeline.
- Un rattrapage après redémarrage ne rétablit pas le respect de 7 h
  si l'édition aurait dû être publiée pendant l'arrêt.

Décision :
Timer systemd retenu pour la cible de production, sous réserve
que le serveur fourni utilise systemd et permette le déploiement
des unités nécessaires.

### Option C — Apache Airflow

Avantages :
- Dépendances entre tâches décrites sous forme de graphe.
- Interface de suivi, consultation des journaux et déclenchement manuel.
- Adapté à une orchestration comportant plusieurs étapes distinctes.

Limites :
- Plusieurs composants à déployer et maintenir.
- Base de métadonnées nécessaire.
- Complexité supplémentaire à justifier pour ce projet.

Décision proposée :
Non retenu au démarrage ; à reconsidérer si les besoins d'orchestration
dépassent une exécution séquentielle maîtrisée.

### Option D — GitHub Actions planifié

Avantages :
- Configuration versionnée avec le projet.
- Exécution possible sur un runner hébergé.

Limites :
- Déclenchements pouvant être retardés ou abandonnés sous forte charge.
- Quotas et conditions de l'offre à vérifier.
- Persistance des données et accès à la production à organiser.

Décision proposée :
Non retenu comme seul ordonnanceur de production pour l'édition de 7 h.
Son éventuelle utilisation pour la CI sera évaluée séparément.

### Choix retenu et conditions de mise en œuvre

Retenir un timer systemd pour la production.
Confirmer la compatibilité du serveur lorsque l'infrastructure
sera communiquée.

Prévoir :
- une heure de déclenchement antérieure à 7 h, ajustée après mesure ;
- un fuseau de publication explicitement défini ;
- un identifiant et un statut pour chaque exécution ;
- des tentatives de reprise bornées et des délais maximaux ;
- une protection contre les exécutions concurrentes ;
- une relance sans doublons ;
- une alerte si l'édition n'est pas disponible à l'heure attendue.

### Sources

- systemd.timer :
  https://www.freedesktop.org/software/systemd/man/latest/systemd.timer.html
- Architecture Apache Airflow :
  https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/overview.html
- Événements GitHub Actions, section schedule :
  https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#schedule

### Décision de l'équipe

Darrel valide le choix d'un timer systemd pour la cible de production,
sous réserve que le serveur fourni utilise Linux avec systemd et permette
le déploiement des unités nécessaires.

Les PC Windows de l'équipe utiliseront WSL 2 pour développer et tester
la configuration. La production devra fonctionner sur une infrastructure
disponible en continu.

La planification devra permettre de terminer les traitements avant
la publication de l'édition à 7 h. La marge de traitement, le fuseau
de référence et la gestion des échecs restent à préciser.

## 3. Stocker les données brutes et propres

### Besoin

Conserver les métadonnées autorisées avant nettoyage pour permettre
la traçabilité et le retraitement, puis stocker les données propres
pour le regroupement par thèmes et la génération des éditions.

Le stockage brut ne doit pas contenir le flux XML intégral, les images
ou le texte intégral des articles. Seuls les champs autorisés et
les métadonnées techniques nécessaires sont conservés.

### Option A — Fichiers JSONL

Principe :
Conserver un objet JSON par ligne, dans des fichiers organisés
par source et par collecte.

Avantages :
- Organisation simple pour les lots collectés.
- Conservation des valeurs autorisées avant nettoyage.
- Possibilité de relire les lots pour rejouer les traitements.

Limites :
- Déduplication et contrôles de cohérence à développer.
- Recherche et jointures moins pratiques qu'avec une base SQL.
- Gestion des fichiers, sauvegardes et rétention à prévoir.

Décision :
Retenir pour les métadonnées brutes autorisées.

### Option B — SQLite

Principe :
Stocker les données nettoyées dans une base relationnelle locale.

Avantages :
- Pas de serveur de base de données à administrer.
- Requêtes SQL pour sélectionner les articles et préparer les éditions.
- Contraintes d'unicité pour éviter les doublons.
- Adapté à un démarrage local avec peu d'écritures concurrentes.

Limites :
- Un seul écrivain à un instant donné par fichier de base.
- Accès direct depuis plusieurs machines à éviter.
- Sauvegarde cohérente et persistance du fichier à organiser.

Décision :
Retenir SQLite pour les données propres, les thèmes et les éditions,
avec écritures coordonnées sur une seule machine.

### Option C — PostgreSQL

Principe :
Utiliser une base relationnelle serveur, avec possibilité de stocker
des données semi-structurées dans des colonnes JSONB.

Avantages :
- Architecture client-serveur.
- Stockage relationnel et possibilités de requêtes sur JSONB.
- Indexation possible des données JSONB.

Limites :
- Service supplémentaire à configurer et surveiller.
- Gestion des accès, sauvegardes et mises à jour à prévoir.
- Complexité initiale supérieure à notre proposition locale.

Décision :
Ne pas retenir au démarrage.
Réexaminer si plusieurs services doivent écrire simultanément
ou accéder à une base commune depuis plusieurs machines.

### Option D — Fichiers Parquet

Principe :
Stocker les données sous forme de fichiers en colonnes,
avec compression.

Avantages :
- Adapté aux analyses de lots de données.
- Format compressé pris en charge par de nombreux outils analytiques.

Limites :
- Ne fournit pas à lui seul les contraintes d'une base relationnelle.
- Gestion des mises à jour, doublons et fichiers à organiser.
- Pas nécessaire comme stockage principal pour notre première version.

Décision :
Ne pas retenir comme stockage principal initial.
À réexaminer pour des exports analytiques si un besoin apparaît.

### Décision retenue

Retenir JSONL pour les métadonnées brutes autorisées et SQLite
pour les données nettoyées, les thèmes et les éditions.

Conditions de validité :
- Exécution sur une seule machine au démarrage.
- Écritures coordonnées pour limiter la concurrence.
- Stockage persistant indépendant du cycle de vie des conteneurs.
- Sauvegardes et politique de rétention à définir.
- Données collectées exclues du dépôt Git.
- Droits de réutilisation des sources à clarifier avant exploitation.

Le choix JSONL + SQLite est validé pour la première architecture
et reporté dans docs/cadrage.md.

En développement, le code et les données d'exécution seront placés
dans le système de fichiers Linux de WSL 2. Chaque membre disposera
de ses données de test locales.

Les mesures de persistance, de sauvegarde et de restauration restent
à mettre en œuvre et à tester lors des TD suivants.

### Sources

- SQLite — Appropriate Uses :
  https://www.sqlite.org/whentouse.html
- PostgreSQL — JSON Types :
  https://www.postgresql.org/docs/current/datatype-json.html
- Apache Parquet — Présentation :
  https://parquet.apache.org/

## 4. Regrouper les textes par themes
A completer.

## 5. Surveiller le pipeline
A completer.

## 6. Decrire l'infrastructure en code
A completer.
