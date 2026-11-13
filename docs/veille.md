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
Solution retenue en priorité, sous réserve de la vérification
des huit sources et de leurs conditions.

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

Nous privilégions les flux RSS/Atom directement publiés par les éditeurs.
Ils constituent la première piste à vérifier pour récupérer les
métadonnées nécessaires sans extraire les pages des articles.

Ce choix sera validé source par source dans sources.csv :
adresse exacte, format, champs disponibles, conditions d'utilisation,
fréquence observée et vérification manuelle d'un élément.

L'accessibilité technique d'un flux ne vaut pas autorisation
de réutilisation de son contenu.

### Sources de la veille

- Spécification RSS :
  https://cyber.harvard.edu/rss/rss.html
- Spécification Atom :
  https://datatracker.ietf.org/doc/html/rfc4287
- Exemple d'API d'agrégation — GDELT DOC :
  https://blog.gdeltproject.org/gdelt-doc-2-0-api-debuts/
- Exemple de conditions commerciales d'une API — NewsAPI :
  https://newsapi.org/pricing

La disponibilité d'API officielles propres aux journaux et les conditions
de chaque flux restent à vérifier ; aucune vérification n'est déclarée
comme réalisée à ce stade.

## 2. Executer le pipeline chaque matin
A completer.

## 3. Stocker les donnees brutes et propres
A completer.

## 4. Regrouper les textes par themes
A completer.

## 5. Surveiller le pipeline
A completer.

## 6. Decrire l'infrastructure en code
A completer.
