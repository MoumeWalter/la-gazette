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

A completer apres la veille technique.

## 5. Alternatives ecartees et justification

A completer.

## 6. Risques et mesures envisagees

A completer.

## 7. Equipe et repartition des roles

A completer.
