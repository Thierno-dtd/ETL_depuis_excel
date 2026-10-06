# TP ETL - Ingénierie des données 

TP réalisé par **Thierno-dtd** dans le cadre du cours d'ingénierie des données.

On part d'un fichier Excel de clients rempli « à la main », avec beaucoup d'erreurs, et on le nettoie étape par étape avec Pandas. À la fin, tout le traitement est regroupé dans un petit pipeline ETL (extraction, transformation, chargement) qui sort des fichiers propres prêts à être utilisés.

## Contenu du dépôt

```
.
├── DonneesTP_ETL.xlsx               # données brutes fournies pour le TP
├── ETL_Ingenierie_de_donnee.ipynb   # le notebook avec tout le travail
└── README.md
```

Le dossier `sortie/` n'est pas dans le dépôt : il est créé automatiquement quand on exécute le notebook.

## Les données

Le fichier Excel contient deux feuilles :

- `Travail demandé` : la liste des 10 parties à traiter ;
- `Donnees_Brutes` : 362 clients décrits par 9 colonnes (âge, revenu mensuel, score de service, ville, sexe, statut professionnel, client actif, date d'inscription, commentaire).

Les problèmes qu'on a trouvés en explorant le fichier :

- des lignes en double ;
- des valeurs manquantes dans presque toutes les colonnes ;
- des valeurs impossibles (âge de -5 ou 130 ans, score de 200 sur 100, revenu négatif, revenus de 5 à 12 millions FCFA) ;
- des villes écrites de plusieurs façons (`LIBREVILLE`, ` Oyem `, `port-gentil`...), ce qui donne 18 villes au lieu de 6 ;
- la même chose pour le sexe (`femme`, `FEMME `...) ;
- la colonne `Client_Actif` qui mélange `True`/`False` et `Oui`/`Non`.

## Ce qui est fait dans le notebook

Le notebook suit les 10 parties demandées :

1. Importation et exploration du fichier
2. Suppression des doublons
3. Traitement des valeurs manquantes (médiane pour les nombres, mode pour le texte)
4. Détection et correction des valeurs aberrantes (méthode IQR + règles métier)
5. Nettoyage et harmonisation du texte et des types
6. Encodage des variables catégorielles (binaire, ordinal, one-hot)
7. Transformation des dates (année, mois, trimestre, jour de la semaine)
8. Création de nouvelles variables (ancienneté, tranches d'âge et de revenu, niveau de score, client satisfait)
9. Visualisations avec Matplotlib et interprétation des graphiques
10. Pipeline ETL complet avec les fonctions `extraire()`, `transformer()` et `charger()`, puis export et contrôles

Chaque choix est expliqué dans le notebook (pourquoi la médiane plutôt que la moyenne, pourquoi on ne garde pas toutes les valeurs signalées par l'IQR, etc.).

## Résultat

| | Avant | Après |
|---|---|---|
| Lignes | 362 | 350 |
| Doublons | 12 | 0 |
| Valeurs manquantes | 50 | 0 |
| Valeurs aberrantes | 13 | 0 |
| Villes différentes | 18 | 6 |
| Colonnes | 9 | 21 (35 dans la version encodée) |

Le pipeline génère dans `sortie/` :

- `clients_propres.csv` : les données nettoyées, lisibles ;
- `clients_encodes.csv` : la version encodée, utilisable pour du machine learning ;
- `clients_ETL.xlsx` : les deux versions plus une feuille `Rapport_Qualite` qui résume les corrections ;
- `clients.db` : une base SQLite avec les tables `clients` et `clients_encodes`.

Les CSV utilisent le `;` comme séparateur pour s'ouvrir correctement dans Excel en français.

## Lancer le projet

Il faut Python 3 avec Jupyter. Avec Anaconda tout est déjà installé, sinon :

```bash
pip install pandas numpy matplotlib openpyxl jupyter
```

Ensuite, depuis le dossier du projet :

```bash
jupyter notebook ETL_Ingenierie_de_donnee.ipynb
```

et faire *Kernel → Restart & Run All*. Le fichier Excel doit rester dans le même dossier que le notebook.

La date de référence pour le calcul de l'ancienneté est fixée au 06/10/2026 dans le notebook. On l'a fixée volontairement pour que les résultats ne changent pas d'une exécution à l'autre.

## Quelques remarques sur les données

En analysant les données propres, on a relevé des points qui ne se corrigent pas avec du code et qui seraient à vérifier avec ceux qui ont produit le fichier :

- le score de service ne correspond pas aux commentaires (les clients qui ont écrit « À améliorer » ont en moyenne le meilleur score) ;
- seulement 34 % des clients de Libreville sont actifs, le taux le plus bas des six villes ;
- les inscriptions s'arrêtent presque après mars 2026 ;
- il n'y a pas d'identifiant client, donc on ne peut pas être sûr à 100 % que les doublons supprimés sont de vrais doublons.
