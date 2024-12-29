# Projet Python pour la Data Science

## Sujet : Prédiction des retards des vols au départ de l'aéroport de JFK en fonction des conditions météorologiques

### Auteurs
- **ED-DAZAZI Khawla**
- **GHORAFI Manal**
- **OUALY Oussama**

---

### Contexte
Ce projet a été réalisé dans le cadre du cours de **Python pour la Data Science** à l'**ENSAE** pour l'année universitaire **2024-2025**.
Il comporte, comme attendu, un jeu de données récupéré et traité, une partie visualisation et une partie modélisation.

### Problématique
On veut prédire les probabilités de retard des vols de départ en tenant compte de l’impact des conditions météorologiques pour mieux planifier les opérations aéroportuaires, réduire les retards et, par conséquent, améliorer l'expérience des usagers.

nous nous concentrerons sur les vols de la compagnie aérienne **American Airlines** à départ de **l'aéroport international John F. Kennedy (JFK) à New York**

### Objectifs
1. L'objectif principal de ce projet est de développer un modèle de prédiction capable d'anticiper les retards des vols à partir de variables liées aux conditions météorologiques.  
2. Réaliser un dashboard pour la compagnie Amercain Airlines 


### Plan
1.  **La récupération et le traitement des données**
2.  **L’analyse descriptive et la représentation graphique**
3.  **La modélisation**
4. **Conclusion**


---

## Installation et utilisation

Dans le notebook [Rapport_final.ipynb](Rapport_final.ipynb) se trouvent les instructions détaillées pour installer les éléments requis et pour utiliser le programme.

<small>Dans le cadre de notre projet, nous avons tenté d'automatiser la tâche de l'importation les données sur les vols à partir du site [Transtats BTS](https://www.transtats.bts.gov/ONTIME/Departures.aspx) à l’aide de Python, en utilisant la bibliothèque <span style="color:darkorange;">**requests**</span>. Cependant, cette démarche a échoué pour plusieurs raisons techniques que nous avons détaillées dans le ficher [Rapport_final.ipynb](Rapport_final.ipynb).

## Récupération de données sur les vols de départs

nous avons opté pour la méthode suivante :  
### . **Téléchargement manuel** :  
Pour télécharger la base de données nécessaire, veuillez suivre scrupuleusement les instructions suivantes :

1. Rendez-vous sur le site officiel de [Transtats BTS](https://www.transtats.bts.gov/ONTIME/Departures.aspx).
2. Effectuez les sélections suivantes dans les paramètres du formulaire :
   - **Origin Airport** : *New York, NY: John F. Kennedy International (JFK)*.
   - **Airline** : *American Airlines Inc. (AA)*.
   - **Month(s)** : *Sélectionnez tous les mois de l'année (January à December)*.
   - **Day(s)** : *Sélectionnez tous les jours du mois (1 à 31)*.
   - **Year(s)** : *Incluez les années 2020, 2021, 2022, 2023 et 2024*.
3. Validez vos choix, puis téléchargez le fichier Excel généré en cliquant sur l’option prévue à cet effet.
 

Bien sur, vous allez trouver la base de données dans le dossier `data` [`Detailed_Statistics_Departures.xlsx`](data/Detailed_Statistics_Departures.xlsx).

## Récupération de données sur les conditions météorologiques
Nous utilisons l'API Open-Meteo pour récupérer des données en  utilisant la bibliothèque <span style="color:darkorange;">**requests**</span>.

## Gestion des données en cas d'erreur API

Au cas où la requête API ne fonctionnerait pas, nous avons mis en place une méthode alternative pour garantir la reproductibilité de notre projet. Cette méthode repose sur l'utilisation de **MinIO**, une implémentation open-source du stockage compatible avec S3, dans le SSP Cloud.

### Stockage des données
Nous avons créé un dossier intitulé `diffusion` à la racine du bucket dans le SSP Cloud, accessible via l'URL : `oualy/diffusion`.

```python
import requests
import pandas as pd

# Requête API
url_api = "https://archive-api.open-meteo.com/v1/archive?latitude=40.64&longitude=-73.78&start_date=2020-11-01&end_date=2024-11-01&hourly=temperature_2m,relative_humidity_2m,dew_point_2m,apparent_temperature,precipitation,rain,snowfall,snow_depth,weather_code,pressure_msl,surface_pressure,cloud_cover,cloud_cover_low,cloud_cover_mid,cloud_cover_high,et0_fao_evapotranspiration,vapour_pressure_deficit,wind_speed_10m,wind_speed_100m,wind_direction_10m,wind_direction_100m,wind_gusts_10m,soil_temperature_0_to_7cm,soil_temperature_7_to_28cm,soil_temperature_28_to_100cm,soil_temperature_100_to_255cm,soil_moisture_0_to_7cm,soil_moisture_7_to_28cm,soil_moisture_28_to_100cm,soil_moisture_100_to_255cm&format=csv"

response_json = requests.get(url_api).json()
df_dpe = pd.json_normalize(response_json["results"])

# Vérification des premières lignes du DataFrame
df_dpe.head(2)

# Définition des chemins dans le bucket
MY_BUCKET = "oualy"
FILE_PATH_OUT_S3 = f"{MY_BUCKET}/diffusion/df_dpe.csv"

# Sauvegarde au format CSV
with fs.open(FILE_PATH_OUT_S3, "w") as file_out:
    df_dpe.to_csv(file_out)

# Vérification du contenu du dossier
fs.ls(f"{MY_BUCKET}/diffusion")

# Sauvegarde au format Parquet
FILE_PATH_OUT_S3 = f"{MY_BUCKET}/diffusion/df_dpe.parquet"

with fs.open(FILE_PATH_OUT_S3, "wb") as file_out:
    df_dpe.to_parquet(file_out)
```

### Utilisation des données sauvegardées

Les données peuvent être facilement récupérées depuis le bucket pour une utilisation ultérieure :

```python
FILE_PATH_S3 = f"{MY_BUCKET}/diffusion/df_dpe.csv"

# Importation des données
with fs.open(FILE_PATH_S3, "r") as file_in:
    df_dpe = pd.read_csv(file_in)

# Vérification des premières lignes du DataFrame
df_dpe.head(2)




