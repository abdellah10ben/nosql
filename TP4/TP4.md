# TP n°4 – NoSQL
## MapReduce avec CouchDB et MongoDB

---

## Informations générales
- **Module** : Bases de données NoSQL  
- **TP** : TP4 – MapReduce  
- **Étudiant** : **Benmessaoud Abdellah**

---

## Objectif du TP
Ce TP a pour objectif de comprendre et manipuler le modèle **MapReduce** dans un contexte NoSQL, à travers :
- l’installation et l’utilisation de **CouchDB** (API REST + vues MapReduce),
- l’exécution d’exercices **MapReduce dans MongoDB** sur une collection `films`.

---

## Partie 1 – Installation et utilisation de CouchDB

### 1) Mise en place de CouchDB avec Docker

#### a) Création d’un volume (optionnel mais recommandé)
```bash
docker volume create couchdb_data
```

#### b) Lancement du conteneur
Exemple de lancement (utilisateur admin + port 5984 + volume) :
```bash
docker run --name couchdb \
  -e COUCHDB_USER=admin \
  -e COUCHDB_PASSWORD=admin \
  -p 5984:5984 \
  -v couchdb_data:/opt/couchdb/data \
  -d couchdb
```

#### c) Accès à l’interface graphique (Fauxton)
L’interface web d’administration est accessible via :
```
http://localhost:5984/_utils
```

---

### 2) Utilisation de l’API REST CouchDB
CouchDB s’appuie sur une API REST : on interagit via **GET / PUT / POST / DELETE** (ex. avec `curl` ou Postman).

#### a) Vérifier l’accès au serveur
```bash
curl -X GET http://admin:admin@localhost:5984
```

#### b) Créer une base de données `films`
```bash
curl -X PUT http://admin:admin@localhost:5984/films
```

#### c) Vérifier l’existence de la base
```bash
curl -X GET http://admin:admin@localhost:5984/films
```

#### d) Insérer plusieurs documents (bulk)
On peut charger un fichier JSON (ex : `films_couchdb.json`) avec l’opération `_bulk_docs` :
```bash
curl -X POST http://admin:admin@localhost:5984/films/_bulk_docs \
  -H "Content-Type: application/json" \
  --data-binary "@films_couchdb.json"
```

#### e) Récupérer un document par son identifiant
Exemple (film identifié `movie:10098`) :
```bash
curl -X GET http://admin:admin@localhost:5984/films/movie:10098
```

---

## Partie 2 – MapReduce avec CouchDB

### 1) Rappel : principe MapReduce
- **Map** : appliquée à chaque document, produit des paires *(clé, valeur)* via `emit(key, value)`.
- **Sort / Shuffle** : regroupement des résultats par clé.
- **Reduce** : agrégation des valeurs associées à chaque clé.

Dans CouchDB, les vues MapReduce sont définies en JavaScript et sont **persistantes** : l’index de la vue est mis à jour automatiquement quand les documents changent.

---

### 2) Exemple de vue : (année → titres)
#### a) Fonction Map
Objectif : récupérer l’année et le titre.
```js
function (doc) {
  emit(doc.year, doc.title);
}
```

#### b) Fonction Reduce
Objectif : compter le nombre de titres par année.
```js
function (keys, values) {
  return values.length;
}
```

---

### 3) Exemple : nombre de films par acteur
Objectif : pour chaque acteur, compter le nombre de films.

#### a) Map
```js
function (doc) {
  for (var i = 0; i < doc.actors.length; i++) {
    emit(
      { prenom: doc.actors[i].first_name, nom: doc.actors[i].last_name },
      doc.title
    );
  }
}
```

#### b) Reduce
```js
function (keys, values) {
  return values.length;
}
```

---

## Partie 3 – Exercices MapReduce avec MongoDB (1 → 9)

### Commande générale
```js
db.films.mapReduce(mapFunction, reduceFunction, {
  out: "nom_sortie"
});
```

> Remarque : selon les jeux de données, la structure des notes peut être `grades.score` ou `grades.note`. Les exercices ci-dessous utilisent `score` (adapter si nécessaire).

---

