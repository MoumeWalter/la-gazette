# Note de cadrage — La gazette

Statut : choix principaux de la première architecture validés ;
cadrage à finaliser et à relire collectivement.

Choix validés :
- Python pour le pipeline ;
- flux RSS des éditeurs pour la collecte ;
- timer systemd pour la planification quotidienne ;
- JSONL et SQLite pour le stockage ;
- TF-IDF et classification hiérarchique pour les thèmes ;
- Prometheus, Pushgateway et Grafana pour la supervision ;
- Ansible, Docker et Docker Compose pour le déploiement ;
- GitHub Actions pour la CI et la préparation des versions,
  avec déploiement explicitement déclenché après validation.

Les droits de réutilisation des sources et la compatibilité
de l'infrastructure fournie restent à clarifier.
Les autres choix ouverts sont indiqués en section 4.

## 1. Besoin

La gazette doit permettre de repérer chaque matin les principaux thèmes
abordés par la presse quotidienne régionale et nationale, sans avoir
à consulter séparément chaque journal.

Elle publie automatiquement une édition quotidienne regroupant les
publications par thèmes, avec les sources associées et les liens
permettant de poursuivre la lecture sur les sites des éditeurs.

L'objectif est de construire un pipeline autonome, reproductible et
observable : l'équipe doit pouvoir vérifier qu'il a fonctionné et
identifier les éventuels incidents.

## 2. Périmètre

### Inclus

- Collecter les titres, extraits, liens et dates accessibles aux machines
  dans le respect des conditions des éditeurs.
- Couvrir au minimum quatre sources de presse quotidienne régionale
  et quatre sources de presse quotidienne nationale.
- Conserver les métadonnées brutes autorisées et leur provenance.
- Nettoyer les données, contrôler leur qualité et éliminer les doublons.
- Regrouper les publications par thèmes.
- Générer, publier et archiver une édition quotidienne.
- Superviser le pipeline avec des métriques, des journaux et des alertes.

### Exclus

- Collecter ou stocker le texte intégral des articles.
- Collecter les images, commentaires, profils ou données d'abonnés.
- Résumer les articles avec un LLM.
- Développer une interface graphique élaborée.

## 3. Contraintes

- Publier une édition chaque jour à 7 h.
- Ne retenir dans l'édition que des articles de moins de 24 h.
- Respecter et documenter les conditions d'utilisation des éditeurs.
- Ne collecter aucune donnée personnelle.
- Utiliser un budget nul et les machines de l'équipe dans un premier temps.
- Préparer le passage vers l'infrastructure de production fournie plus tard.
- Décrire l'infrastructure en code et versionner les fichiers du projet.
- Privilégier la robustesse et l'observabilité plutôt que l'ajout
  de fonctionnalités.

Le fuseau de référence de l'édition reste à définir explicitement.
L'âge des articles sera contrôlé au moment de la publication.

## 4. Stack retenue et justification

### Langage — Python

Choix validé par l'équipe : utiliser Python comme langage principal
pour la collecte, le nettoyage, le stockage, le regroupement
et la génération de l'édition.

Ce choix assure la cohérence avec JSONL, SQLite et les traitements
TF-IDF et de classification hiérarchique prévus avec scikit-learn.
Un langage commun limite les interfaces entre les étapes.

Le développement et les tests utiliseront WSL 2 ; l'exécution cible
sera conteneurisée. La version de Python, les bibliothèques,
les dépendances et l'outillage de test restent à préciser.

Aucun résultat de performance n'est revendiqué.

### Collecte — flux RSS des éditeurs

Retenir les flux RSS pour récupérer les titres, extraits, liens et dates
sans extraire les pages des articles.

Les huit sources ont été inspectées manuellement ; les observations
et restrictions sont consignées dans sources.csv.
La validation technique ne vaut pas autorisation de réutilisation.

Les conditions encore ouvertes devront être clarifiées avant exploitation.
Une source incompatible devra être remplacée ou faire l'objet
d'une autorisation adaptée.

### Ordonnancement — timer systemd

Choix validé par Darrel : retenir un timer systemd pour la production,
sous réserve que l'infrastructure fournie soit un serveur Linux utilisant
systemd et permette le déploiement des unités nécessaires.

Ce choix convient à l'architecture envisagée sur un serveur unique :
la planification est décrite dans une unité .timer et l'exécution dans
une unité .service. Ces fichiers seront versionnés dans le dépôt.

