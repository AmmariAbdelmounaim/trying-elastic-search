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

### 1. Recherche des films de Ridley Scott
1. Recherche multi-champs
```powershell
curl -X GET "localhost:9205/movies/_search" -H "Content-Type: application/json" -d '{
  "query": {
    "multi_match": {
      "query": "Ridley Scott",
      "fields": ["fields.directors", "fields.plot", "fields.title"]
    }
  }
}'
```

### 2. Recherche spécifique dans le champ "directors"
```powershell
curl -X GET "localhost:9205/movies/_search" -H "Content-Type: application/json" -d '{
  "query": {
    "match": {
      "fields.directors": "Ridley Scott"
    }
  }
}'
```
### 3. Recherche combinée (Ridley Scott + Russell Crowe)
```powershell
curl -X GET "localhost:9205/movies/_search" -H "Content-Type: application/json" -d '{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "fields.directors": "Ridley Scott"
          }
        },
        {
          "match": {
            "fields.actors": "Russell Crowe"
          }
        }
      ]
    }
  }
}'
```


# Requêtes Complexes ElasticSearch

## 1. Films de Ridley Scott (multi-champs)
```json
{
  "query": {
    "multi_match": {
      "query": "Ridley Scott",
      "fields": ["fields.directors", "fields.plot", "fields.title"]
    }
  }
}
```

## 2. Films de Ridley Scott (spécifiquement comme réalisateur)
```json
{
  "query": {
    "match": {
      "fields.directors": "Ridley Scott"
    }
  }
}
```

## 3. Films de Ridley Scott avec Russell Crowe
```json
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "fields.directors": "Ridley Scott"
          }
        },
        {
          "match": {
            "fields.actors": "Russell Crowe"
          }
        }
      ]
    }
  }
}
```

## 4. Films de Ridley Scott avec rank < 2000
```json
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "fields.directors": "Ridley Scott"
          }
        },
        {
          "range": {
            "fields.rank": {
              "lt": 2000
            }
          }
        }
      ]
    }
  }
}
```

## 5. Films de Ridley Scott avec rating > 6 et genre Adventure
```json
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "fields.directors": "Ridley Scott"
          }
        },
        {
          "range": {
            "fields.rating": {
              "gt": 6
            }
          }
        },
        {
          "match": {
            "fields.genres": "Adventure"
          }
        }
      ]
    }
  }
}
```

Pour exécuter ces requêtes :
Ou avec curl :
```bash
curl -X GET "localhost:9205/movies/_search" -H "Content-Type: application/json" -d '[Insérer le JSON ici]'
```



# Requêtes d'Agrégation ElasticSearch

## 1. Nombre de films par année
```json
{
  "size": 0,
  "aggs": {
    "films_par_annee": {
      "terms": {
        "field": "fields.year",
        "size": 100,
        "order": {
          "_count": "desc"
        }
      }
    }
  }
}
```


## 2. Statistiques des films de Ridley Scott (rank et note moyens)
```json
{
  "size": 0,
  "query": {
    "match": {
      "fields.directors": "Ridley Scott"
    }
  },
  "aggs": {
    "rank_moyen": {
      "avg": {
        "field": "fields.rank"
      }
    },
    "note_moyenne": {
      "avg": {
        "field": "fields.rating"
      }
    }
  }
}
```


## 3. Note moyenne par année (triée)
```json
{
  "size": 0,
  "aggs": {
    "notes_par_annee": {
      "terms": {
        "field": "fields.year",
        "size": 100,
        "order": {
          "note_moyenne": "desc"
        }
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


## 4. Distribution des films par plage de notes
```json
{
  "size": 0,
  "aggs": {
    "distribution_notes": {
      "range": {
        "field": "fields.rating",
        "ranges": [
          {
            "to": 2,
            "key": "Très mauvais (<2)"
          },
          {
            "from": 3,
            "to": 5,
            "key": "Moyen (3-5)"
          },
          {
            "from": 7,
            "key": "Très bon (>7)"
          }
        ]
      }
    }
  }
}
```


Pour exécuter ces requêtes en PowerShell :
```powershell
$query = @'
[Insérer le JSON de la requête ici]
'@
Invoke-RestMethod -Method GET -Uri "http://localhost:9205/movies/_search" -ContentType "application/json" -Body $query
```


**Explications :**

1. **Nombre de films par année**
   - `size: 0` : ne retourne que les agrégations, pas les documents
   - `terms` : groupe par valeur unique
   - `order`: trie les résultats par compte descendant

2. **Statistiques Ridley Scott**
   - Combine une requête de filtrage avec des agrégations
   - Utilise `avg` pour calculer les moyennes

3. **Note moyenne par année**
   - Agrégation imbriquée : groupe d'abord par année
   - Calcule la moyenne des notes pour chaque groupe
   - Trie par note moyenne décroissante

4. **Distribution par plage de notes**
   - Utilise `range` pour définir des intervalles
   - Chaque intervalle a une clé descriptive
   - Les bornes sont inclusives/exclusives selon leur position (from/to)

**Notes importantes :**
- `size: 0` est utilisé pour ne pas retourner les documents, seulement les agrégations
- Les agrégations peuvent être combinées avec des requêtes normales
- L'ordre des résultats peut être défini sur différents critères
- Les clés personnalisées rendent les résultats plus lisibles



# TP Kibana - Analyse de Films

## Installation
L'installation d'Elasticsearch et Kibana a été réalisée via Docker :

```bash
# Installation d'Elasticsearch
docker run -p 9205:9200 -p 9305:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.17.0
```


## Configuration et Import des Données
1. Création d'un mapping approprié avec des champs agrégeables pour directors et genres
2. Import des données via l'API bulk d'Elasticsearch
3. Création d'un index pattern dans Kibana avec "fields.release_date" comme champ temporel

## Discover - Analyse des Films

### Films depuis 1950
![3](https://github.com/user-attachments/assets/bf6ec4cb-9251-46a8-8e3f-1a390531a1dc)
*Figure 1: Vue Discover montrant la distribution des films depuis 1950*

Comme montré dans la capture d'écran, nous avons :
- Utilisé le filtre `fields.year >= 1950`
- Le graphique temporel montre la distribution des films sur la période
- 4,706 films correspondent à ce critère
- On observe une augmentation significative du nombre de films à partir des années 1990

### Films de Ridley Scott
![4](https://github.com/user-attachments/assets/2fad2cad-ed4c-454a-8b79-b3cb1abc9133)
*Figure 2: Filtrage des films de Ridley Scott*


## Création du Camembert

*Figure 4: Diagramme circulaire des principaux réalisateurs et leurs genres*
![5](https://github.com/user-attachments/assets/068875b8-c27c-4e76-a666-22c4ba5f442a)
Pour créer le diagramme circulaire montrant les 6 principaux réalisateurs et leurs genres :
1. Utiliser l'agrégation sur `fields.directors.keyword` (top 6)
2. Sous-agrégation sur `fields.genres.keyword`
3. Les données sont maintenant agrégeables grâce au mapping modifié incluant les champs `.keyword`

Cette visualisation permet de voir rapidement :
- Les réalisateurs les plus prolifiques
- La diversité des genres pour chaque réalisateur
- La distribution des films par genre pour chaque réalisateur
