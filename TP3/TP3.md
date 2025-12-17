# TP n°3 – NoSQL
## Partitionnement (Sharding) sous MongoDB

---

## Informations générales
- **Module** : Bases de données NoSQL  
- **TP** : TP3 – Partitionnement (Sharding)  
- **Étudiant** : **Benmessaoud Abdellah**

---

## Objectif du TP
L’objectif de ce TP est de mettre en place un cluster MongoDB shardé, d’activer le sharding sur une base de données et une collection, puis d’analyser le fonctionnement du système ainsi que les concepts clés liés au partitionnement des données.

---

## Architecture d’un cluster MongoDB shardé
Un cluster MongoDB shardé repose sur trois composants principaux :
- **Shards** : serveurs qui stockent les données, généralement organisés sous forme de replica sets afin d’assurer la tolérance aux pannes.
- **Config Servers (CSRS)** : stockent les métadonnées du cluster, notamment la localisation des chunks et la configuration globale.
- **Routeur mongos** : point d’entrée pour les clients, il redirige les requêtes vers les shards appropriés et rassemble les résultats.

---

## Mise en place du sharding

### Démarrage des composants
La mise en place du sharding nécessite le lancement de plusieurs instances MongoDB :
- un serveur de configuration ;
- deux shards ;
- un routeur mongos.

Chaque instance est exécutée sur un port différent afin de simuler un environnement distribué.

---

### Initialisation des Replica Sets
Chaque serveur est initialisé sous forme de replica set afin d’assurer la cohérence et la tolérance aux pannes.

```js
rs.initiate()
```

---

### Ajout des shards au cluster
Une fois les replica sets initialisés, les shards sont ajoutés au cluster à partir du routeur mongos.

```js
sh.addShard("replicashard1/localhost:20004")
sh.addShard("replicashard2/localhost:20005")
```

---

## Activation du sharding

### Activation sur la base de données
```js
sh.enableSharding("mabasefilms")
```

### Sharding d’une collection
```js
sh.shardCollection("mabasefilms.films", { "titre": 1 })
```

La clé de sharding choisie permet de répartir les documents sur les différents shards.

---

## Vérification du cluster shardé
Pour vérifier l’état du cluster et la distribution des données, on utilise la commande suivante :

```js
sh.status()
```

Cette commande affiche la liste des shards, les bases de données shardées ainsi que la répartition des chunks.

---

## Questions / Réponses

### 1. Qu’est-ce que le sharding dans MongoDB et pourquoi est-il utilisé ?
Le sharding est une technique de partitionnement horizontal des données. Il consiste à répartir une collection sur plusieurs shards afin de gérer de grands volumes de données et d’améliorer la scalabilité et les performances.

### 2. Quelle est la différence entre le sharding et la réplication ?
La réplication permet de dupliquer les données pour assurer la haute disponibilité, tandis que le sharding répartit les données afin de gérer la croissance du volume et de la charge.

### 3. Quels sont les composants d’une architecture shardée ?
Une architecture shardée est composée des shards, des config servers et du routeur mongos.

### 4. Quelles sont les responsabilités des config servers ?
Les config servers stockent les métadonnées du cluster et permettent à MongoDB de savoir où se trouvent les données.

### 5. Quel est le rôle du routeur mongos ?
Le mongos reçoit les requêtes des clients, les redirige vers les shards concernés et rassemble les résultats.

### 6. Comment MongoDB décide-t-il sur quel shard stocker un document ?
MongoDB utilise la clé de sharding pour déterminer le shard sur lequel le document sera stocké.

### 7. Qu’est-ce qu’une clé de sharding et pourquoi est-elle essentielle ?
La clé de sharding est un champ utilisé pour répartir les documents. Elle est essentielle pour assurer une distribution équilibrée des données.

### 8. Quels sont les critères d’une bonne clé de sharding ?
Une bonne clé de sharding doit avoir une forte cardinalité, être stable et être fréquemment utilisée dans les requêtes.

### 9. Qu’est-ce qu’un chunk ?
Un chunk est une portion de données d’une collection shardée correspondant à une plage de valeurs de la clé de sharding.

### 10. Comment fonctionne le splitting des chunks ?
Lorsque la taille d’un chunk devient trop importante, MongoDB le divise automatiquement en plusieurs chunks plus petits.

### 11. Quel est le rôle du balancer ?
Le balancer répartit les chunks de manière équilibrée entre les shards afin d’éviter les déséquilibres.

### 12. Qu’est-ce qu’un hot shard ?
Un hot shard est un shard surchargé par un grand nombre de requêtes ou d’écritures.

---

## Conclusion
Ce TP a permis de comprendre le principe du partitionnement des données sous MongoDB. Le sharding est une solution efficace pour assurer la montée en charge horizontale et améliorer les performances. Le choix d’une clé de sharding adaptée est un élément essentiel pour éviter les déséquilibres et garantir le bon fonctionnement du cluster.

---

**Fin du TP**