L'équipe utilise des PC Windows et prévoit WSL 2 pour développer et tester
les unités dans un environnement Linux compatible.
WSL reste un environnement de développement et de test, pas une garantie
de disponibilité pour l'édition quotidienne.

Le traitement sera déclenché avant 7 h avec une marge fondée sur
sa durée mesurée. Le fuseau de référence, la gestion des échecs,
les alertes et les procédures de relance seront explicitement définis.

Persistent=true sera envisagé pour rattraper une échéance manquée.
Un rattrapage après 7 h restera comptabilisé comme un retard de publication.

La compatibilité de l'infrastructure de production reste à confirmer.

### Stockage — JSONL et SQLite

Décision de l'équipe : conserver les métadonnées brutes autorisées
dans des fichiers JSONL et les données nettoyées dans une base SQLite.

JSONL permet de conserver les valeurs avant nettoyage et leur provenance,
puis de rejouer les traitements sans effectuer une nouvelle collecte.
Le stockage brut exclut le XML intégral, les images et le texte intégral
des articles.

SQLite est retenu pour interroger les articles propres, appliquer les
contraintes d'unicité définies et gérer les thèmes et les éditions.

L'architecture prévoit une exécution sur une machine unique et des
écritures coordonnées, en tenant compte de la limite d'un écrivain
simultané par base.

En développement, le code et les données d'exécution seront placés dans
le système de fichiers Linux de WSL 2. Chaque membre utilisera sa propre
base de test ; les données collectées et les bases ne seront pas
versionnées dans Git.

En production, les données seront stockées de manière persistante,
hors du cycle de vie des conteneurs. Les chemins seront configurables.
Une procédure de sauvegarde et un test de restauration seront prévus.

Ce choix est définitif pour la première architecture. Il devra être
réexaminé si le projet exige ultérieurement plusieurs serveurs écrivains.

### Regroupement par thèmes — TF-IDF et classification hiérarchique

Choix validé par Yannick : représenter les titres et extraits autorisés
avec TF-IDF, puis regrouper les publications par classification
hiérarchique avec distance cosinus et liaison moyenne.

Cette approche est retenue pour permettre un nombre de groupes variable
selon les publications du jour, sans imposer un nombre fixe de thèmes.

Les termes caractéristiques serviront à proposer des libellés,
sans générer de résumés d'articles avec un LLM.

Le seuil de distance, le prétraitement, les textes sans représentation
exploitable et le traitement des publications isolées seront évalués
sur un échantillon annoté par l'équipe.

Les paramètres, les versions et les résultats des expériences seront
tracés. Les performances restent à mesurer.

### Supervision — Prometheus, Pushgateway et Grafana

Choix validé par Yannick : utiliser Prometheus pour les métriques,
Pushgateway pour les résultats des traitements batch et Grafana
pour les tableaux de bord et les alertes.

Les journaux applicatifs structurés complètent les métriques
pour le diagnostic ; les journaux du service systemd seront
consultables avec journalctl.

La supervision contrôlera les résultats des étapes, les dates
des dernières exécutions, la présence de l'édition attendue,
son heure de publication et la fraîcheur des articles.

Un ancien succès ne devra pas masquer une absence d'exécution.

Grafana Alerting est retenu comme unique moteur d'alerte initial.
Le canal de notification, les seuils, les rétentions et le traitement
des métriques absentes restent à définir et à tester.

Le risque d'indisponibilité simultanée du pipeline et de sa supervision
sur une même machine est identifié. Aucun dispositif externe
de surveillance n'est encore retenu.

### Infrastructure et conteneurisation — Ansible, Docker et Compose

Choix validé par Darrel : utiliser Ansible pour configurer le serveur
Linux et déployer les fichiers, Docker pour conteneuriser les composants
et Docker Compose pour décrire les services, réseaux et volumes.

Cette séparation distingue la configuration de l'hôte, l'exécution
des composants et la planification quotidienne par systemd.

Les fichiers Compose, configurations de supervision et unités systemd
seront versionnés et déployés depuis le dépôt.

Les données JSONL, la base SQLite et les données de supervision
seront conservées dans un stockage persistant, hors du dépôt Git.

Le traitement sera exécuté à la demande, tandis que les composants
de supervision devront rester disponibles.
SQLite restera un fichier de base persistant, sans serveur de base
de données distinct.

En développement, l'équipe utilisera WSL 2 avec une procédure Docker
commune à documenter. Les tests locaux seront complétés par des tests
sur le serveur de production.

La cible envisagée est un serveur existant : sa création n'est pas
prise en charge par cette architecture. Son système, ses ressources,
ses accès et les privilèges disponibles restent à confirmer.

