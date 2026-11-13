# Veille technique — TD 1

Comparer les options, leurs limites et justifier les décisions.
Les références consultées sont indiquées dans chaque section.

## 1. Récupérer les publications de presse

Responsable : Walter.
Relecture : Darrel.
Statut : collecte par flux RSS retenue ; vérifications techniques
terminées pour huit sources ; conditions de réutilisation à finaliser.

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
- Disponibilité et contenu des flux à contrôler.
- Dates ou extraits potentiellement absents.
- Fréquence de mise à jour et profondeur du flux variables.
- Conditions de réutilisation à examiner pour chaque source.
- Présence possible de contenu hors périmètre, à exclure du stockage
  et des journaux.

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
Non retenue comme solution principale : l'accès adapté aux huit sources
et au budget nul n'est pas confirmé.

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
Non retenue : son adéquation à la couverture, à la fraîcheur
et au budget du projet n'est pas établie.

### Option D — Extraction des pages HTML

Principe :
Extraire les métadonnées directement depuis les pages web des journaux.

Limites :
- Extraction des pages non retenue dans notre périmètre.
- Scraping du texte intégral explicitement exclu.
- Aucun texte intégral ne doit être collecté ou stocké.

Décision :
Option écartée ; utiliser les flux publiés par les éditeurs.

### Choix retenu et justification

Les flux RSS des éditeurs sont retenus pour récupérer les métadonnées
nécessaires sans extraire les pages des articles.

Les huit flux sélectionnés ont été inspectés manuellement :
titres, extraits, liens, dates et publications du jour ont été observés.
Les adresses et les résultats sont documentés dans sources.csv.

La fraîcheur de chaque item doit être contrôlée : un flux récent
peut également contenir des articles anciens.

Cette vérification technique ne vaut pas autorisation de réutilisation.
Les restrictions connues et les conditions restant à clarifier sont
consignées dans sources.csv. Une source incompatible avec l'usage prévu
devra être remplacée ou faire l'objet d'une autorisation adaptée.

### Sources

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

Décision :
Alternative non retenue pour la première architecture ;
l'équipe privilégie la gestion par unités systemd et le rattrapage
possible des échéances manquées.

### Option B — Timer systemd sur Linux

Avantages :
- Planification dans une unité .timer associée à une unité .service.
- Rattrapage possible d'un déclenchement manqué avec Persistent=true.
- Configuration déclarative pouvant être versionnée.

Limites :
- Nécessite une machine Linux utilisant systemd.
- Ne fournit pas à lui seul un suivi détaillé des étapes du pipeline.
- Un rattrapage après redémarrage ne rétablit pas le respect de 7 h
  si l'édition aurait dû être publiée pendant l'arrêt.
- Le rattrapage d'une échéance ne remplace pas une politique de reprise
  après échec du traitement.

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

Décision :
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

Décision :
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

Les PC Windows de l'équipe utiliseront WSL 2 pour développer et tester
la configuration avec systemd activé.

WSL reste un environnement de développement et de test.
La production devra fonctionner sur une infrastructure disponible
en continu.

### Sources

- systemd.timer :
  https://www.freedesktop.org/software/systemd/man/latest/systemd.timer.html
- Architecture Apache Airflow :
  https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/overview.html
- Événements GitHub Actions, section schedule :
  https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#schedule
- Microsoft — systemd dans WSL :
  https://learn.microsoft.com/en-us/windows/wsl/systemd

## 3. Stocker les données brutes et propres

Responsable : Walter.
Relecture : Darrel.
Statut : JSONL et SQLite validés pour la première architecture.

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

### Choix retenu et justification

Retenir JSONL pour les métadonnées brutes autorisées et SQLite
pour les données nettoyées, les thèmes et les éditions.

JSONL conserve les valeurs avant nettoyage et facilite le retraitement.
SQLite répond aux besoins de requêtes et d'unicité des données propres.

Conditions de mise en œuvre :
- Exécution sur une seule machine au démarrage.
- Écritures coordonnées pour limiter la concurrence.
- Stockage persistant indépendant du cycle de vie des conteneurs.
- Sauvegardes, restauration et politique de rétention à définir.
- Données collectées et bases exclues du dépôt Git.
- Droits de réutilisation des sources à clarifier avant exploitation.

En développement, le code et les données d'exécution seront placés
dans le système de fichiers Linux de WSL 2. Chaque membre disposera
de ses données de test locales.

Les mesures de persistance, de sauvegarde et de restauration restent
à mettre en œuvre et à tester lors des TD suivants.

### Sources

- JSON Lines :
  https://jsonlines.org/
