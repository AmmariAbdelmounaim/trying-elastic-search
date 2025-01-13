# TP Elasticsearch

## Table des matières
1. [Introduction](#introduction)
2. [Installation](#installation)
3. [Configuration](#configuration)
4. [Structure des données](#structure-des-données)
5. [Importation des données](#importation-des-données)
6. [Requêtes](#requêtes)
7. [Exercices pratiques](#exercices-pratiques)

## Introduction
Elasticsearch est un moteur de recherche et d'analyse distribué, conçu pour le traitement de données textuelles, numériques et géospatiales. Ce TP couvre l'installation, la configuration et l'utilisation d'Elasticsearch pour créer un moteur de recherche de films.

## Installation
Pour ce TP, nous utilisons Docker pour installer Elasticsearch. Voici la commande utilisée :

```bash
docker run -p 9205:9200 -p 9305:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.17.0
```

Vérification de l'installation :
```bash
curl localhost:9205
```
![image](https://github.com/user-attachments/assets/5e0fb95b-d2d5-4484-bff9-0962ac56bee2)

## Configuration
### Structure du projet
```
projet/
│
├── mapping.json    # Définition de la structure
└── movies.json     # Données des films
```

## Structure des données
### Mapping (mapping.json)
```json
{
    "mappings": {
        "properties": {
            "fields": {
                "properties": {
                    "actors": {
                        "type": "text",
                        "fields": {
                            "raw": {
                                "type": "text",
                                "index": false
                            }
                        }
                    },
                    "directors": {
                        "type": "text",
                        "fields": {
                            "raw": {
                                "type": "text",
                                "index": false
                            }
                        }
                    },
                    // ... autres champs
                }
            }
        }
    }
}
```

### Format des données (movies.json)
```json
{"index":{"_index": "movies","_id":1}}
{
    "fields": {
        "title": "Don Jon",
        "directors": ["Joseph Gordon-Levitt"],
        "actors": ["Joseph Gordon-Levitt", "Scarlett Johansson"],
        "release_date": "2013-01-18T00:00:00Z",
        "rating": 7.4,
        "genres": ["Comedy", "Drama"]
        // ... autres champs
    }
}
```

## Importation des données

### Création de l'index avec mapping
```powershell
$mapping = Get-Content -Raw mapping.json
Invoke-RestMethod -Method PUT -Uri "http://localhost:9205/movies" -ContentType "application/json" -Body $mapping
```

### Import des données
```powershell
$data = Get-Content -Raw movies.json
Invoke-RestMethod -Method POST -Uri "http://localhost:9205/_bulk" -ContentType "application/json" -Body $data
```

## Requêtes

### 1. Vérification de base
```powershell
# Nombre total de documents
GET localhost:9205/movies/_count

# Détails de l'index
GET localhost:9205/movies

# Premiers 10 films
GET localhost:9205/movies/_search
{
  "size": 10
}
```

### 2. Recherche simple
```powershell
# Recherche par titre
GET localhost:9205/movies/_search
{
  "query": {
    "match": {
      "fields.title": "Inception"
    }
  }
}
```

### 3. Recherche avec filtres
```powershell
# Films avec une note supérieure à 8
GET localhost:9205/movies/_search
{
  "query": {
    "range": {
      "fields.rating": {
        "gte": 8.0
      }
    }
  }
}
```

### 4. Agrégations
```powershell
# Note moyenne des films
GET localhost:9205/movies/_search
{
  "size": 0,
  "aggs": {
    "avg_rating": {
      "avg": {
        "field": "fields.rating"
      }
    }
  }
}
```

### 5. Recherche complexe
```powershell
# Films avec un acteur spécifique et une note minimum
GET localhost:9205/movies/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "fields.actors": "Brad Pitt"
          }
        },
        {
          "range": {
            "fields.rating": {
              "gte": 7.0
            }
          }
        }
      ]
    }
  }
}
```

## Cas pratiques

1. Trouver tous les films d'un réalisateur spécifique
2. Lister les films par genre
3. Calculer la note moyenne par année
4. Rechercher des films par mots-clés dans le résumé
5. Trouver les films avec les meilleures notes pour une année donnée

### Trouver tous les films d'un réalisateur spécifique
```powershell
GET localhost:9205/movies/_search
{
  "query": {
    "match": {
      "fields.directors": "Christopher Nolan"
    }
  },
  "sort": [
    {
      "fields.year": "asc"
    }
  ]
}
```
### 2. Lister les films par genre
Pour lister les films regroupés par genre, utilisez une agrégation :

``` powershell
GET localhost:9205/movies/_search
{
  "size": 0,
  "aggs": {
    "films_par_genre": {
      "terms": {
        "field": "fields.genres.keyword",
        "size": 10
      }
    }
  }
}
```

### 3. Calculer la note moyenne par année
Utilisez une agrégation date_histogram pour calculer la note moyenne par année :

```powershell
GET localhost:9205/movies/_search
{
  "size": 0,
  "aggs": {
    "notes_par_année": {
      "date_histogram": {
        "field": "fields.release_date",
        "calendar_interval": "year"
      },
      "aggs": {
        "note_moyenne": {
          "avg": {
            "field": "fields.rating"
          }
        }
      }
    }
  }
}
```

### 4. Rechercher des films par mots-clés dans le résumé
Si le champ summary est indexé comme text, utilisez une requête match pour effectuer une recherche par mots-clés :

```powershell
GET localhost:9205/movies/_search
{
  "query": {
    "match": {
      "fields.summary": "rêve braquage"
    }
  }
}
```

### 5. Trouver les films avec les meilleures notes pour une année donnée
Pour trouver les films les mieux notés pour une année spécifique (par exemple, 2010), combinez une requête range et un tri :

```powershell
GET localhost:9205/movies/_search
{
  "query": {
    "bool": {
      "must": [
      {
          "range": {
            "fields.release_date": {
              "gte": "2010-01-01",
              "lte": "2010-12-31"
            }
          }
        }
      ]
    }
  },
  "sort": [
    {
      "fields.rating": "desc"
    }
  ],
  "size": 10
}
```

## Conclusion
Ce TP nous a permis de :
- Installer et configurer Elasticsearch avec Docker
- Comprendre la structure des données et le mapping
- Importer des données en masse
- Effectuer différents types de recherches et d'analyses
- Manipuler des données réelles dans un contexte pratique

## Ressources supplémentaires
- [Documentation officielle Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
- [Guide des requêtes](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)
- [Documentation des agrégations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)
