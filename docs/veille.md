# Veille technique — TD 1

## Synthèse

Cette veille compare les solutions envisagées pour La gazette,
leurs limites et les raisons des choix retenus.

Les choix décrivent la première architecture cible.
Ils ne signifient pas que les composants sont déjà installés ou testés.

| Sujet | Choix retenu | Responsable |
|---|---|---|
| Collecte | Flux RSS des éditeurs | Walter |
| Planification | Timer systemd | Darrel |
| Stockage | JSONL pour le brut autorisé ; SQLite pour le propre | Walter |
| Thèmes | TF-IDF et classification hiérarchique à distance cosinus | Yannick |
| Supervision | Prometheus, Pushgateway, Grafana et journaux structurés | Yannick |
| Infrastructure | Ansible, Docker et Docker Compose | Darrel |
| Langage | Python | Walter / équipe |
| CI/CD | GitHub Actions et déploiement contrôlé avec Ansible | Darrel |
| Publication | Édition HTML statique servie par Nginx, avec archives datées | Walter / Darrel |

Réserves transversales :
- Les droits de réutilisation des sources restent à clarifier.
- Le système, les ressources et les accès du serveur fourni
  restent à confirmer.
- WSL 2 constitue l'environnement de développement et de test,
  pas la cible de production disponible en continu.
- Les paramètres opérationnels seront précisés et testés
  lors des TD suivants.

## 1. Récupérer les publications de presse

Responsable : Walter.
Relecture : Darrel.
Statut : collecte RSS retenue ; huit flux inspectés manuellement ;
conditions de réutilisation à finaliser.

### Besoin

Récupérer les titres, extraits, liens et dates des publications
d'au moins quatre PQR et quatre PQN, sans collecter le texte intégral
et dans le respect des conditions des éditeurs.

### Option A — Flux RSS/Atom des éditeurs

Principe :
Lire les flux de syndication publiés directement par les journaux.

Avantages :
- Publications structurées.
- Accès direct aux sources sélectionnées.
- Sélection des seuls champs autorisés.

Limites :
- Disponibilité et contenu variables selon les flux.
- Dates ou extraits potentiellement absents.
- Articles anciens pouvant rester présents dans un flux récent.
- Conditions de réutilisation à examiner pour chaque source.
- Contenu hors périmètre à exclure du stockage et des journaux.

Décision :
Retenir les flux RSS des éditeurs.
Les huit sources ont été inspectées manuellement et les observations
sont consignées dans sources.csv.

### Option B — API officielle d'un éditeur

Principe :
Utiliser une interface documentée proposée par un journal.

Avantages :
- Accès structuré lorsqu'une API existe.
- Documentation des champs et modalités d'accès.

Limites :
- Existence et accessibilité à vérifier pour chaque journal.
- Authentification, quotas ou coûts possibles.
- Intégrations différentes selon les éditeurs.

Décision :
Non retenue : aucun accès adapté aux huit sources et au budget nul
n'a été confirmé.

### Option C — API d'un agrégateur de presse

Principe :
Interroger un service centralisant plusieurs médias.

Avantages :
- Point d'accès commun.
- Recherche et filtres selon le fournisseur.

Limites :
- Couverture des huit journaux à vérifier.
- Présence des extraits et dates à contrôler.
- Quotas, fraîcheur et conditions variables.
- Dépendance à un intermédiaire.
- Gratuité en développement ne garantissant pas l'usage en production.

Décision :
Non retenue : adéquation au périmètre et au budget non établie.

### Option D — Extraction des pages HTML

Principe :
Extraire les métadonnées depuis les pages des journaux.

Limites :
- Extraction des pages non retenue dans notre périmètre.
- Scraping du texte intégral explicitement exclu.

Décision :
Option écartée au profit des flux publiés par les éditeurs.

### Choix retenu et conditions

Les flux RSS permettent de récupérer les métadonnées nécessaires
sans extraire les pages des articles.

Les titres, extraits, liens, dates et publications du jour ont été
observés dans les huit flux. Ces observations ne garantissent pas
la présence future de tous les champs.

La date de chaque item sera contrôlée indépendamment de celle du flux.

La vérification technique ne vaut pas autorisation de réutilisation.
Les restrictions sont documentées dans sources.csv.
Toute source incompatible devra être remplacée ou faire l'objet
d'une autorisation adaptée avant exploitation.

### Sources

- RSS :
  https://cyber.harvard.edu/rss/rss.html
- Atom :
  https://datatracker.ietf.org/doc/html/rfc4287
- GDELT DOC :
  https://blog.gdeltproject.org/gdelt-doc-2-0-api-debuts/
- NewsAPI :
  https://newsapi.org/pricing

## 2. Exécuter le pipeline chaque matin

Responsable : Darrel.
Relecture : Yannick.
Statut : timer systemd validé par Darrel ;
compatibilité du serveur à confirmer.

### Besoin

Déclencher les traitements assez tôt pour publier l'édition à 7 h,
suivre leur résultat et permettre une relance contrôlée.

### Option A — Cron sur Linux

Avantages :
- Déclenchement léger à heure fixe.
- Configuration simple à versionner.

Limites :
- Cron standard ne rattrape pas une échéance manquée pendant un arrêt.
- Journaux, reprises, alertes et dépendances à organiser séparément.

Décision :
Non retenu au profit de la gestion par unités systemd
et du rattrapage possible des échéances.

