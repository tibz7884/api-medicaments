# API GraphQL pour la Base de Données Publique des Médicaments

![Visualisation du graphe](./visualization.png)

## Utilisation

Démarrer le serveur :

```bash
node src/server.js
```

Une fois le serveur démarré, les requêtes (HTTP POST) peuvent être envoyées à [`localhost:4000/graphql`](http://localhost:4000/graphql) (accéder à cette adresse depuis un navigateur affichera une interface graphique).

```bash
curl http://localhost:4000/graphql -H "Content-Type: application/graphql" -d "{ medicaments(CIS: [62204255]) { denomination } }"

# Si la requête GraphQL est contenue dans le fichier "requete.graphql" :
curl http://localhost:4000/graphql -H "Content-Type: application/graphql" -d @requete.graphql
```

## Exemples de requêtes

Le schéma complet est disponible dans [`schema.graphql`](./schema.graphql).

### Requête par code

Demander un médicament à partir de son CIS (Code Identifiant de Spécialité), sa dénomination, les substances qu'il contient avec leur code et leur dénomination :

```graphql
{
  medicaments(CIS: ["68572075"]) {
    denomination
    substances {
      code_substance
      denomination
    }
  }
}
```

Réponse :

```json
{
  "data": {
    "medicaments": [
      {
        "denomination": "FAMOTIDINE MYLAN 20 mg, comprimé pelliculé",
        "substances": [
          {
            "code_substance": "04034",
            "denomination": "FAMOTIDINE"
          }
        ]
      }
    ]
  }
}
```

Demander une substance à partir de son code, sa dénomination, et tous les médicaments qui la contiennent avec leur dénomination :

```graphql
{
  substances(codes_substances: ["04034"]) {
    denomination
    medicaments {
      denomination
    }
  }
}
```

Réponse :

```json
{
  "data": {
    "substances": [
      {
        "denomination": "FAMOTIDINE",
        "medicaments": [
          {
            "denomination": "FAMOTIDINE EG 20 mg, comprimé pelliculé"
          },
          {
            "denomination": "FAMOTIDINE MYLAN 40 mg, comprimé pelliculé"
          },
          {
            "denomination": "FAMOTIDINE EG 40 mg, comprimé pelliculé"
          },
          {
            "denomination": "FAMOTIDINE MYLAN 20 mg, comprimé pelliculé"
          }
        ]
      }
    ]
  }
}
```

Demander un groupe générique à partir de son identifiant, et tous les médicaments qu'ils contient, avec pour chaque médicament leur dénomination et celle de leurs substances :

```graphql
{
  groupes_generiques(ids: ["4"]) {
    libelle
    type
    medicaments {
      denomination
      substances {
        denomination
      }
    }
  }
}
```

Réponse :

```json
{
  "data": {
    "groupes_generiques": [
      {
        "libelle": "CIMETIDINE 800 mg - TAGAMET 800 mg, comprimé pelliculé sécable",
        "type": "princeps",
        "medicaments": [
          {
            "denomination": "CIMETIDINE MYLAN 800 mg, comprimé pelliculé",
            "substances": [
              {
                "denomination": "CIMÉTIDINE"
              }
            ]
          },
          {
            "denomination": "CIMETIDINE MYLAN 400 mg, comprimé pelliculé",
            "substances": [
              {
                "denomination": "CIMÉTIDINE"
              }
            ]
          }
        ]
      }
    ]
  }
}
```

### Requête par produit

Demander tous les médicaments avec leur dénomination, et toutes les substances avec leur dénomination (l'argument `limit` permet de limiter le nombre de résultats) :

```graphql
{
  medicaments(limit: 3) {
    denomination
  }
  
  substances(limit: 3) {
    denomination
  }
}
```

Réponse :

```json
{
  "data": {
    "medicaments": [
      {
        "denomination": "ANASTROZOLE ACCORD 1 mg, comprimé pelliculé"
      },
      {
        "denomination": "RANITIDINE BIOGARAN 150 mg, comprimé effervescent"
      },
      {
        "denomination": "ACTAEA RACEMOSA FERRIER, degré de dilution compris entre 2CH et 30CH ou entre 4DH et 60DH"
      }
    ],
    "substances": [
      {
        "denomination": "CHLORHYDRATE DE LOPÉRAMIDE"
      },
      {
        "denomination": "ACIDE ACÉTYLSALICYLIQUE"
      },
      {
        "denomination": "ASPARTIQUE (ACIDE)"
      }
    ]
  }
}
```

### Requête paginée

Pour une même requête, les éléments sont toujours renvoyés dans le même ordre.

Il est possible d'effectuer une requête paginée avec les arguments `from` et `limit` :

- `from` est l'index du premier élément à renvoyer (à partir de 0),
- `limit` est le nombre maximum d'éléments à renvoyer.

Exemple :

```graphql
{
  page_1: presentations(limit: 3, from: 0) {
    CIP7
    libelle
  }
  
  page_2: presentations(limit: 3, from: 3) {
    CIP7
    libelle
  }
  
  pages_1_et_2: presentations(limit: 6) {
    CIP7
    libelle
  }
}
```

Réponse :

```json
{
  "data": {
    "page_1": [
      {
        "CIP7": "2160191",
        "libelle": "plaquette(s) thermoformée(s) aluminium de 28 comprimé(s)"
      },
      {
        "CIP7": "2160363",
        "libelle": "4 poche(s) bicompartimenté(e)(s) polymère multicouches BIOFINE de 1000 ml"
      },
      {
        "CIP7": "2160417",
        "libelle": "flacon(s) polyéthylène haute densité (PEHD) de 28 comprimé(s)"
      }
    ],
    "page_2": [
      {
        "CIP7": "2160423",
        "libelle": "flacon(s) polyéthylène haute densité (PEHD) de 14 comprimé(s)"
      },
      {
        "CIP7": "2160469",
        "libelle": "1 flacon(s) polyéthylène de 5 ml avec compte-gouttes"
      },
      {
        "CIP7": "2160908",
        "libelle": "4 seringue(s) préremplie(s) en verre de 0,5 ml dans stylo pré-rempli"
      }
    ],
    "pages_1_et_2": [
      {
        "CIP7": "2160191",
        "libelle": "plaquette(s) thermoformée(s) aluminium de 28 comprimé(s)"
      },
      {
        "CIP7": "2160363",
        "libelle": "4 poche(s) bicompartimenté(e)(s) polymère multicouches BIOFINE de 1000 ml"
      },
      {
        "CIP7": "2160417",
        "libelle": "flacon(s) polyéthylène haute densité (PEHD) de 28 comprimé(s)"
      },
      {
        "CIP7": "2160423",
        "libelle": "flacon(s) polyéthylène haute densité (PEHD) de 14 comprimé(s)"
      },
      {
        "CIP7": "2160469",
        "libelle": "1 flacon(s) polyéthylène de 5 ml avec compte-gouttes"
      },
      {
        "CIP7": "2160908",
        "libelle": "4 seringue(s) préremplie(s) en verre de 0,5 ml dans stylo pré-rempli"
      }
    ]
  }
}
```

### Requête filtrée par date

Il est possible de filtrer les médicaments par date d'autorisation de mise sur le marché (AMM).

Le paramètre `date_AMM` est [de type `DateFilter`](./schema.graphql) et a deux propriétés :

- `before` renverra les médicaments mis sur le marché avant ou à cette date,
- `after` renverra les médicaments mis sur le marché à ou après cette date.

Spécifier une même date pour les deux propriétés renverra les médicaments mis sur le marché à cette date exactement.

Une date doit avoir un des formats suivants :

- `DD/MM/YYYY`
- `DD-MM-YYYY`
- `YYYY-MM-DD`

Exemple :

```graphql
{
  dateAvant: medicaments(
    date_AMM: {before: "22/12/1999"},
    limit: 2
  ) {
    denomination
    date_AMM
  }
  
  dateApres: medicaments(
    date_AMM: {after: "22/12/1999"},
    limit: 2
  ) {
    denomination
    date_AMM
  }
  
  dateExacte: medicaments(
    date_AMM: {after: "22/12/1999", before: "22/12/1999"}
  ) {
    denomination
    date_AMM
  }
  
  periode: medicaments(
    date_AMM: {after: "01/11/1999", before: "08/11/1999"}
  ) {
    denomination
    date_AMM
  }
}
```

Réponse :

```json
{
  "data": {
    "dateAvant": [
      {
        "denomination": "RANITIDINE BIOGARAN 150 mg, comprimé effervescent",
        "date_AMM": "04/07/1989"
      },
      {
        "denomination": "FENOFIBRATE TEVA 100 mg, gélule",
        "date_AMM": "06/12/1996"
      }
    ],
    "dateApres": [
      {
        "denomination": "ANASTROZOLE ACCORD 1 mg, comprimé pelliculé",
        "date_AMM": "28/10/2010"
      },
      {
        "denomination": "ACTAEA RACEMOSA FERRIER, degré de dilution compris entre 2CH et 30CH ou entre 4DH et 60DH",
        "date_AMM": "03/01/2008"
      }
    ],
    "dateExacte": [
      {
        "denomination": "FAMOTIDINE EG 20 mg, comprimé pelliculé",
        "date_AMM": "22/12/1999"
      },
      {
        "denomination": "FAMOTIDINE EG 40 mg, comprimé pelliculé",
        "date_AMM": "22/12/1999"
      }
    ],
    "periode": [
      {
        "denomination": "KABIVEN, émulsion pour perfusion",
        "date_AMM": "08/11/1999"
      },
      {
        "denomination": "CISPLATINE MYLAN 25 mg/25 ml, solution à diluer pour perfusion",
        "date_AMM": "02/11/1999"
      }
    ]
  }
}
```

### Autres exemples

Demander un médicament à partir de son CIS, et les présentations sous lesquelles il est vendu, associées à plusieurs caractéristiques :

```graphql
{
  medicaments(CIS: ["62204255"]) {
    denomination
    presentations {
      CIP7
      libelle
      taux_remboursement
      prix_sans_honoraires
      prix_avec_honoraires
    }
  }
}
```

Réponse :

```json
{
  "data": {
    "medicaments": [
      {
        "denomination": "AMLODIPINE PFIZER 5 mg, gélule",
        "presentations": [
          {
            "CIP7": "3334167",
            "libelle": "plaquette(s) PVC PVDC aluminium de 30 gélule(s)",
            "taux_remboursement": 65,
            "prix_sans_honoraires": 4.36,
            "prix_avec_honoraires": 5.38
          },
          {
            "CIP7": "3823529",
            "libelle": "plaquette(s) PVC PVDC aluminium de 90 gélule(s)",
            "taux_remboursement": 65,
            "prix_sans_honoraires": 12.32,
            "prix_avec_honoraires": 15.08
          }
        ]
      }
    ]
  }
}
```

Les requêtes peuvent être très imbriquées. Par exemple, en partant d'une substance, il est possible de demander la liste des médicaments qui la contiennent, et pour chaque médicament, les présentations sous lesquelles ils sont vendus :

```graphql
{
  substances(codes_substances: ["01743"]) {
    denomination
    medicaments {
      denomination
      presentations {
        libelle
        taux_remboursement
      }
    }
  }
}
```

Réponse :

```json
{
  "data": {
    "substances": [
      {
        "denomination": "DIPYRIDAMOLE",
        "medicaments": [
          {
            "denomination": "PERSANTINE 75 mg, comprimé enrobé",
            "presentations": [
              {
                "libelle": "plaquette(s) PVC PVDC aluminium de 30 comprimé(s)",
                "taux_remboursement": null
              }
            ]
          },
          {
            "denomination": "ASASANTINE L.P. 200 mg/25 mg, gélule à libération prolongée",
            "presentations": [
              {
                "libelle": "flacon(s) polypropylène de 60 gélule(s)",
                "taux_remboursement": 65
              }
            ]
          },
          {
            "denomination": "CLERIDIUM 150 mg, comprimé pellicullé sécable",
            "presentations": [
              {
                "libelle": "plaquette(s) thermoformée(s) PVC-aluminium de 60 comprimé(s)",
                "taux_remboursement": null
              }
            ]
          },
          {
            "denomination": "PERSANTINE 10 mg/2 mL, solution injectable, ampoule",
            "presentations": [
              {
                "libelle": "10 ampoule(s) en verre de 2  ml",
                "taux_remboursement": null
              }
            ]
          }
        ]
      }
    ]
  }
}
```

Etc.