- SQLite — Appropriate Uses :
  https://www.sqlite.org/whentouse.html
- PostgreSQL — JSON Types :
  https://www.postgresql.org/docs/current/datatype-json.html
- Apache Parquet :
  https://parquet.apache.org/
- Microsoft — Systèmes de fichiers Windows et Linux :
  https://learn.microsoft.com/en-us/windows/wsl/filesystems

## 4. Regrouper les textes par thèmes

Responsable : Yannick.
Relecture : Walter.
Statut : approche TF-IDF avec regroupement hiérarchique à distance
cosinus validée par Yannick pour la première architecture.
Les paramètres et la qualité des résultats restent à évaluer.

### Besoin

Regrouper les publications du jour traitant de sujets proches,
à partir des titres et extraits autorisés, sans récupérer le texte
intégral et sans générer de résumés avec un LLM.

L'édition doit présenter les groupes, leurs sources et les liens
vers les publications.

### Option A — Règles et mots-clés

Principe :
Définir des listes de mots et des règles pour affecter les publications
à des thèmes préparés par l'équipe.

Avantages :
- Décisions faciles à expliquer.
- Point de comparaison simple pour évaluer les autres approches.

Limites :
- Règles à entretenir.
- Difficulté à couvrir les nouveaux sujets et les formulations variées.
- Catégories générales pouvant mélanger des événements distincts.

Décision :
Ne pas retenir comme méthode principale.
Conserver comme référence simple pour l'évaluation.

### Option B — TF-IDF et K-means

Principe :
Représenter les titres et extraits en vecteurs TF-IDF,
puis répartir les publications dans un nombre choisi de groupes.

Avantages :
- Approche documentée pour le regroupement de textes.
- Termes caractéristiques utilisables pour examiner les groupes.

Limites :
- Nombre de groupes à déterminer.
- Sensibilité à l'initialisation.
- Difficultés possibles sur des textes courts et de grande dimension.
- Similarité lexicale ne garantissant pas un même sujet.

Décision :
Ne pas retenir comme méthode principale.
Conserver comme alternative expérimentale.

### Option C — TF-IDF et regroupement hiérarchique

Principe :
Représenter les titres et extraits avec TF-IDF, puis regrouper
les publications par distance cosinus avec un seuil de distance.

La configuration envisagée utilise une liaison moyenne,
n_clusters=None et un distance_threshold à déterminer par évaluation.
Le linkage Ward ne sera pas utilisé avec la distance cosinus.

Avantages :
- Pas de nombre fixe de groupes à imposer chaque matin.
- Seuil permettant d'ajuster le niveau de rapprochement.
- Termes caractéristiques disponibles pour examiner les résultats.

Limites :
- Seuil à régler sur un échantillon représentatif.
- Coût de calcul et mémoire à mesurer selon le volume quotidien.
- Risque de séparer des formulations différentes du même sujet
  ou de réunir des sujets partageant un vocabulaire similaire.
- Traitement des textes vides et des groupes isolés à définir.

Décision :
Approche retenue et validée par Yannick pour la première architecture.

Le seuil de distance, le prétraitement et le traitement des publications
isolées seront définis à partir des évaluations prévues.
Aucun résultat de performance n'est encore revendiqué.

### Option D — Représentations sémantiques préentraînées et regroupement

Principe :
Utiliser un modèle de représentation de phrases pour produire
des vecteurs, puis appliquer un algorithme de regroupement.

Avantage recherché :
Mieux rapprocher des formulations différentes d'un même sujet.

Limites à étudier :
- Modèle francophone ou multilingue à sélectionner et licence à vérifier.
- Ressources CPU, mémoire, taille du modèle et durée à mesurer.
- Gain de pertinence à démontrer par rapport à TF-IDF.
- Modèle et paramètres à versionner.
- Aucune génération de résumé ne sera ajoutée.

Décision :
Ne pas retenir pour la première architecture.
Réexaminer uniquement si les résultats justifient une amélioration.

### Fonctionnement retenu — paramètres à évaluer

1. Sélectionner les publications propres admissibles pour l'édition.
2. Construire le texte de travail : titre et extrait.
3. Appliquer un prétraitement documenté et adapté au français.
4. Produire les vecteurs TF-IDF.
5. Traiter les textes sans représentation exploitable selon une règle
   explicite avant le calcul des distances.
6. Regrouper par distance cosinus avec un seuil évalué.
7. Extraire des termes caractéristiques pour proposer des libellés.
8. Associer les sources et les liens à chaque groupe.

Les libellés issus des termes caractéristiques ne seront pas présentés
comme des résumés générés des articles.