### Option B — Timer systemd

Avantages :
- Unité .timer associée à une unité .service.
- Rattrapage possible avec Persistent=true.
- Configuration versionnable.

Limites :
- Nécessite Linux avec systemd.
- Ne fournit pas seul le suivi métier des étapes.
- Un rattrapage tardif ne rétablit pas le respect de 7 h.
- Le rattrapage d'une échéance ne remplace pas une reprise après échec.

Décision :
Retenu pour la production, sous réserve des caractéristiques
et des droits disponibles sur le serveur fourni.

### Option C — Apache Airflow

Avantages :
- Dépendances entre tâches décrites par un graphe.
- Interface de suivi, journaux et déclenchements manuels.

Limites :
- Plusieurs composants à exploiter.
- Base de métadonnées nécessaire.
- Complexité supplémentaire pour notre première architecture.

Décision :
Non retenu au démarrage ; à réexaminer si l'orchestration dépasse
une exécution séquentielle maîtrisée.

### Option D — GitHub Actions planifié

Avantages :
- Configuration versionnée.
- Possibilité de runner hébergé.

Limites :
- Déclenchements pouvant être retardés ou abandonnés sous forte charge.
- Quotas et conditions à vérifier.
- Persistance et accès à la production à organiser.

Décision :
Non retenu comme ordonnanceur de l'édition de 7 h.
GitHub Actions est en revanche retenu pour la CI et la préparation
des versions, comme décrit en section 8.

### Choix retenu et conditions

Retenir systemd et prévoir :
- un déclenchement avant 7 h avec une marge fondée sur les mesures ;
- un fuseau de publication explicite ;
- un identifiant et un statut par exécution ;
- des reprises bornées et des délais maximaux ;
- une protection contre les exécutions concurrentes ;
- une relance sans doublons ;
- une alerte si l'édition attendue est absente.

Les tests locaux utiliseront WSL 2 avec systemd activé.
La production devra fonctionner sur une infrastructure disponible
en continu.

### Sources

- systemd.timer :
  https://www.freedesktop.org/software/systemd/man/latest/systemd.timer.html
- Apache Airflow :
  https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/overview.html
- GitHub Actions, schedule :
  https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#schedule
- systemd dans WSL :
  https://learn.microsoft.com/en-us/windows/wsl/systemd

## 3. Stocker les données brutes et propres

Responsable : Walter.
Relecture : Darrel.
Statut : JSONL et SQLite validés.

### Besoin

Conserver les métadonnées autorisées avant nettoyage pour permettre
la traçabilité et le retraitement, puis stocker les données propres
pour les thèmes et les éditions.

Le brut n'est pas une copie intégrale du XML :
images et texte intégral sont exclus.

### Option A — Fichiers JSONL

Avantages :
- Un objet JSON par ligne.
- Conservation des valeurs autorisées avant nettoyage.
- Relecture des lots pour rejouer les traitements.

Limites :
- Déduplication et cohérence à développer.
- Requêtes et jointures moins pratiques qu'en base SQL.
- Gestion des fichiers et de leur rétention à prévoir.

Décision :
Retenir pour les métadonnées brutes autorisées.

### Option B — SQLite

Avantages :
- Pas de serveur de base à administrer.
- Requêtes SQL.
- Contraintes d'unicité.
- Adaptation à une exécution locale avec peu d'écritures concurrentes.

Limites :
- Un seul écrivain à la fois par base.
- Accès direct depuis plusieurs machines à éviter.
- Persistance et sauvegarde cohérente à organiser.

Décision :
Retenir pour les données propres, les thèmes et les éditions,
avec écritures coordonnées sur une machine unique.

### Option C — PostgreSQL

Avantages :
- Architecture client-serveur.
- Stockage relationnel et JSONB.
- Possibilités d'indexation de JSONB.

Limites :
- Service supplémentaire à configurer et exploiter.
- Accès, sauvegardes et mises à jour à organiser.
- Complexité supérieure à la solution locale envisagée.

Décision :
Non retenu au démarrage.
À réexaminer si des accès distribués ou davantage d'écritures
concurrentes deviennent nécessaires.

### Option D — Parquet

Avantages :
- Format en colonnes avec compression.
- Adapté aux analyses par lots.

Limites :
- Ne fournit pas seul les contraintes d'une base relationnelle.
- Mises à jour, doublons et fichiers à gérer.
- Aucun besoin initial justifiant ce stockage principal.

Décision :
Non retenu comme stockage opérationnel principal.
À réexaminer pour des exports analytiques.

### Choix retenu et conditions

JSONL conserve les valeurs avant nettoyage.
SQLite permet les requêtes et les contraintes sur les données propres.

Prévoir :
- des écritures coordonnées ;
- un stockage persistant hors du cycle de vie des conteneurs ;
- des sauvegardes et tests de restauration ;
- une politique de rétention ;
- l'exclusion des données et bases du dépôt Git.

Sous WSL 2, le code et les données d'exécution seront placés dans
le système de fichiers Linux. Chaque membre utilisera ses données
de test locales.

Ces mesures restent à réaliser et à tester.

### Sources

- JSON Lines :
  https://jsonlines.org/
- SQLite :
  https://www.sqlite.org/whentouse.html
- PostgreSQL JSON :
  https://www.postgresql.org/docs/current/datatype-json.html
- Parquet :
  https://parquet.apache.org/
