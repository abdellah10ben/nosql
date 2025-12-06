# Comprendre simplement un Replica Set MongoDB
------------------------------------------------------------------------

## 🔹 1. Pourquoi utiliser un Replica Set ?

Un *Replica Set* permet d'avoir plusieurs copies d'une même base de
données toujours synchronisées.
L'idée est simple : si un serveur tombe, un autre prend le relais sans
perte de données.
Grâce à ça, on obtient :

-   une meilleure disponibilité du service,
-   une protection contre la perte de données,
-   un système plus fiable et tolérant aux pannes.

Un Replica Set est composé d'un **PRIMARY**  et de
**SECONDARY**.

------------------------------------------------------------------------

## 🔹 2. Simuler plusieurs serveurs en local

Même avec une seule machine, on peut créer plusieurs serveurs MongoDB :

-   chaque instance utilise un **port différent** (ex : 27018, 27019,
    27020),
-   chaque instance a son **propre dossier de données** (disk1, disk2,
    disk3),
-   toutes les instances partagent le **même nom de Replica Set** 

C'est pratique pour s'entraîner comme si on avait une vraie petite
infrastructure.

------------------------------------------------------------------------

## 🔹 3. Lancer les serveurs

Chaque serveur MongoDB se lance avec une commande comme :

    mongod --replSet monReplica --port PORT --dbpath DOSSIER


------------------------------------------------------------------------

## 🔹 4. Initialiser le Replica Set

Dans un nouveau terminal, on se connecte au premier serveur :

    mongo --port 27018

Puis on lance l'initialisation :

    rs.initiate()

Ensuite on ajoute les autres serveurs :

    rs.add("localhost:27019")
    rs.add("localhost:27020")

À partir de là, MongoDB s'occupe tout seul de la réplication.

------------------------------------------------------------------------

## 🔹 5. Rôles : PRIMARY et SECONDARY

Une fois tout configuré :

-   un serveur devient **PRIMARY** → c'est lui qui accepte les
    écritures,
-   les autres deviennent **SECONDARY** → ils répliquent les données et
    peuvent servir aux lectures (si configuré).

Les SECONDARY sont aussi là pour prendre le relais si le PRIMARY tombe.

------------------------------------------------------------------------

## 🔹 6. Tester la résilience

Pour voir comment MongoDB gère les pannes :

1.  On insère une donnée dans le PRIMARY.
2.  On éteint le PRIMARY (CTRL+C).
3.  MongoDB élit automatiquement un **nouveau PRIMARY** parmi les
    SECONDARY.