### Évaluation prévue

- Préparer un petit échantillon de publications annoté par l'équipe.
- Vérifier si les publications réunies traitent réellement du même sujet.
- Examiner aussi les publications proches séparées à tort.
- Comparer plusieurs seuils et l'alternative K-means.
- Mesurer le temps d'exécution et l'usage mémoire.
- Définir le traitement des groupes d'une seule publication.
- Tracer les données d'entrée, les paramètres, les versions,
  les métriques et les résultats de chaque expérience.

Aucun résultat de performance n'est encore revendiqué.

### Sources

- scikit-learn — Regroupement de documents avec K-means :
  https://scikit-learn.org/stable/auto_examples/text/plot_document_clustering.html
- scikit-learn — AgglomerativeClustering :
  https://scikit-learn.org/stable/modules/generated/sklearn.cluster.AgglomerativeClustering.html

La documentation d'un éventuel modèle sémantique sera ajoutée
si cette option est étudiée.

## 5. Surveiller le pipeline

Responsable : Yannick.
Relecture : Walter.
Statut : architecture de supervision validée par Yannick.
La configuration, les seuils et le canal de notification restent
à définir ; les composants ne sont pas encore déployés ni testés.

### Besoin

Savoir si le pipeline a fonctionné sans intervention humaine,
identifier les étapes en échec et vérifier que l'édition attendue
est disponible à 7 h avec des articles de moins de 24 h.

Distinguer :
- les métriques : valeurs numériques suivies dans le temps ;
- les journaux : événements détaillés pour le diagnostic ;
- les alertes : notifications nécessitant une intervention ;
- le journal de bord : suivi humain des incidents et correctifs.

### Option A — Journaux systemd et script de contrôle

Principe :
Consulter les journaux du service avec journalctl et prévoir
un contrôle spécifique du résultat de l'exécution.

Avantages :
- Réutilise l'environnement systemd déjà retenu.
- Permet de consulter les erreurs du service.
- Peu de composants supplémentaires.

Limites :
- Historique de métriques et tableaux de bord à construire séparément.
- Un contrôle exécuté seulement par le pipeline ne détecte pas
  nécessairement que le pipeline n'a pas démarré.
- Notifications et vérification de l'édition à organiser.

Avis proposé :
Conserver les journaux systemd pour le diagnostic, mais ne pas
les utiliser comme seul dispositif de supervision.

### Option B — Prometheus, Grafana et Pushgateway

Principe :
Le pipeline transmet ses métriques à Pushgateway.
Prometheus les collecte et conserve leur historique.
Grafana les affiche et évalue les règles d'alerte.

Avantages :
- Historique des métriques et tableaux de bord.
- Alertes fondées sur les résultats et leur fraîcheur.
- Adaptation aux traitements batch qui ne restent pas actifs
  assez longtemps pour être interrogés directement.
- Possibilité de versionner les configurations.

Limites :
- Plusieurs composants supplémentaires à déployer et maintenir.
- Stockage persistant et rétention à configurer.
- Les métriques conservées par Pushgateway peuvent devenir anciennes.
- Une panne de toute l'infrastructure peut rendre la supervision
  locale indisponible.

Décision :
Solution retenue et validée par Yannick pour les métriques,
les tableaux de bord et les alertes, avec des journaux structurés
en complément.

### Option C — Prometheus, Grafana et Node Exporter textfile collector

Principe :
Le pipeline écrit ses métriques dans un fichier local exposé
par le textfile collector de Node Exporter.

Avantages :
- Pas de Pushgateway nécessaire.
- Possibilité de regrouper supervision de la machine
  et indicateurs de traitements locaux.

Limites :
- Métriques liées au système de fichiers de la machine.
- Fichier à publier de manière cohérente et à surveiller.
- Dates de mise à jour à contrôler pour éviter les faux succès.
- Prometheus recommande cette approche pour les traitements
  liés à une machine ; notre édition est un résultat de service.

Avis proposé :
Alternative pertinente si l'équipe choisit une supervision
principalement centrée sur l'hôte ; non privilégiée pour l'instant.

### Option D — Service de supervision hébergé

Principe :
Utiliser un service externe de suivi, de tableaux de bord
ou de réception de signaux d'exécution.

Avantages recherchés :
- Réduction de certains composants à héberger.
- Possibilité de détecter une indisponibilité de la machine
  depuis un système externe.

Limites à vérifier :
- Offre gratuite, quotas et durée de rétention.
- Gestion des comptes, secrets et données transmises.
- Dépendance à un service tiers.
- Capacité à suivre les contrôles métier propres à l'édition.