- Fichiers Windows et Linux :
  https://learn.microsoft.com/en-us/windows/wsl/filesystems

## 4. Regrouper les textes par thèmes

Responsable : Yannick.
Relecture : Walter.
Statut : TF-IDF et classification hiérarchique à distance cosinus
validés par Yannick ; paramètres et performances à évaluer.

### Besoin

Regrouper les publications par sujets proches à partir des titres
et extraits autorisés, sans texte intégral ni résumé généré par LLM.

Présenter les groupes avec leurs sources et liens.

### Option A — Règles et mots-clés

Avantages :
- Décisions explicables.
- Référence simple pour l'évaluation.

Limites :
- Entretien manuel.
- Couverture limitée des nouveaux sujets et formulations.
- Risque de mélanger plusieurs événements dans une catégorie générale.

Décision :
Non retenus comme méthode principale ; conservés comme référence.

### Option B — TF-IDF et K-means

Avantages :
- Approche documentée de regroupement de textes.
- Termes caractéristiques disponibles pour examiner les groupes.

Limites :
- Nombre de groupes à choisir.
- Sensibilité à l'initialisation.
- Difficultés possibles avec les textes courts.
- Proximité lexicale ne garantissant pas un même sujet.

Décision :
Non retenu comme méthode principale ; conservé comme alternative
expérimentale.

### Option C — TF-IDF et classification hiérarchique

Principe :
Utiliser la distance cosinus, une liaison moyenne et un seuil
de distance évalué, avec n_clusters=None.

Le linkage Ward ne sera pas utilisé avec la distance cosinus.

Avantages :
- Nombre de groupes variable selon les publications.
- Niveau de rapprochement ajustable par le seuil.
- Termes caractéristiques disponibles pour examiner les résultats.

Limites :
- Seuil à régler sur un échantillon représentatif.
- Temps et mémoire à mesurer selon le volume.
- Erreurs possibles de fusion ou de séparation des sujets.
- Textes vides et publications isolées à traiter explicitement.

Décision :
Approche retenue et validée par Yannick.
Aucune performance n'est encore revendiquée.

### Option D — Représentations sémantiques préentraînées

Principe :
Produire des vecteurs de phrases avec un modèle préentraîné,
puis appliquer un regroupement.

Avantage recherché :
Rapprocher des formulations différentes du même sujet.

Limites :
- Modèle et licence à vérifier.
- Ressources et durée à mesurer.
- Gain par rapport à TF-IDF à démontrer.
- Versions et paramètres à tracer.

Décision :
Non retenues pour la première architecture.
À réexaminer si l'évaluation justifie une amélioration.
Aucune génération de résumé ne sera ajoutée.

### Fonctionnement retenu

1. Sélectionner les publications propres admissibles pour l'édition.
2. Construire le texte de travail avec le titre et l'extrait.
3. Appliquer un prétraitement documenté et adapté au français.
4. Produire les vecteurs TF-IDF.
5. Traiter les textes sans représentation exploitable.
6. Regrouper par distance cosinus avec un seuil évalué.
7. Extraire des termes caractéristiques pour proposer des libellés.
8. Associer sources et liens aux groupes.

Les libellés ne seront pas présentés comme des résumés générés.

### Évaluation prévue

- Constituer un échantillon annoté par l'équipe.
- Examiner les publications regroupées à tort et séparées à tort.
- Comparer plusieurs seuils et l'alternative K-means.
- Mesurer temps et mémoire.
- Définir le traitement des groupes d'une seule publication.
- Tracer données d'entrée, paramètres, versions et résultats.

### Sources

- Regroupement de documents :
  https://scikit-learn.org/stable/auto_examples/text/plot_document_clustering.html
- Classification hiérarchique :
  https://scikit-learn.org/stable/modules/generated/sklearn.cluster.AgglomerativeClustering.html

La référence d'un modèle sémantique sera ajoutée si cette piste
est effectivement étudiée.

## 5. Surveiller le pipeline

Responsable : Yannick.
Relecture : Walter.
Statut : architecture validée par Yannick ;
configuration, seuils et canal de notification à définir.

### Besoin

Vérifier le fonctionnement sans intervention humaine, diagnostiquer
les échecs et contrôler l'édition attendue à 7 h.

Distinguer :
- métriques : valeurs numériques suivies dans le temps ;
- journaux : événements techniques pour le diagnostic ;
- alertes : notifications appelant une intervention ;
- journal de bord : suivi humain des incidents et correctifs.

### Option A — Journaux systemd et script de contrôle

Avantages :
- Réutilisation de systemd.
- Consultation des erreurs du service.
- Peu de composants supplémentaires.

Limites :
- Historique de métriques et tableaux de bord à construire.
- Contrôle lancé uniquement par le pipeline insuffisant pour détecter
  son absence de démarrage.
- Notifications à organiser.

Décision :
Conserver les journaux pour le diagnostic, pas comme unique supervision.

### Option B — Prometheus, Grafana et Pushgateway

Principe :
Le batch transmet ses métriques à Pushgateway.
Prometheus les collecte et Grafana les affiche et évalue les alertes.

Avantages :
- Historique et tableaux de bord.
- Alertes fondées sur les résultats et leur fraîcheur.
- Adaptation aux traitements courts.
- Configurations versionnables.

