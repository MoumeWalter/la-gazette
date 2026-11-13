# Note de cadrage — La gazette

Statut : première architecture validée ; relecture finale et schéma
à terminer. Droits de réutilisation des sources, fuseau de publication
et compatibilité de l'hébergement à clarifier.
Aucun composant n'est déclaré déployé ou testé à ce stade.

## 1. Besoin

Publier chaque matin une édition présentant les principaux thèmes
de la presse quotidienne régionale et nationale, avec les sources,
les titres et les liens permettant de poursuivre la lecture
chez les éditeurs.

Le résultat attendu est un pipeline autonome, reproductible et observable :
l'équipe doit pouvoir démontrer qu'il a fonctionné sans intervention
et identifier les incidents.

## 2. Périmètre

Inclus :
- Collecte des titres, extraits, liens et dates autorisés d'au moins
  quatre PQR et quatre PQN.
- Conservation des métadonnées brutes autorisées et de leur provenance.
- Nettoyage, contrôle qualité et déduplication.
- Regroupement des publications par thèmes.
- Génération, publication et archivage d'une édition HTML.
- Supervision des traitements et de l'édition réellement accessible.

Exclus :
- Texte intégral des articles, images, commentaires, profils
  et données d'abonnés.
- Résumés d'articles générés avec un LLM.
- Interface élaborée et suivi des lecteurs.

## 3. Contraintes

- Une édition quotidienne disponible à 7 h.
- Articles strictement âgés de moins de 24 h lors de la publication.
- Respect des conditions des éditeurs et absence de collecte
  de données personnelles.
- Budget nul ; développement sur les machines de l'équipe,
  puis déploiement sur l'infrastructure fournie.
- Infrastructure et configurations décrites en code.
- Priorité à la robustesse, à l'observabilité et à la reproductibilité.

Le fuseau de référence reste à définir.
Les conditions de réutilisation et d'archivage doivent être clarifiées
avant exploitation ; l'accès technique à un flux ne vaut pas autorisation.

## 4. Stack retenue et justification

| Composant | Choix validé et justification |
|---|---|
| Langage | Python : langage commun aux étapes, cohérent avec JSONL, SQLite et les traitements scikit-learn retenus. |
| Collecte | Flux RSS des éditeurs : métadonnées structurées sans extraction des pages d'articles. Huit flux inspectés manuellement ; observations et restrictions dans sources.csv. |
| Planification | Timer systemd associé à une unité service : configuration versionnée et exécution quotidienne sur un serveur Linux compatible. |
| Stockage brut | JSONL : conservation des champs autorisés avant nettoyage, avec provenance, pour permettre le retraitement. Aucun XML intégral ni contenu hors périmètre n'est conservé. |
| Stockage propre | SQLite : requêtes SQL et contraintes d'unicité ; exécution sur un hôte unique avec écritures coordonnées. |
| Thèmes | TF-IDF et classification hiérarchique à distance cosinus, avec liaison moyenne : nombre de groupes variable selon les publications. |
| Supervision | Prometheus, Pushgateway et Grafana : métriques du batch, historique, tableaux de bord et alertes ; journaux structurés pour le diagnostic. |
| Infrastructure | Ansible configure l'hôte et déploie les fichiers ; Docker et Docker Compose décrivent et exécutent les composants, réseaux et volumes. |
| CI/CD | GitHub Actions contrôle les modifications et construit l'image ; déploiement avec Ansible explicitement déclenché après réussite des contrôles et relecture. |
| Livraison | HTML statique généré par Python et servi par Nginx : consultation simple sans application dynamique, avec archives datées. |

### Exécution et persistance

Le pipeline sera déclenché avant 7 h avec une marge fondée sur
sa durée mesurée. Persistent=true sera envisagé pour rattraper
une échéance manquée ; un rattrapage après 7 h restera un retard.

Le développement et les tests utiliseront WSL 2, avec code et données
d'exécution dans le système de fichiers Linux. WSL n'est pas la cible
de production disponible en continu. La procédure d'accès à Docker
sera commune et documentée.

La cible est un serveur Linux existant, dont le système, les ressources,
les accès et les privilèges restent à confirmer. Sa création n'est pas
prise en charge par cette architecture.

Les traitements s'exécuteront à la demande ; les services de supervision
et Nginx devront rester disponibles. Les données JSONL, SQLite,
les éditions et les données de supervision seront persistantes,
hors du dépôt Git et du cycle de vie des conteneurs.

### Regroupement et supervision

Le seuil de regroupement, le prétraitement et le traitement des textes
vides et publications isolées seront évalués sur un échantillon annoté.
Les entrées, paramètres, versions et résultats seront tracés.
Aucune performance n'est encore revendiquée.

La supervision contrôlera les étapes, les dernières tentatives
et réussites, la présence de l'édition attendue, son heure de publication
et la fraîcheur des articles. Un ancien succès ne devra pas masquer
une absence d'exécution.

Grafana Alerting sera l'unique moteur d'alerte initial.
Les journaux du service systemd seront consultables avec journalctl.

### Livraison et déploiement

Nginx, validé par Darrel, sera un service distinct dans Docker Compose.
Il accédera au répertoire de publication en lecture seule.
Sa configuration sera versionnée et déployée avec Ansible.
Les données internes et les secrets ne seront pas exposés.

L'édition affichera sa date, son heure réelle de publication, les thèmes,
les sources, les titres et les liens. Les extraits ne seront diffusés
que si les droits le permettent.

Une édition candidate sera contrôlée avant publication.
Une édition invalide ne remplacera pas la précédente.
L'accès à l'édition effectivement attendue sera vérifié :
la génération d'un fichier ne suffit pas à prouver la publication.

