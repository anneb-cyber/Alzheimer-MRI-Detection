# Détection précoce de la maladie d'Alzheimer par IRM structurelle

Projet réalisé dans le cadre de l'UE *Medical Imaging and 3D Modeling*.

## Objectif

Développer une chaîne d'analyse basée sur l'IRM structurelle pour détecter
précocement des signes anatomiques associés à la maladie d'Alzheimer, et
étudier le lien entre biomarqueurs d'imagerie et caractéristiques cliniques.

La question centrale du projet est la **détection précoce** : peut-on repérer,
sur un scan pris à un instant où le sujet est encore cliniquement sain, des
signes anatomiques annonciateurs d'un déclin cognitif à venir ?

## Dataset : OASIS-2 (Longitudinal MRI Data)

- **Source** : [OASIS Brains](https://sites.wustl.edu/oasisbrains/) (Washington University, accès via NITRC-IR)
- **150 sujets**, 60-96 ans, chacun scanné sur 2 visites ou plus (373 sessions au total)
- **Groupes cliniques** : Nondemented (72), Demented dès la première visite (64), **Converted** (14 — sains à la première visite, diagnostiqués déments à une visite ultérieure)
- Format brut : 3-4 acquisitions T1 (`mpr-1/2/3(/4).nifti.img`) par session, **non prétraitées** (pas de normalisation ni de segmentation fournie, contrairement à OASIS-1)

Ce dataset a été choisi après avoir écarté **OASIS-1** (transversal, une seule
IRM par sujet) qui ne permet pas d'étudier une conversion dans le temps, donc
pas de véritable détection précoce.

## Pipeline du projet

| Notebook | Rôle |
|---|---|
| `exploration.ipynb` | Inventaire des sessions, arborescence des données, chargement du fichier clinique (`Group`, `Visit`, CDR, etc.) |
| `preprocessing.ipynb` | Moyenne des acquisitions répétées, skull stripping (seuillage d'Otsu + plus grande composante connexe), segmentation tissulaire (LCR / matière grise / matière blanche) par mélange gaussien (GMM), calcul des volumes 3D |
| `biomarkers_2D.ipynb` | Extraction de biomarqueurs sur la coupe axiale centrale : aires tissulaires 2D, asymétrie hémisphérique gauche/droite, features radiomiques (intensité, texture GLCM) |
| `classification_CNN.ipynb` | Détection précoce : à partir du **premier scan uniquement** de chaque sujet, classification binaire Nondemented vs Converted par CNN (PyTorch), validation croisée stratifiée à 5 plis (échantillon très déséquilibré, ~14 Converted) |

## Choix méthodologiques importants

- **Pas d'utilisation de l'âge comme variable prédictive** : dans OASIS, seuls
  les sujets âgés sont cliniquement évalués, ce qui créerait une corrélation
  artificielle âge/label sans rapport avec l'anatomie réelle.
- **Prétraitement "maison"** (Otsu + GMM) plutôt que FSL/FreeSurfer, par choix
  pragmatique — moins précis qu'un outil de référence, à documenter comme
  limite méthodologique.
- **Focalisation sur Nondemented vs Converted** (et non les 4 stades CDR) pour
  rester fidèle à la question de détection *précoce* posée par le sujet.

## Limites connues

- Échantillon très réduit de convertisseurs (~14 sujets) : les métriques du
  CNN doivent être interprétées avec prudence et complétées par un modèle
  classique (régression logistique) sur les biomarqueurs de la Partie 3.
- Segmentation tissulaire simplifiée (GMM sur intensités) plutôt qu'un outil
  de segmentation anatomique dédié.
- Pas de segmentation spécifique de l'hippocampe à ce stade (nécessiterait un
  atlas ou FSL-FIRST/FreeSurfer).

## Installation

```bash
python3 -m venv Alzheimer
source Alzheimer/bin/activate
pip install nibabel scipy scikit-image scikit-learn pandas matplotlib openpyxl torch torchvision ipykernel
python -m ipykernel install --user --name=alzheimer --display-name="Python (Alzheimer)"
```

## Structure du dépôt

```
├── notebooks/
│   ├── exploration.ipynb
│   ├── preprocessing.ipynb
│   ├── biomarkers_2D.ipynb
│   └── classification_CNN.ipynb
├── src/
│   └── Data/          (non versionné, voir .gitignore)
└── README.md
```

## Références

- Jack C.R. et al. (2018). *NIA-AA Research Framework: Toward a Biological Definition of Alzheimer's Disease.*
- Marcus D.S. et al. (2010). *Open Access Series of Imaging Studies (OASIS): Longitudinal MRI Data in Nondemented and Demented Older Adults.*
- Wen J. et al. (2020). *Convolutional Neural Networks for Classification of Alzheimer's Disease: Overview and Reproducible Evaluation.*
- [OASIS Brains](https://sites.wustl.edu/oasisbrains)