Limites :
- Composants supplémentaires à exploiter.
- Persistance et rétention à configurer.
- Métriques Pushgateway pouvant devenir anciennes.
- Supervision locale indisponible si toute la machine tombe en panne.

Décision :
Solution retenue et validée par Yannick, avec journaux structurés.

### Option C — Prometheus, Grafana et textfile collector

Principe :
Écrire les métriques dans un fichier exposé par le textfile collector
de Node Exporter.

Avantages :
- Pas de Pushgateway nécessaire.
- Possibilité de suivre l'hôte et ses traitements locaux.

Limites :
- Dépendance aux fichiers de la machine.
- Publication cohérente et fraîcheur des fichiers à contrôler.
- Approche orientée traitements liés à l'hôte, alors que notre édition
  constitue un résultat de service.

Décision :
Non privilégiée pour le suivi du batch.
Une utilisation ultérieure pour les métriques de l'hôte reste possible.

### Option D — Supervision hébergée

Avantages recherchés :
- Moins de composants locaux à exploiter.
- Possibilité de contrôle depuis l'extérieur du serveur.

Limites :
- Fournisseur, gratuité, quotas et rétention non vérifiés.
- Secrets, comptes et données transmises à gérer.
- Dépendance supplémentaire.
- Contrôles métier à intégrer.

Décision :
Non retenue comme socle principal.
À réexaminer pour un contrôle externe.

### Architecture retenue

- Prometheus pour les métriques.
- Pushgateway pour les résultats du batch.
- Grafana pour les tableaux de bord et les alertes.
- Journaux applicatifs structurés pour le diagnostic.
- journalctl pour les journaux du service systemd.

Grafana Alerting constitue l'unique moteur d'alerte initial.

### Indicateurs envisagés

Exécution :
- Dates de dernière tentative et dernière réussite.
- Résultat et durée des étapes.
- Échecs par source.

Données :
- Items reçus par source.
- Articles conservés, rejetés et dédupliqués.
- Sources représentées dans l'édition.
- Dates absentes ou non exploitables.

Publication :
- Date de l'édition et heure de publication.
- Présence et lisibilité de l'édition attendue.
- Âge maximal des articles au moment de la publication.

### Alertes envisagées

- Échec après les tentatives prévues.
- Absence d'exécution dans la fenêtre attendue.
- Édition absente ou indisponible à l'échéance.
- Article de 24 h ou plus au moment de la publication.
- Source en échec ou sans résultat, selon des seuils à évaluer.
- Couverture insuffisante selon une règle à définir.

Le fuseau de référence et la fréquence d'évaluation seront explicites.

### Précautions

- Contrôler les dates : un ancien succès ne prouve pas l'exécution du jour.
- Définir le comportement en cas de métriques absentes.
- Ne pas utiliser les identifiants d'exécution ou URLs d'articles
  comme labels Prometheus.
- Conserver les identifiants d'exécution dans les journaux.
- Éviter un groupe Pushgateway par exécution.
- Gérer les métriques obsolètes.
- Ne pas effacer les résultats avant leur collecte par Prometheus.
- Exclure secrets, extraits et profils des journaux techniques.
- Définir persistance, rétention et permissions.
- Configurer et tester le canal de notification.

Une supervision sur la même machine ne garantit pas la notification
d'une panne complète de celle-ci. Ce risque reste documenté.

### Tests prévus

- Source indisponible et étape en échec.
- Absence totale de lancement.
- Ancienne réussite ne masquant pas une édition manquante.
- Publication de test visible dans le tableau de bord.
- Réception d'une notification.
- Métriques absentes.

Aucun composant n'est déclaré déployé ni testé.

### Sources

- Pushgateway :
  https://prometheus.io/docs/practices/pushing/
- Métriques des traitements courts :
  https://prometheus.io/docs/instrumenting/pushing/
- Grafana Alerting avec Prometheus :
  https://grafana.com/docs/grafana/latest/datasources/prometheus/alerting/
- journalctl :
  https://www.freedesktop.org/software/systemd/man/journalctl.html

## 6. Décrire l'infrastructure en code

Responsable : Darrel.
Relecture : Yannick.
Statut : Ansible, Docker et Docker Compose validés par Darrel.
Compatibilité, ressources et droits du serveur fourni à confirmer.

### Besoin

Versionner la préparation de l'environnement, le déploiement,
les configurations, la persistance et la planification.

Permettre de reconstruire l'environnement sans dépendre
d'une succession d'opérations manuelles non documentées.

La cible envisagée est un serveur Linux existant fourni ultérieurement.

### Option A — Scripts shell et Docker Compose

Principe :
Préparer l'hôte par scripts et décrire les conteneurs avec Compose.

Avantages :
- Approche directe.
- Fichiers versionnés.
- Peu d'outils supplémentaires.

Limites :
- Gestion de l'état existant à développer.
- Réexécution et traitement des erreurs à sécuriser.
- Compose ne configure pas seul tout l'hôte.

Décision :
Ne pas retenir les scripts comme mécanisme principal de configuration.
Conserver des scripts courts pour des opérations ciblées.

### Option B — Ansible et Docker Compose

Principe :
Configurer l'hôte et déployer les fichiers avec Ansible ;
décrire les services, réseaux et volumes avec Compose.

Avantages :
- Séparation entre configuration de l'hôte et services applicatifs.
- Modules vérifiant l'état attendu.
- Déploiement des unités systemd et configurations depuis le dépôt.
- Gestion possible de Compose via community.docker.
- Réexécution structurée à vérifier.

