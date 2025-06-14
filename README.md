Voici un exemple de README clair et structuré pour ton projet basé sur le cahier des charges que tu as fourni :

---

# Contrôle qualité des surfaces brûlées issues de la chaîne de détections des surfaces brûlées de l'OEIL

## Description

Ce projet vise à déployer une solution automatisée de contrôle qualité des surfaces brûlées détectées via la chaîne de l’OEIL, en s’appuyant sur des données Sentinel-2 et des librairies Python (notamment Pystac et Stackstac).

Le PoC (Proof of Concept) a démontré la faisabilité de la méthode, un cahier des charges détaille les étapes nécessaires pour passer à un environnement de production, avec intégration, parallélisation, bancarisation des résultats, et enrichissement des données.

---

## Fonctionnalités principales

* Intégration automatisée des scripts PoC dans le système de l’OEIL avec gestion d’environnement (dotenv) et parallélisation (dask).
* Recherche, sélection et traitement des images Sentinel-2 sur une plage temporelle autour de la date de détection d'une surface brûlée.
* Calcul d’indices spectrales (NDVI, NBR, BAI, etc.) corrigés des offsets des bandes.
* Création de masques géographiques sur les surfaces brûlées à partir des géométries fournies.
* Bancarisation des données dans des fichiers netCDF.
* Préparation d’un catalogue STAC pour organiser les données produites.
* Enrichissement de la table attributaire des surfaces brûlées avec des indicateurs statistiques basés sur les variations temporelles des indices.

---

## Technologies et librairies utilisées

* **Python 3** (environnement Jupyter)
* Traitement spatial et satellite :  `rioxarray`, `rasterio`, `xarray`, `shapely`, `pyproj`, `geopandas`, `pandas`
* Calcul scientifique et visualisation : `numpy`, `matplotlib`
* Gestion environnement : `python-dotenv`
* Parallélisation : `dask`
* Chargement de données : `intake`,  `pystac-client`,`stackstac`

---

## Structure du traitement

1. **Prétraitement des données géospatiales**
   Conversion des multipolygones en polygones, extraction des dates, et définition de l’intervalle temporel (120 jours avant et 60 jours après la date de détection).

2. **Construction de la stack Sentinel-2**
   Recherche des scènes Sentinel-2 sur la bounding box d’une surface brûlée, sans filtre restrictif sur la couverture nuageuse.

3. **Filtrage des scènes selon la couverture nuageuse**
   Sélection d’images où la zone brûlée est visible, grâce à la classification des pixels.

4. **Calcul des indices spectraux**
   Correction des offsets, calcul des indices NDVI, NBR, BAI, etc.

5. **Création du masque de surface brûlée**
   Masquage des pixels hors de la géométrie de la surface brûlée.

6. **Bancarisation des résultats**
   Sauvegarde des données traitées dans des fichiers netCDF, avec nettoyage des variables inutiles.

7. **Enrichissement des données**
   Analyse des variations des indices sur des intervalles courts pour détecter et qualifier les événements de feu.

---

## Exemple de fonction clé

```python
def select_scenes_without_cc(gdf_filter, stack):
    """
    Sélectionne les scènes Sentinel-2 où la zone brûlée est visible (couverture nuageuse ≤ 5%).
    """
    data_times = pd.to_datetime(stack['time']).date
    dates_burnedarea = gdf_filter['date_'].values
    images_to_keep = []

    for i, time in enumerate(data_times):
        if time in dates_burnedarea:
            images_to_keep.append(i)
            continue
        scl_data = stack.isel(time=i).sel(band="scl")
        mask = (scl_data >= 4) & (scl_data <= 7)
        filtered_data = scl_data.where(mask)
        percentage = filtered_data.count() / scl_data.count() * 100
        if percentage > 95:
            images_to_keep.append(i)

    return stack.isel(time=images_to_keep)
```

---

## Déploiement et intégration

* Parallélisation avec `dask` pour optimiser les traitements lourds.
* Adaptation des scripts à l’environnement production, gestion des variables d’environnement.
* Étude possible du déploiement via Docker.
* Mise en place d’un catalogue STAC pour la gestion des données produites.

---

## Perspectives et améliorations

* Réalisation d’une étude statistique approfondie sur un échantillon plus large pour définir les seuils précis d’identification des événements feu.
* Enrichissement automatique de la table attributaire des surfaces brûlées avec des indicateurs temporels et statistiques.
* Exploitation des résultats pour automatiser la validation des surfaces brûlées détectées.

---

## Estimation des charges

* Intégration, parallélisation et mise en production : environ 6 jours (avec marge de sécurité).
* Mise en place du catalogue STAC : 1-2 jours.
* Développement de l’enrichissement des données attributaires : 2 jours.

---

## Contact

**Thomas Avron**
Email : [tavron@apid.nc](mailto:tavron@apid.nc)

**Hugo Roussaffa**
Email : [hugo.roussaffa@oeil.nc](mailto:hugo.roussaffa@oeil.nc)
