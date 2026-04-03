# TP4 - Intro à la statistique spatiale 2A - Autocorrélation spatiale

## Objectifs du TP

- S'initier à l'étude de l'**autocorrélation spatiale** sur données surfaciques
- Plateforme : [datalab SSP Cloud](https://datalab.sspcloud.fr)

```r
library(dplyr)
library(sf)
library(spdep)
library(leaflet)
library(RColorBrewer)
```

---

## Exercice 1 — Autocorrélation spatiale des revenus médians à Marseille

### 1. Création du jeu de données

Sélectionner la ville de Marseille dans le fond d'IRIS et réaliser la jointure avec les données de revenus.

---

### 2. Système de projection

Le système de projection de votre table est en **WGS84**. Convertissez-le en **Lambert-93** (EPSG 2154).

---

### 3. Résumé statistique et boxplot

- Faire un résumé statistique de la variable de revenu médian
- Faire un boxplot du revenu moyen en fonction des arrondissements

---

### 4. Supprimer les valeurs manquantes et cartographier

Supprimer les NA puis représenter la carte de Marseille selon les revenus avec `plot()` (tester différentes méthodes de discrétisation via l'argument `breaks`).

> Au vu de la carte, semble-t-il y avoir un phénomène spatial dans la distribution des revenus ?

---

### 5. Distribution aléatoire des revenus

Pour comparer la distribution réelle à une distribution aléatoire :

#### a. Permutation aléatoire

Créer une permutation aléatoire des revenus médians avec `sample()` et stocker le résultat dans une nouvelle variable `DISP_MED18_ALEA`.

#### b. Représentation cartographique

Représenter la distribution géographique de la variable aléatoire et comparer avec la distribution réelle. La distribution spatiale réelle est-elle proche de la distribution aléatoire ?

---

### 6. Mesure et test de l'autocorrélation spatiale

Un phénomène est **autocorrélé spatialement** quand la valeur d'une variable en un lieu est plus liée aux valeurs de ses voisins qu'aux valeurs des autres.

- **Autocorrélation positive** : les voisins tendent à prendre des valeurs similaires
- **Autocorrélation négative** : les voisins tendent à prendre des valeurs différentes

#### a. Type d'autocorrélation attendu

Quel type d'autocorrélation spatiale semble exister pour les revenus médians ?

#### b. Matrice de voisinage (contiguïté)

Construire la liste des voisins de chaque IRIS avec `spdep::poly2nb()` (contiguïté QUEEN par défaut). Prendre connaissance de l'objet et en faire un résumé avec `summary()`.

#### c. Nombre de voisins du 4e élément

Combien de voisins a le quatrième élément de la liste ?

---

### 7. Matrice de pondérations

#### a. Créer la liste de poids

```r
# Modèle
ponderation <- spdep::nb2listw(voisins, zero.policy = TRUE)
```

> `zero.policy = TRUE` permet d'intégrer les IRIS sans voisins.

#### b. Explorer l'objet

Utiliser `str(., max.level = 1)` et `summary()`.

#### c. Vérification

Vérifier que la somme des pondérations associées à chaque observation est bien égale à 1.

---

### 8. Diagramme de Moran

#### a. Variable standardisée

Créer une variable centrée-réduite avec `scale()`, nommée `DISP_MED18_STD`. Vérifier que mean = 0 et sd = 1.

#### b. Diagramme de Moran

```r
moran.plot(as.numeric(marseille$DISP_MED18_STD), listw = ponderation)
```

#### c. Interprétation des quadrants

Le diagramme croise :
- **Abscisse** : revenu médian observé dans l'IRIS (centré-réduit)
- **Ordonnée** : moyenne des revenus des IRIS voisins

Interpréter les quatre quadrants.

#### d. Conclusion sur l'autocorrélation

Les revenus médians semblent-ils autocorrélés spatialement ? Si oui, l'autocorrélation est-elle positive ou négative ?

---

### 9. I de Moran global

#### a. Calcul

```r
moran.test(marseille$DISP_MED18_STD, ponderation, randomisation = TRUE)
```

> `randomisation = TRUE` : la distribution observée est comparée à une distribution aléatoire par permutation.

#### b. Interprétation

Le résultat confirme-t-il ou infirme-t-il votre hypothèse ?

---

### 10. BONUS — Indicateurs locaux d'autocorrélation (LISA)

L'I de Moran global masque d'éventuelles hétérogénéités locales. Les **LISA** (Local Indicators of Spatial Association) — ou I de Moran locaux — permettent d'analyser l'autocorrélation à l'échelle de chaque IRIS.

#### a. Calcul des LISA

```r
mars_rev_lisa <- spdep::localmoran(marseille$DISP_MED18_STD, ponderation)
```

#### b. Explorer l'objet

Utiliser `class()`, `str(., max.level = 1)` et `summary()`.

#### c. Moyenne des indicateurs locaux

Quelle est la moyenne des Iᵢ ?

#### d. Nombre de LISA négatifs

Un LISA positif → l'IRIS est entouré d'IRIS aux revenus similaires.  
Un LISA négatif → l'IRIS est entouré d'IRIS aux revenus différents.

Combien d'indicateurs locaux sont négatifs ?

#### e. Carte des LISA

Ajouter les LISA comme variable du fond des IRIS (`LISA`). Utiliser :

```r
pal <- rev(RColorBrewer::brewer.pal(8, "RdYlBu"))
breaks <- c(-8.5, -1.2, -0.7, -0.1, 0, 0.1, 0.7, 1.2, 8.5)
```

#### f. Interprétation de la carte

Que révèle la carte ?

#### g. P-valeur des LISA

Repérer dans `mars_rev_lisa` la colonne correspondant à la p-valeur et la stocker dans la variable `LISA_PVAL` du fond des IRIS.

#### h. Nombre de LISA significatifs à 95 %

Combien de LISA sont significativement différents de zéro pour un niveau de confiance à 95 % ?

#### i. Carte des p-valeurs

Représenter les p-valeurs avec les bornes : `0, 0.01, 0.05, 0.1, 1`.

#### j. Zones significatives

Les zones repérées sur la carte des LISA font-elles partie des zones où les LISA sont les plus significatifs ?