Limites :
- Inventaire, accès et privilèges à configurer.
- Versions et dépendances à maîtriser.
- Idempotence non automatique pour toutes les tâches.
- Ne crée pas à elle seule le serveur.
- Ressources disponibles encore inconnues.

Décision :
Solution retenue et validée par Darrel.
Ansible configure l'hôte et déploie les fichiers.
Docker et Compose exécutent les services conteneurisés
avec stockage persistant.

### Option C — Terraform et outil de configuration

Principe :
Gérer les ressources via un fournisseur Terraform identifié,
puis configurer le système avec un outil adapté.

Avantage recherché :
Versionner aussi la création des machines et réseaux si nécessaire.

Limites :
- Fournisseur et droits de création non communiqués.
- Aucun besoin confirmé de créer des machines ou réseaux.
- État d'infrastructure et secrets à gérer.
- Ne remplace pas toute la configuration interne de l'hôte.

Décision :
Non retenu à ce stade.
Réexaminer si la plateforme fournie exige la création de ressources.

### Option D — Manifests Kubernetes

Principe :
Décrire les applications dans les ressources d'un cluster.

Avantage recherché :
Déployer dans un cluster lorsqu'il constitue la cible imposée.

Limites :
- Aucun cluster ni besoin multi-nœud confirmé.
- Complexité supplémentaire à justifier.
- Ne remplace pas seul la préparation de l'infrastructure.

Décision :
Non retenu pour la première architecture.
Réexaminer uniquement si la plateforme fournie l'impose.

### Architecture retenue

- Ansible pour configurer l'hôte Linux.
- Docker pour conteneuriser les composants.
- Docker Compose pour les services, réseaux et volumes.
- systemd pour le déclenchement quotidien.

Ansible déploiera les fichiers Compose, les configurations
de supervision et les unités systemd.

Le serveur devra permettre ces opérations.
Son système, ses ressources, ses accès et les privilèges disponibles
restent à confirmer.

### Organisation envisagée

- Conteneur de traitement exécuté à la demande.
- Services de supervision maintenus disponibles.
- JSONL, SQLite et données de supervision en stockage persistant.
- Écritures SQLite coordonnées.
- Chemins et paramètres adaptés à l'environnement.
- Secrets et données collectées exclus du dépôt.

SQLite reste un fichier persistant, pas un service de base distinct.

Le déclenchement quotidien relève de systemd et non d'une politique
de redémarrage permanent du conteneur de traitement.

La collecte des journaux sera explicitée :
journaux des services Compose et sortie du traitement consultable
depuis le service systemd.

### Développement sous WSL 2

Le projet utilisé pour l'exécution sera placé dans le système
de fichiers Linux de WSL 2.

Une procédure commune d'accès à Docker sera choisie et documentée.
Les installations concurrentes non maîtrisées seront évitées.

WSL permettra les tests locaux.
Les permissions, réseaux, démarrages et ressources seront également
vérifiés sur le serveur de production.

### Secrets et accès

- Ne pas commiter mots de passe, clés privées ou jetons.
- Versionner des exemples sans valeurs sensibles.
- Définir les permissions des fichiers et répertoires.
- Protéger Grafana, Prometheus et Pushgateway.
- Définir la fourniture des secrets avant déploiement.

### Vérifications prévues

- Validation de syntaxe.
- Déploiement sur un environnement de test.
- Réexécution et examen des changements.
- Persistance après recréation des conteneurs.
- Installation et activation du timer.
- Sauvegarde et restauration.
- Mise à jour et retour à une version précédente.

Aucun déploiement, test d'idempotence ou test de restauration
n'est déclaré réalisé au TD 1.

### Sources

- Ansible — Playbooks :
  https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html
- Ansible — Docker Compose :
  https://docs.ansible.com/ansible/latest/collections/community/docker/docker_compose_v2_module.html
- Docker Compose :
  https://docs.docker.com/compose/
- Terraform :
  https://developer.hashicorp.com/terraform/intro
- Kubernetes :
  https://kubernetes.io/docs/concepts/overview/

Les modalités propres à l'hébergement seront ajoutées
lorsqu'elles seront communiquées.

## 7. Choisir le langage du pipeline

Responsable : Walter.
Validation : équipe.
Statut : Python validé par l'équipe.
Comparaison documentaire qualitative ; aucun benchmark réalisé.

### Besoin

Utiliser un langage commun pour la collecte, le nettoyage,
le stockage, le regroupement par thèmes et la génération de l'édition.

Limiter les interfaces entre plusieurs langages et conserver
un environnement reproductible sous WSL 2 et dans les conteneurs.

### Option A — Python

Avantages :
- Bibliothèque standard comprenant notamment json, csv, logging
  et une interface SQLite via sqlite3.
- Écosystème adapté au traitement de textes et au machine learning.
- Possibilité d'utiliser scikit-learn pour TF-IDF et la classification
  hiérarchique déjà retenus.
- Possibilité de réunir les étapes du pipeline dans un même projet.

Limites :
- Dépendances et versions à maîtriser.
- Gestion explicite des erreurs, délais, reprises et transactions.
- Temps de calcul et mémoire du regroupement à mesurer.
- Le choix du langage ne garantit pas la robustesse opérationnelle.