Avis proposé :
Ne pas retenir comme socle principal sans fournisseur identifié
et conditions vérifiées. Réexaminer pour un contrôle externe
si l'infrastructure le permet.

### Décision retenue

Architecture de supervision validée par Yannick :
- Prometheus pour collecter et conserver les métriques ;
- Pushgateway pour exposer les résultats des traitements batch ;
- Grafana pour les tableaux de bord et les alertes ;
- des journaux applicatifs structurés pour le diagnostic ;
- journalctl pour consulter les journaux du service systemd.

Grafana Alerting est retenu comme unique moteur d'alerte initial.
Aucun second moteur ne sera ajouté sans besoin identifié.

La supervision devra contrôler l'édition effectivement publiée,
sa fraîcheur et les dates des dernières exécutions.
Un ancien résultat de succès ne devra pas masquer l'absence
d'une nouvelle édition.

Le canal de notification, les seuils, la rétention et les configurations
seront définis et testés lors de la réalisation.

### Indicateurs envisagés

Exécution :
- Date de la dernière tentative.
- Date de la dernière exécution réussie.
- Résultat et durée des différentes étapes.
- Nombre d'échecs de collecte par source.

Données :
- Nombre d'items reçus par source lors de la dernière collecte.
- Nombre d'articles conservés, rejetés et dédupliqués.
- Nombre de sources représentées dans l'édition.
- Nombre d'articles sans date ou avec date non exploitable.

Publication :
- Date de l'édition effectivement publiée.
- Date et heure de publication.
- Présence de l'édition attendue et possibilité de la lire.
- Âge maximal des articles au moment de la publication.

Les contrôles de publication ne doivent pas reposer uniquement
sur le succès de la collecte ou du regroupement.

### Alertes envisagées

- Échec d'une étape après les tentatives prévues.
- Absence de nouvelle exécution dans la fenêtre attendue.
- Édition du jour absente ou indisponible à l'échéance.
- Présence d'un article de 24 h ou plus dans l'édition publiée.
- Source en échec ou sans résultat, avec seuil à évaluer.
- Couverture insuffisante des sources selon une règle à définir.

Le fuseau de référence et la fréquence d'évaluation des alertes
seront fixés explicitement.

### Précautions de réalisation

- Suivre les dates de dernière réussite et de publication :
  une ancienne valeur de succès ne prouve pas l'exécution du jour.
- Définir le traitement des métriques absentes et des erreurs
  de la source de données.
- Ne pas utiliser un identifiant unique d'exécution ou une URL
  d'article comme label Prometheus.
- Conserver les identifiants d'exécution dans les journaux.
- Éviter les groupes Pushgateway créés pour chaque exécution.
- Prévoir la suppression des métriques devenues obsolètes.
- Ne pas effacer systématiquement les résultats dès la fin du batch :
  Prometheus doit pouvoir les collecter.
- Ne pas inclure d'extraits d'articles, de secrets ou de profils
  dans les journaux techniques.
- Définir les rétentions et la persistance des métriques et journaux.
- Configurer le canal de notification et tester sa réception.
- Protéger l'accès aux composants de supervision.

Une supervision hébergée sur la même machine ne peut pas garantir
la notification d'une panne complète de cette machine.
Ce risque résiduel sera documenté.

### Tests prévus

- Simuler une source indisponible.
- Simuler l'échec d'une étape.
- Ne pas lancer le pipeline et vérifier la détection de l'absence.
- Laisser une ancienne réussite et vérifier qu'elle ne masque pas
  l'absence d'édition du jour.
- Publier une édition de test et vérifier le tableau de bord.
- Vérifier la réception d'une notification.
- Tester le comportement en cas de métriques absentes.

Aucun composant n'est déclaré installé ni testé à ce stade.

### Sources

- Prometheus — Quand utiliser Pushgateway :
  https://prometheus.io/docs/practices/pushing/
- Prometheus — Métriques des traitements courts :
  https://prometheus.io/docs/instrumenting/pushing/
- Grafana — Alertes avec une source Prometheus :
  https://grafana.com/docs/grafana/latest/datasources/prometheus/alerting/
- systemd — journalctl :
  https://www.freedesktop.org/software/systemd/man/journalctl.html

## 6. Décrire l'infrastructure en code

Responsable : Darrel.
Relecture : Yannick.
Statut : à compléter et à valider.

À compléter :
- Comparer les solutions adaptées à l'hébergement envisagé.
- Justifier la solution retenue et les alternatives écartées.
- Préciser la gestion de la configuration, des secrets et de la persistance.
- Articuler le déploiement avec les unités systemd.
- Ajouter les références consultées.