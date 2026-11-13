# Note de cadrage — La gazette

Statut : a completer et a valider en equipe.

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
- Conserver les données brutes collectées et leur provenance.
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

## 4. Stack retenue et justification

### Ordonnancement — timer systemd

Choix validé par Darrel : retenir un timer systemd pour la production,
sous réserve que l'infrastructure fournie soit un serveur Linux utilisant
systemd et permette le déploiement des unités nécessaires.

Ce choix convient à un pipeline quotidien exécuté sur un serveur unique :
la planification est décrite dans une unité .timer et l'exécution dans
une unité .service. Ces fichiers seront versionnés dans le dépôt.

L'équipe utilise des PC Windows avec WSL 2 pour développer et tester
les unités dans un environnement Linux compatible. WSL reste un
environnement de développement et de test, pas une garantie de
disponibilité pour l'édition quotidienne.

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

## 5. Alternatives ecartees et justification


- Tout stocker en fichiers JSONL : non retenu pour les données propres,
  afin de disposer de requêtes SQL et de contraintes d'unicité en base.
- PostgreSQL : non retenu pour cette première architecture, qui prévoit
  un serveur unique et des écritures coordonnées ; l'équipe privilégie
  une exploitation initiale plus simple.
- Parquet : non retenu comme stockage opérationnel principal ; aucun
  besoin d'export analytique en colonnes ne le justifie à ce stade.

## 6. Risques et mesures envisagées

| Risque | Mesure envisagée | Référent |
|---|---|---|
| Conditions des éditeurs incompatibles avec notre usage collectif | Clarifier les autorisations avant exploitation ; remplacer une source si nécessaire ; documenter la décision dans sources.csv. | Walter |
| Source indisponible ou champs manquants | Prévoir des délais d'attente, des tentatives limitées et un contrôle des champs ; signaler les sources en échec. | Walter |
| Articles trop anciens, dates incorrectes ou doublons | Contrôler chaque date avec son fuseau horaire ; exclure de l'édition les articles de plus de 24 h ou non datables ; définir une règle de déduplication. | Walter |
| Collecte accidentelle de contenus hors périmètre | Limiter les champs conservés aux titres, extraits, liens, dates et métadonnées techniques nécessaires ; exclure texte intégral, images, commentaires et profils. | Walter |
| Édition absente à 7 h ou pipeline interrompu | Prévoir une marge avant publication, une procédure de relance et une alerte ; mesurer le respect de l'horaire. | Darrel / Yannick |
| Perte des données ou environnement non reproductible | Prévoir un stockage persistant, des sauvegardes avec test de restauration et une infrastructure décrite en code ; ne pas versionner les données collectées ni les secrets dans Git. | Darrel |
| Regroupements thématiques peu pertinents ou difficiles à reproduire | Évaluer un échantillon de résultats et tracer les paramètres, les versions et les exécutions du modèle. | Yannick |
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

Chaque membre documente et teste sa partie. Les choix techniques
sont validés collectivement et les contributions sont relues par
un autre membre.