Décision :
Python retenu par l'équipe pour le pipeline.

### Option B — JavaScript/TypeScript avec Node.js

Intérêt envisagé :
Constituer une autre approche pour développer la collecte,
les transformations et la génération de l'édition.

Limites dans notre architecture :
- Nécessiterait de reconsidérer l'implémentation du regroupement retenu
  ou de maintenir un composant Python séparé.
- Aucun besoin d'application JavaScript ne justifie ici l'ajout
  d'un second environnement de développement.

Décision :
Non retenu comme langage principal pour conserver la cohérence
avec les traitements de regroupement prévus en Python.

Ce choix ne signifie pas que Node.js est incapable de réaliser
un pipeline de données.

### Option C — Scripts shell comme langage principal

Intérêt envisagé :
Déclencher les commandes et réaliser des opérations système ciblées.

Limites dans notre architecture :
- L'équipe devrait assembler plusieurs outils pour traiter
  les flux, les dates, les données structurées et les thèmes.
- Les traitements métier et leurs tests seraient répartis entre
  plusieurs commandes plutôt que centralisés dans le projet Python.

Décision :
Ne pas retenir shell comme langage principal du pipeline.
Conserver des scripts courts pour les opérations système ciblées.

### Justification du choix

Python est retenu pour son adéquation aux choix déjà validés :
JSONL, SQLite et regroupement TF-IDF avec classification hiérarchique.

Un langage principal commun limite les interfaces entre les étapes.
Le traitement sera développé et testé sous WSL 2, puis exécuté
dans un environnement conteneurisé défini dans le dépôt.

Cette décision repose sur une comparaison qualitative d'adéquation
au projet, pas sur un benchmark de performances ni sur une supériorité
générale de Python sur les autres langages.

### Points à préciser au TD 2

- Version de Python compatible avec les dépendances retenues.
- Bibliothèque de lecture RSS.
- Bibliothèque d'accès HTTP et gestion des délais.
- Outillage de test.
- Déclaration et verrouillage des dépendances.
- Commande d'exécution et organisation des modules.
- Tests de compatibilité dans le conteneur.

Le choix de Python ne vaut pas validation automatique
de toutes les bibliothèques associées.

### Sources

- Bibliothèque standard Python :
  https://docs.python.org/3/library/index.html
- Interface SQLite :
  https://docs.python.org/3/library/sqlite3.html
- scikit-learn — Regroupement de documents :
  https://scikit-learn.org/stable/auto_examples/text/plot_document_clustering.html
- scikit-learn — Classification hiérarchique :
  https://scikit-learn.org/stable/modules/generated/sklearn.cluster.AgglomerativeClustering.html

## 8. Tester et livrer les modifications — CI/CD

Responsable : Darrel.
Relecture : Walter et Yannick.
Statut : stratégie validée par Darrel.
GitHub Actions est retenu pour la CI et la préparation des versions.
Le déploiement avec Ansible sera déclenché explicitement après validation.
Les workflows et la procédure de déploiement restent à réaliser et à tester.

### Besoin

Contrôler les modifications avant leur déploiement, produire
une version identifiable de l'application et permettre
un déploiement reproductible avec retour arrière.

La CI/CD est distincte de la planification quotidienne :
- la CI contrôle les modifications du projet ;
- la livraison prépare une version déployable ;
- le déploiement installe cette version ;
- systemd déclenche l'exécution quotidienne du pipeline.

### Option A — GitHub Actions

Principe :
Décrire les contrôles dans des workflows versionnés dans le dépôt.

Avantages :
- Intégration au dépôt GitHub déjà utilisé.
- Exécution sur push et pull request.
- Prise en charge des tests Python et de la construction d'images.
- Résultats associés aux modifications du code.

Limites :
- Quotas, stockage des artefacts et conditions du compte à vérifier.
- Dépendance à GitHub pour les contrôles hébergés.
- Accès au serveur et secrets de déploiement à protéger.
- Fonctionnalités de protection des environnements à vérifier
  selon l'offre et la visibilité du dépôt.

Décision :
Solution retenue et validée par Darrel pour les contrôles automatiques,
la construction des images et la préparation des versions.

### Option B — GitLab CI/CD

Principe :
Décrire les contrôles et livraisons dans des pipelines GitLab.

Avantages :
- Automatisation des tests, constructions et déploiements.
- Configuration versionnée.
- Exécution sur des runners gérés ou auto-hébergés.

Limites dans notre projet :
- Dépôt principal actuellement sur GitHub.
- Migration ou synchronisation supplémentaire à organiser.
- Quotas et runners à vérifier.

Décision :
Non retenu pour éviter d'ajouter une seconde plateforme
ou de migrer le dépôt existant.

### Option C — Jenkins

Principe :
Mettre en place un service Jenkins pour exécuter les pipelines.

Avantages :
- Automatisation des tests, constructions et déploiements.
- Possibilité de maîtriser l'environnement d'exécution.

Limites dans notre projet :
- Service supplémentaire à héberger, sécuriser et maintenir.
- Configuration des agents et accès à organiser.
- Charge d'exploitation non justifiée pour la première architecture.

Décision :
Non retenu au démarrage.

### Option D — Contrôles manuels uniquement

Principe :
Lancer les vérifications et constructions depuis les postes de l'équipe.

Avantages :
- Aucun service CI supplémentaire.
- Utile pour vérifier rapidement une modification avant un push.

