# Churn Revolut — projet final ML for Business

Binôme : Julien Malherbe & Héloïse [nom]

## Installation
1. Cloner le dépôt
2. Copier dans ce dossier les 3 fichiers de données (non versionnés, trop lourds) :
   `rev-users.csv`, `rev-transactions.csv`, `rev-notifications.csv`
3. `pip install numpy pandas matplotlib scikit-learn jupyter`
4. Lancer Jupyter depuis ce dossier

## Organisation
| Fichier | Qui | Contenu |
|---|---|---|
| `churn_revolut.ipynb` | les deux | **notebook final** à rendre — on n'y colle que du code validé |
| `brouillon_julien.ipynb` | Julien | modèles, seuil, euros, recommandation |
| `brouillon_heloise.ipynb` | Héloïse | audit, `build_dataset(cutoff)`, features, anti-fuite |

Chacun travaille dans **son** brouillon → aucun conflit Git (les notebooks se fusionnent mal).
Quand une partie est prête : on la recopie dans le notebook final, à deux, et on la relit ensemble.

## Routine
```
git pull                     # avant de commencer
git add <fichier>            # après avoir travaillé
git commit -m "ce que j'ai fait"
git push
```
Avant de commiter un notebook : **Restart & Run All**, pour qu'il tourne de haut en bas.

## Contrat entre les deux brouillons
`build_dataset(cutoff)` renvoie un DataFrame : une ligne par client actif à la coupure,
les features (calculées **avant** la coupure) et une colonne `churn` (0/1, calculée **après**).