Les secrets ne seront pas stockés dans le dépôt.
Leur mode de fourniture et les permissions nécessaires seront définis
avant le déploiement.

Le déploiement, sa réexécution, la persistance et la restauration
seront testés lors des TD suivants.

### CI/CD — GitHub Actions et déploiement contrôlé

Choix validé par Darrel : utiliser GitHub Actions pour les contrôles
de qualité, les tests automatisés, la validation des configurations
et la construction de l'image Docker.

Ce choix conserve les contrôles avec le dépôt GitHub existant,
sans ajouter une plateforme CI distincte à exploiter.

Les contrôles seront déclenchés sur les push et pull requests
selon les règles définies dans les workflows.

Le déploiement de production sera déclenché explicitement après
réussite des contrôles et relecture. Ansible déploiera la configuration
et la version choisie.

Chaque version sera identifiée par son commit et une référence
d'image traçable. Aucun déploiement automatique sur chaque push
n'est retenu pour la première architecture.

Systemd conserve la planification quotidienne.
GitHub Actions n'est pas utilisé comme ordonnanceur de l'édition de 7 h.

Le registre ou le transfert des images, les quotas, les accès au serveur
et la procédure de retour arrière restent à préciser et à tester.

### Points restant à préciser

Décisions de cadrage :
- Format et mode de publication de l'édition.
- Fuseau de référence de l'édition.

Paramètres de réalisation :
- Version de Python, bibliothèques et outillage de test.
- Organisation des modules et commande d'exécution.
- Workflows de CI et règles de déclenchement.
- Registre ou mécanisme de transfert des images.
- Quotas, accès au serveur et déclenchement du déploiement.
- Horaires, délais, seuils, rétentions et canal de notification.
- Sauvegardes, restauration et retour arrière.

Ces points restent ouverts ; ils ne sont pas présentés comme validés.

## 5. Alternatives écartées et justification

### Langage

- JavaScript/TypeScript avec Node.js : non retenu comme langage principal
  pour éviter de reconsidérer l'implémentation du regroupement
  ou d'ajouter un composant Python séparé.
- Scripts shell comme langage principal : non retenus pour centraliser
  les traitements métier et leurs tests dans le projet Python ;
  des scripts système ciblés restent possibles.

### Collecte

- API officielles des éditeurs : accès adapté aux huit sources
  et au budget nul non confirmé.
- API d'agrégation : couverture, fraîcheur et conditions de production
  non établies pour le projet.
- Extraction des pages HTML : non retenue dans notre périmètre ;
  la collecte privilégie les flux des éditeurs.

### Ordonnancement

- Cron : non retenu au profit de la gestion par unités systemd
  et du rattrapage possible des échéances manquées.
- Apache Airflow : complexité initiale non justifiée pour l'exécution
  séquentielle envisagée sur un serveur unique.
- GitHub Actions planifié : non retenu comme ordonnanceur
  de l'édition de 7 h ; GitHub Actions est en revanche retenu
  pour la CI et la préparation des versions.

### Stockage

- Tout stocker en JSONL : non retenu pour les données propres,
  afin de disposer de requêtes SQL et de contraintes d'unicité en base.
- PostgreSQL : non retenu pour cette première architecture, qui prévoit
  un serveur unique et des écritures coordonnées ; l'équipe privilégie
  une exploitation initiale plus simple.
- Parquet : non retenu comme stockage opérationnel principal ; aucun
  besoin d'export analytique en colonnes ne le justifie à ce stade.

### Regroupement par thèmes

- Règles et mots-clés : non retenus comme méthode principale en raison
  de leur entretien manuel et de leur couverture limitée des nouveaux
  sujets ; conservés comme référence simple.
- TF-IDF + K-means : non retenu comme méthode principale, car il impose
  de choisir un nombre de groupes ; conservé comme alternative
  expérimentale.
- Représentations sémantiques préentraînées : non retenues pour
  la première architecture ; leur gain de pertinence, leur licence
  et leurs besoins en ressources devront être évalués avant adoption.

### Supervision

- Journaux systemd et script de contrôle seuls : non retenus comme
  dispositif complet ; les journaux restent utilisés pour le diagnostic.
- Prometheus avec Node Exporter textfile collector : non privilégié
  pour les résultats du service batch ; Pushgateway est retenu.
- Service de supervision hébergé : non retenu comme socle principal,
  faute de fournisseur et de conditions validés ; un contrôle externe
  pourra être étudié pour couvrir l'indisponibilité totale du serveur.