Limites :
- Exécution non systématique.
- Résultats dépendants du poste.
- Risque d'oubli et preuves de contrôle dispersées.

Décision :
Conserver les contrôles locaux en complément,
mais pas comme seule validation avant livraison.

### Stratégie retenue

Choix validé par Darrel : utiliser GitHub Actions pour :
- installer l'environnement de test ;
- exécuter les contrôles de qualité ;
- exécuter les tests automatisés ;
- valider les configurations concernées ;
- construire l'image Docker.

Les contrôles seront déclenchés sur les push et pull requests
selon les règles définies dans les workflows.

Le déploiement de production sera déclenché explicitement
après réussite des contrôles et relecture.
Ansible appliquera la configuration et la version choisie.

Aucun déploiement automatique sur chaque push n'est retenu
pour la première architecture.

Systemd reste responsable du déclenchement quotidien du pipeline.
GitHub Actions n'est pas utilisé comme ordonnanceur de l'édition de 7 h.

Le registre d'images, les quotas, les accès au serveur et les modalités
exactes du déclenchement de déploiement restent à préciser.

### Contrôles envisagés

- Tests unitaires du nettoyage, des dates et de la déduplication.
- Tests des entrées invalides et champs absents.
- Tests du regroupement sur un jeu de données maîtrisé.
- Tests d'intégration avec une base SQLite temporaire.
- Validation du fichier sources.csv.
- Validation de la configuration Docker Compose.
- Vérification syntaxique des playbooks Ansible.
- Construction de l'image Docker.

Les bibliothèques, commandes et versions seront définies au TD 2.
Les tests métier seront ajoutés au fur et à mesure du développement
des composants concernés.

Les tests ne dépendront pas systématiquement des flux de presse réels.
Des exemples synthétiques ou des fixtures dont l'utilisation est
autorisée permettront de tester sans dépendance réseau.

### Livraison et déploiement

- Identifier chaque version par le commit Git correspondant.
- Utiliser une référence d'image identifiable et non uniquement latest.
- Définir le registre d'images ou le mécanisme de transfert.
- Vérifier ses conditions d'utilisation dans le budget nul.
- Déployer avec Ansible après validation explicite.
- Vérifier le fonctionnement après déploiement.
- Documenter le retour à une version précédente.

Le registre, le mode d'accès au serveur et l'automatisation exacte
du déploiement restent à choisir après connaissance de l'hébergement.

### Sécurité et budget

- Ne pas stocker les secrets dans le dépôt.
- Limiter les permissions des workflows et des accès au serveur.
- Ne pas exposer les secrets de production aux contributions
  non approuvées.
- Maîtriser les versions des actions et dépendances utilisées.
- Vérifier les quotas et éviter les dépenses automatiques.
- Définir une rétention limitée des artefacts.

### Vérifications prévues

- Un test volontairement en échec bloque la livraison.
- Un push ou une pull request déclenche les contrôles prévus.
- La version construite est identifiable.
- Un déploiement de test peut être vérifié et annulé.
- Les modifications de données et de schéma sont prises en compte
  dans la procédure de retour arrière.

Aucun workflow, déploiement ou retour arrière n'est déclaré
testé à ce stade.

### Sources

- GitHub Actions :
  https://docs.github.com/articles/getting-started-with-github-actions
- Tests Python :
  https://docs.github.com/actions/guides/building-and-testing-python
- Environnements de déploiement :
  https://docs.github.com/en/actions/concepts/workflows-and-actions/deployment-environments
- Comparaison avec GitLab CI/CD :
  https://docs.github.com/actions/learn-github-actions/migrating-from-gitlab-cicd-to-github-actions
- Comparaison avec Jenkins :
  https://docs.github.com/actions/learn-github-actions/migrating-from-jenkins-to-github-actions

## 9. Points transversaux restant à préciser

### Décisions de cadrage encore ouvertes

- Adresse de consultation de l'édition HTML, modalités HTTPS
  et éventuelle restriction d'accès.
- Durée de conservation et droits d'archivage des éditions.
- Fuseau de référence de l'édition.
- Conditions de réutilisation des sources encore non clarifiées.

### Paramètres à préciser lors de la réalisation

- Version de Python, bibliothèques et outillage de test.
- Organisation des modules et commande d'exécution.
- Workflows de CI et règles de déclenchement.
- Registre ou mécanisme de transfert des images.
- Quotas et rétention des artefacts.
- Accès au serveur et déclenchement du déploiement.
- Heure de démarrage du traitement et délais maximaux.
- Seuil de regroupement et règles pour les publications isolées.
- Seuils d'alerte et canal de notification.
- Rétention des données, journaux et métriques.
- Procédures de sauvegarde, restauration et retour arrière.

### Vérifications dépendant de l'infrastructure fournie

- Distribution Linux et présence de systemd.
- Possibilité d'installer ou d'utiliser Docker et Docker Compose.
- Accès nécessaires au déploiement Ansible.
- Ressources CPU, mémoire et stockage.
- Réseau, accès à l'édition et protection des services.
- Disponibilité de la machine et conditions d'exploitation.

Ces points seront suivis sans présenter les mesures prévues
comme déjà mises en œuvre ou testées.

## 9. Publier et archiver l'édition

Responsable : Walter.
Relecture : Darrel et Yannick.
Statut : format HTML statique validé par l'équipe ;
Nginx validé par Darrel pour la diffusion.
Les modalités d'accès et les droits de publication restent à préciser.