### 1) Compter le nombre total de films
```js
var mapTotalFilms = function () {
  emit("total", 1);
};

var reduceTotalFilms = function (key, values) {
  return Array.sum(values);
};
```

---

### 2) Compter le nombre de films par genre
```js
var mapFilmsParGenre = function () {
  if (this.genre) {
    // si genre est un tableau
    if (Array.isArray(this.genre)) {
      this.genre.forEach(g => emit(g, 1));
    } else {
      emit(this.genre, 1);
    }
  }
};

var reduceFilmsParGenre = function (key, values) {
  return Array.sum(values);
};
```

---

### 3) Compter le nombre de films par réalisateur
```js
var mapFilmsParRealisateur = function () {
  if (this.director) {
    // selon le modèle : director peut être un objet ou une chaîne
    if (typeof this.director === "object" && this.director.last_name) {
      emit(this.director.last_name, 1);
    } else {
      emit(this.director, 1);
    }
  }
};

var reduceFilmsParRealisateur = function (key, values) {
  return Array.sum(values);
};
```

---

### 4) Compter le nombre d’acteurs uniques
```js
var mapActeursUniques = function () {
  if (this.actors) {
    this.actors.forEach(function (a) {
      // selon le modèle : acteur peut être un objet ou une chaîne
      if (typeof a === "object" && a.first_name && a.last_name) {
        emit(a.first_name + " " + a.last_name, 1);
      } else {
        emit(a, 1);
      }
    });
  }
};

var reduceActeursUniques = function (key, values) {
  // chaque acteur unique apparaît au moins une fois
  return 1;
};
```

---

### 5) Lister le nombre de films par année de sortie
```js
var mapFilmsParAnnee = function () {
  emit(this.year, 1);
};

var reduceFilmsParAnnee = function (key, values) {
  return Array.sum(values);
};
```

---

### 6) Calculer la note moyenne par film
```js
var mapNoteMoyenneParFilm = function () {
  if (this.grades && this.grades.length > 0) {
    var mean = this.grades.reduce((a, b) => a + b.score, 0) / this.grades.length;
    emit(this.title, mean);
  }
};

var reduceNoteMoyenneParFilm = function (key, values) {
  return Array.sum(values) / values.length;
};
```

---

### 7) Calculer la note moyenne par genre
```js
var mapNoteMoyenneParGenre = function () {
  if (this.grades && this.grades.length > 0 && this.genre) {
    var avg = this.grades.reduce((a, b) => a + b.score, 0) / this.grades.length;

    if (Array.isArray(this.genre)) {
      this.genre.forEach(g => emit(g, avg));
    } else {
      emit(this.genre, avg);
    }
  }
};

var reduceNoteMoyenneParGenre = function (key, values) {
  return Array.sum(values) / values.length;
};
```

---

### 8) Calculer la note moyenne par réalisateur
```js
var mapNoteMoyenneParRealisateur = function () {
  if (this.director && this.grades && this.grades.length > 0) {
    var avg = this.grades.reduce((a, b) => a + b.score, 0) / this.grades.length;

    if (typeof this.director === "object" && this.director.last_name) {
      emit(this.director.last_name, avg);
    } else {
      emit(this.director, avg);
    }
  }
};

var reduceNoteMoyenneParRealisateur = function (key, values) {
  return Array.sum(values) / values.length;
};
```

---

### 9) Trouver le film avec la note maximale
```js
var mapFilmNoteMax = function () {
  if (this.grades && this.grades.length > 0) {
    var max = Math.max(...this.grades.map(g => g.score));
    emit(this.title, max);
  }
};

var reduceFilmNoteMax = function (key, values) {
  return Math.max(...values);
};
```

---

## Conclusion
Ce TP a permis de manipuler MapReduce dans deux environnements NoSQL :
- **CouchDB**, où les vues MapReduce constituent un mécanisme central d’interrogation (vues persistantes),
- **MongoDB**, où MapReduce permet de réaliser des agrégations distribuées via des fonctions `map` et `reduce`.

Les exercices ont illustré des cas classiques : comptages, regroupements, calculs de moyennes et recherche de maximum.

---

**Fin du TP4**