### Infrastructure et déploiement

- Scripts shell comme mécanisme principal : non retenus pour
  la configuration de l'hôte ; Ansible est privilégié pour décrire
  et rejouer cette configuration. Des scripts ciblés restent possibles.
- Terraform : non retenu à ce stade, car aucun besoin ni droit
  de création de ressources n'est confirmé ; le serveur sera fourni.
- Kubernetes : non retenu pour la première architecture, faute
  de cluster imposé ou de besoin multi-nœud confirmé.

### CI/CD

- GitLab CI/CD : non retenu pour éviter une migration ou l'ajout
  d'une seconde plateforme alors que le dépôt est sur GitHub.
- Jenkins : non retenu pour éviter un service supplémentaire
  à héberger, sécuriser et maintenir.
- Contrôles manuels seuls : insuffisants pour assurer une vérification
  systématique et traçable ; conservés en complément de la CI.

Les alternatives relatives à la publication seront ajoutées
après comparaison et validation par l'équipe.

## 6. Risques et mesures envisagées

| Risque | Mesure envisagée | Référent |
|---|---|---|
| Conditions des éditeurs incompatibles avec notre usage collectif | Clarifier les autorisations avant exploitation ; remplacer une source si nécessaire ; documenter la décision dans sources.csv. | Walter |
| Source indisponible ou champs manquants | Prévoir des délais d'attente, des tentatives limitées et un contrôle des champs ; signaler les sources en échec. | Walter |
| Articles trop anciens, dates incorrectes ou doublons | Contrôler chaque date avec son fuseau horaire ; exclure de l'édition les articles de 24 h ou plus ou non datables ; définir une règle de déduplication. | Walter |
| Collecte accidentelle de contenus hors périmètre | Limiter les champs conservés aux titres, extraits, liens, dates et métadonnées techniques nécessaires ; exclure texte intégral, images, commentaires et profils. | Walter |
| Édition absente à 7 h ou pipeline interrompu | Prévoir une marge avant publication, une procédure de relance et une alerte ; mesurer le respect de l'horaire. | Darrel / Yannick |
| Perte des données ou environnement non reproductible | Prévoir un stockage persistant, des sauvegardes avec test de restauration et une infrastructure décrite en code ; ne pas versionner les données collectées ni les secrets dans Git. | Darrel |
| Déploiement incompatible avec le serveur fourni | Confirmer le système, les ressources, les accès et les privilèges avant déploiement ; documenter toute adaptation de l'architecture. | Darrel |
| Regroupements peu pertinents ou difficiles à reproduire | Évaluer un échantillon annoté ; régler le seuil ; définir le traitement des textes vides et des publications isolées ; tracer les paramètres et versions. | Yannick |
| Anciennes métriques interprétées comme une réussite du jour | Contrôler les dates de dernière exécution et de publication ; tester les métriques absentes et les résultats périmés. | Yannick |
| Pipeline et supervision indisponibles simultanément | Documenter ce risque lié à l'hébergement commun ; étudier un contrôle externe si les conditions le permettent. | Darrel / Yannick |
| Version déployée non identifiable ou retour arrière incomplet | Relier le déploiement au commit et à l'image ; conserver la référence précédente ; prévoir le traitement des changements de données et de schéma. | Darrel |
| Secrets exposés ou dépassement du budget CI | Limiter les permissions, protéger les secrets, vérifier les quotas et définir la rétention des artefacts. | Darrel |
| Dépendance à un seul membre ou contributions incohérentes | Documenter chaque partie, faire relire les modifications par un autre membre et valider collectivement les décisions techniques. | Toute l'équipe |

Ces mesures sont prévues au cadrage ; elles ne sont pas encore
implémentées ni testées. Elles seront précisées lors des TD suivants.

## 7. Équipe et répartition des rôles

- Walter — Référent données et coordination :
  sélection et documentation des sources, collecte, nettoyage,
  qualité et stockage des données ; consolidation des livrables.

- Darrel — Référent infrastructure et automatisation :
  conteneurisation, CI/CD, ordonnancement et infrastructure en code.

- Yannick — Référent MLOps et observabilité :
  regroupement par thèmes, évaluation et versionnement du modèle,
  métriques, journaux et alertes.

Chaque membre documente et teste sa partie.
Les choix techniques sont validés collectivement et les contributions
sont relues par un autre membre.

Les comparaisons détaillées et leurs références figurent
dans docs/veille.md. Le schéma est maintenu dans docs/architecture.md.