### Besoin

Rendre l'édition quotidienne consultable à 7 h et conserver
une archive datée, sans développer une interface élaborée.

Présenter les thèmes, les sources et les liens vers les publications.
Les extraits ne seront affichés que si les conditions de réutilisation
le permettent.

### Option A — Édition HTML statique

Principe :
Générer une page HTML avec Python à partir des données propres
et des groupes thématiques.

Avantages retenus pour le projet :
- Lecture dans un navigateur.
- Séparation entre génération et consultation.
- Pas d'application dynamique nécessaire pour présenter l'édition.
- Archivage sous forme de fichiers datés.

Limites :
- Serveur de diffusion et accès réseau à définir.
- Publication cohérente des fichiers à organiser.
- Traitement sécurisé des textes et liens à prévoir.
- Gestion du cache et vérification de l'édition accessible à tester.

Décision :
Format retenu et validé par l'équipe.

### Option B — Markdown

Principe :
Générer un document Markdown pour chaque édition.

Avantages :
- Format simple à produire et à relire.
- Adapté à la documentation technique.

Limites pour notre projet :
- Consultation dépendante de l'outil de rendu.
- Point d'accès quotidien moins direct pour le lecteur.
- Diffusion à organiser malgré tout.

Décision :
Non retenu comme format principal de consultation.

### Option C — PDF

Principe :
Générer un document PDF pour chaque édition.

Avantages :
- Document autonome.
- Présentation conservée lors du partage.

Limites pour notre projet :
- Génération et contrôles supplémentaires.
- Accès à la dernière édition à organiser.
- Aucun besoin initial imposant ce format.

Décision :
Non retenu au démarrage.

### Option D — Application web dynamique

Principe :
Calculer les pages à la demande avec une application web.

Avantages recherchés :
- Possibilité de recherche et d'interactions supplémentaires.

Limites :
- Fonctionnalités non nécessaires au périmètre initial.
- Application et dépendances supplémentaires à exploiter.
- Effort pouvant détourner l'équipe de la robustesse du pipeline.

Décision :
Non retenue pour la première architecture.

### Format retenu

Une édition HTML statique sera générée par Python.

Elle présentera :
- la date de l'édition ;
- l'heure réelle de publication ;
- les groupes thématiques et leurs libellés ;
- les titres, les journaux, les dates et les liens ;
- les extraits uniquement si leur diffusion est autorisée ;
- les éventuelles limites de couverture ;
- un accès aux archives datées.

Aucune interface élaborée, aucun commentaire et aucun suivi
des lecteurs ne seront ajoutés.

### Organisation envisagée

- Une page d'accueil pour la dernière édition publiée.
- Un répertoire d'archive par date.
- Des fichiers persistants, distincts des données internes.
- Un accès en lecture seule aux fichiers pour le serveur de diffusion.

Nginx est retenu et validé par Darrel pour servir les fichiers HTML.

Il sera déployé comme service distinct dans Docker Compose,
avec un accès en lecture seule au répertoire de publication.
Sa configuration sera versionnée et déployée avec Ansible.

Les fichiers JSONL, la base SQLite, les journaux internes et les secrets
ne seront pas exposés par le serveur web.

L'adresse de consultation, les modalités HTTPS et une éventuelle
restriction d'accès dépendront de l'infrastructure fournie
et des droits de réutilisation.

### Publication et contrôles prévus

1. Générer une édition candidate hors de l'emplacement publié.
2. Vérifier la date de l'édition et les champs nécessaires.
3. Contrôler la fraîcheur de chaque article au moment de publication.
4. Écarter les dates futures, non exploitables ou âgées de 24 h ou plus.
5. Vérifier les liens et l'absence de contenu hors périmètre.
6. Publier uniquement une édition validée, sans exposer un fichier
   partiellement écrit.
7. Vérifier l'accès à l'édition effectivement attendue.
8. Enregistrer le résultat et l'heure réelle de publication.

Les textes et liens des sources seront traités comme des entrées
non fiables. L'échappement HTML et la validation des liens
seront prévus.

Le succès de la génération ne suffira pas à déclarer la publication
réussie.

### Échecs et archives

Une édition invalide ne remplacera pas la dernière édition publiée.

La date restera visible pour qu'une ancienne édition ne soit pas
présentée comme celle du jour. Une alerte signalera l'édition manquante.

Les archives seront clairement identifiées comme historiques.
La contrainte de fraîcheur s'applique au moment de la publication.

La durée de conservation et les droits d'archivage restent à définir.
Aucune diffusion publique ne sera engagée sans clarification
des conditions applicables.

### Tests prévus

- Génération à partir de données synthétiques.
- Affichage des accents et caractères spéciaux.
- Absence d'interprétation de HTML provenant des sources.
- Rejet des dates non admissibles.
- Échec sans remplacement de l'édition précédente.
- Accès à l'édition attendue et aux archives.
- Contrôle du cache et de la date de l'édition accessible.
- Absence d'exposition des fichiers JSONL, de SQLite et des secrets.

Aucun mécanisme de publication n'est déclaré réalisé ni testé.

### Sources

- Nginx — Guide de démarrage et contenu statique :
  https://nginx.org/en/docs/beginners_guide.html
- Nginx — Documentation :
  https://nginx.org/en/docs/