Les archives seront datées et identifiées comme historiques ;
la fraîcheur de moins de 24 h s'applique lors de leur publication initiale.

Chaque version applicative sera reliée à un commit et à une référence
d'image traçable. Aucun déploiement automatique sur chaque push
n'est retenu. Systemd, et non GitHub Actions, déclenchera le batch quotidien.

### Points restant à préciser

Décisions de cadrage :
- Fuseau de référence de l'édition.
- Droits de réutilisation, de diffusion et d'archivage des sources.
- Adresse de consultation, modalités HTTPS et restriction d'accès
  éventuelle selon l'infrastructure fournie.

Paramètres de réalisation :
- Version de Python, bibliothèques et outillage de test.
- Workflows, registre ou transfert des images, quotas et accès au serveur.
- Horaires, délais, règles de qualité, seuil de regroupement et alertes.
- Rétentions, canal de notification, sauvegardes, restauration
  et retour arrière.

## 5. Alternatives écartées et justification

| Domaine | Alternatives et raisons |
|---|---|
| Langage | Node.js : nécessiterait de reconsidérer le regroupement ou d'ajouter un composant Python. Shell seul : traitements métier et tests dispersés entre commandes. |
| Collecte | API d'éditeurs ou d'agrégateurs : couverture, accès, fraîcheur et usage à budget nul non confirmés. Extraction HTML : non retenue dans notre périmètre. |
| Planification | Cron : moins adapté au rattrapage prévu. Airflow : complexité initiale non justifiée. GitHub Actions planifié : non retenu pour l'échéance de 7 h, mais retenu pour la CI. |
| Stockage | JSONL seul : moins adapté aux requêtes et contraintes des données propres. PostgreSQL : service supplémentaire non nécessaire à l'architecture initiale. Parquet : aucun besoin initial d'export analytique le justifiant. |
| Thèmes | Mots-clés : entretien manuel et couverture limitée. K-means : nombre de groupes à choisir. Modèles sémantiques : gain, licence et ressources à évaluer avant adoption. |
| Supervision | Journaux seuls : pas de supervision complète. Textfile collector : non privilégié pour le batch de service. Offre hébergée : fournisseur et conditions non validés. |
| Infrastructure | Scripts seuls : gestion de l'état à développer. Terraform : aucune création de ressources confirmée. Kubernetes : aucun cluster ni besoin multi-nœud imposé. |
| CI/CD | GitLab CI/CD : seconde plateforme ou migration. Jenkins : service supplémentaire à exploiter. Contrôles manuels seuls : vérifications non systématiques. |
| Livraison | Markdown : consultation moins directe. PDF : génération supplémentaire sans besoin initial. Application dynamique : complexité et fonctionnalités hors priorité. |

Les règles et mots-clés ainsi que K-means restent des références
possibles pour l'évaluation. Des scripts système ciblés restent autorisés.

## 6. Risques et mesures envisagées

| Risque | Mesure prévue | Référent |
|---|---|---|
| Droits incompatibles avec l'usage collectif | Clarifier les autorisations ; remplacer les sources incompatibles ; tracer les décisions dans sources.csv. | Walter |
| Source indisponible ou champs absents | Délais et reprises bornés ; contrôle des champs ; signalement des échecs. | Walter |
| Dates incorrectes, articles anciens ou doublons | Contrôler les fuseaux ; rejeter dates futures, non exploitables et articles de 24 h ou plus ; définir la déduplication. | Walter |
| Contenu collecté hors périmètre | Liste explicite des champs conservés ; exclusion du texte intégral, des images, commentaires et profils. | Walter |
| Édition absente, en retard ou partiellement publiée | Marge avant 7 h, validation avant publication, contrôle de la page accessible et alerte ; ancienne édition clairement datée. | Darrel / Yannick |
| Perte de données ou environnement non reproductible | Persistance, configuration en code, sauvegardes et tests de restauration. | Darrel |
| Serveur incompatible ou ressources insuffisantes | Confirmer système, accès, privilèges et ressources ; documenter les adaptations. | Darrel |
| Thèmes peu pertinents ou résultats non reproductibles | Échantillon annoté, évaluation du seuil, traitement des cas limites et traçabilité. | Yannick |
| Métriques anciennes donnant un faux succès | Vérifier les dates de réussite et de publication ; tester données absentes et périmées. | Yannick |
| Pipeline et supervision arrêtés ensemble | Documenter le risque de l'hôte unique ; étudier un contrôle externe si possible. | Darrel / Yannick |
| Version non identifiable ou retour arrière incomplet | Référence de commit et d'image ; conservation de la version précédente ; prise en compte des changements de données et de schéma. | Darrel |
| Secrets exposés ou dépassement du budget | Permissions limitées, secrets hors Git, services protégés et contrôle des quotas. | Darrel |
| Dépendance à une personne | Documentation, relecture croisée et validation collective. | Toute l'équipe |

Ces mesures sont prévues, pas encore implémentées ni testées.

## 7. Équipe et répartition des rôles

- Walter — Données et coordination :
  sources, collecte, nettoyage, qualité, stockage et consolidation
  des livrables.
- Darrel — Infrastructure et automatisation :
  conteneurisation, CI/CD, planification, déploiement et infrastructure
  en code.
- Yannick — MLOps et observabilité :
  regroupement, évaluation, versionnement, métriques, journaux et alertes.

Chaque membre documente et teste sa partie.
Les contributions sont relues par un autre membre et les décisions
techniques sont validées collectivement.

Documents associés :
- docs/veille.md : comparaisons détaillées et références.
- docs/architecture.md : six blocs, données échangées et lieux d'exécution.
- sources.csv : sources, accès, conditions et vérifications.