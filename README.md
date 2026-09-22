# Atelier Préparation de Données Images

Nettoyage et préparation d'un jeu de données d'images de déchets, en vue de
l'entraînement d'un modèle de classification (Machine Learning / Deep Learning).

## Contexte

Une entreprise souhaite développer un système capable de reconnaître
automatiquement le type de déchet présent sur une photographie, afin
d'améliorer le tri des déchets. Le modèle doit classer chaque image dans
l'une des six catégories suivantes :

- **cardboard** : cartons ondulés, cartons plats...
- **plastic** : bouteilles, emballages plastiques...
- **paper** : feuilles, journaux...
- **glass** : bouteilles et objets en verre...
- **metal** : canettes, boîtes métalliques...
- **trash** : emballages bonbons, tasses jetables...

Les images collectées proviennent de plusieurs sources et ne sont donc pas
homogènes : dimensions et formats différents, images RGB et grayscale,
images trop petites, corrompues, vides, dupliquées, mal classées, et classes
déséquilibrées. L'objectif de l'atelier est de construire un jeu de données
propre et homogène, prêt à l'entraînement.

## Structure du projet

```
atelier_prepa_donnees_images/
│
├── notebooks/
│   └── atelier_prepa_donnees_images.ipynb
├── reports/
│   └── audit_images.csv
└── data/
    ├── raw/           # dataset original, lecture seule
    │   ├── cardboard/
    │   ├── glass/
    │   ├── metal/
    │   ├── paper/
    │   ├── plastic/
    │   └── trash/
    ├── cleaned/        # images nettoyées et redimensionnées (224x224)
    │   └── ... (mêmes 6 classes)
    ├── train/           # 70 % du dataset nettoyé, stratifié par classe
    ├── valid/           # 15 %
    └── test/            # 15 %
```

`data/raw/` reste en lecture seule tout au long de l'atelier : aucune image
n'y est modifiée ni supprimée. Les résultats des traitements sont toujours
écrits ailleurs (`cleaned/`, `train/`, `valid/`, `test/`).

## Pipeline de nettoyage (notebook)

Le notebook `atelier_prepa_donnees_images.ipynb` suit les étapes suivantes,
dans l'ordre :

1. **Exploration** — construction d'un tableau (`DataFrame`) recensant, pour
   chaque image : nom, classe, format, mode, dimensions, écart-type des
   pixels, nombre de canaux, taille en octets.
2. **Détection des images corrompues** — via `Image.verify()` puis
   `Image.load()`, pour repérer aussi les fichiers dont l'en-tête est valide
   mais dont les pixels sont tronqués.
3. **Détection des images vides** — image entièrement noire, entièrement
   blanche, ou à très faible variation de pixels (seuil sur l'écart-type).
4. **Analyse des résolutions** — résolution min/max, résolutions les plus
   fréquentes, détection des images sous la taille minimale (64 × 64).
5. **Analyse des canaux** — répartition par mode (`RGB`, `RGBA`, `P`...) et
   par nombre de canaux.
6. **Détection des doublons** — via un hash des pixels décodés (et non des
   octets bruts du fichier), pour repérer les copies même enregistrées dans
   un format ou une compression différente. Les doublons présents dans deux
   classes différentes sont un signal fort d'image mal classée.
7. **Contrôle visuel** — parcours par grilles d'images, classe par classe,
   pour identifier les images mal placées.
8. **Analyse du déséquilibre des classes** — comptage avant/après nettoyage,
   ratio majoritaire/minoritaire.
9. **Redimensionnement** — toutes les images conservées sont redimensionnées
   en 224 × 224, en conservant les proportions et en ajoutant du padding
   noir pour compléter le canevas.
10. **Uniformisation des canaux** — toutes les images sont converties en RGB
    (la transparence RGBA/LA est fusionnée sur fond blanc).
11. **Normalisation des pixels** — fonction de chargement qui renvoie les
    images sous forme de tableaux `float32` dans l'intervalle [0, 1].
12. **Découpage train/valid/test** — répartition stratifiée (70/15/15) par
    classe, avec copie physique des fichiers dans `data/train/`,
    `data/valid/` et `data/test/`.
13. **Data augmentation** — génération d'images augmentées (rotation,
    translation, zoom, flip horizontal, luminosité, contraste, variation de
    couleur) pour la classe minoritaire de `train/`, jusqu'à équilibrer son
    nombre d'images avec la classe majoritaire.
14. **Bonus** — fonctionnalité additionnelle proposée librement.

## Décisions prises pendant le nettoyage

- **Images conservées dans `cleaned/`** : ni corrompues, ni vides, ni trop
  petites (< 64 × 64), ni doublons, ni exclues manuellement (images de test
  synthétiques sans rapport avec un déchet réel).
- **Images mal classées** : reclassées dans leur vraie catégorie
  (`classe_finale`) après contrôle visuel, plutôt que supprimées.
- **Doublons entre classes différentes** : tranchés au cas par cas après
  inspection visuelle, en gardant l'exemplaire de la bonne classe.
- **Padding de redimensionnement** : noir, choisi pour ne pas introduire de
  biais de luminosité par rapport aux vraies photos.

## Rapport d'audit

Le fichier `reports/audit_images.csv` résume les indicateurs suivants,
calculés sur le dataset `raw/` :

- Total d'images
- Images corrompues
- Images quasi vides
- Images trop petites
- Doublons
- Images grayscale
- Images RGBA
- Images mal classées

## Prérequis

Les dépendances sont listées dans `requirements.txt` (numpy, pandas,
Pillow, matplotlib, tensorflow — ce dernier fournit Keras, utilisé pour la
data augmentation). Installation :

```
pip install -r requirements.txt
```

## Utilisation

1. Installer les dépendances (`pip install -r requirements.txt`).
2. Placer les images du dataset original dans `data/raw/<classe>/`.
3. Ouvrir `notebooks/atelier_prepa_donnees_images.ipynb` et exécuter les
   cellules dans l'ordre, de haut en bas.
4. Le dossier `data/cleaned/` puis les dossiers `data/train/`,
   `data/valid/` et `data/test/` doivent être créés (dossiers de classes
   vides) avant l'exécution des parties correspondantes.
5. Consulter `reports/audit_images.csv` pour la synthèse des indicateurs de
   qualité du dataset original.
