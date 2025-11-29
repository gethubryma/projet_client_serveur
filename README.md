# Serveur clé–valeur TCP multithreadé (type mini-Redis)

## 1. Description du projet

Ce projet implémente un **petit serveur clé–valeur accessible via TCP**, inspiré du fonctionnement de Redis.

Un **serveur en C** qui :

* accepte plusieurs clients grâce aux **threads (pthread)**
* gère un petit stockage en mémoire (**tableau clé–valeur**)
* écrit les opérations dans un fichier de persistance **Cle_Valeur.data**

Un **client en C** permet d’envoyer des commandes au serveur.

### Fonctionnalités principales

* Stockage / lecture / suppression de clés
* Incrémentation / décrémentation de valeurs numériques
* Test d’existence de clé
* Echo de messages
* Ajout de texte à une valeur
* Manipulation de listes (RPUSH, LRANGE)
* Renommage de clé
* Déconnexion propre du client

---

## 2. Contenu du projet

### 2.1. Fichiers présents

* **server.c** — sockets, threads, traitement des commandes
* **server.h** — définitions des structures (Entry, Context)
* **client.c** — connexion, lecture/saisie, envoi des commandes
* **makefile** — compilation (server, client, clean)
* **cle_valeur.data / Cle_Valeur.data** — journalisation des opérations

**Exemple de contenu :**

```
A 4
A 5
A 1
B 2
TY 34
TY 45
A 5
A 43
B 45
Y 1
B hello
A 67
```

---

## 3. Compilation du projet

### Étape 1 : Prérequis

* Compilateur C (ex : gcc)
* libc, pthread, sockets
* Système Unix/Linux ou WSL

### Étape 2 : Compilation

```bash
make
```

Le makefile utilise :

```makefile
CC = gcc
CFLAGS = -Wall -pthread
```

Nettoyage :

```bash
make clean
```

---

## 4. Lancement du serveur

### Étape 1 : Choisir un port

Ex : **5050** ou **8080**

### Étape 2 : Démarrer le serveur

```bash
./server <port>
```

Exemple :

```bash
./server 5050
```

Affichage :

```
le serveur est en ecoute sur le port 5050...
client connecté
```

---

## 5. Lancement du client

### Étape 1 : Connexion

```bash
./client <server_ip> <port>
```

Exemple local :

```bash
./client 127.0.0.1 5050
```

Autre machine :

```bash
./client 192.168.1.10 5050
```

### Étape 2 : Utilisation

Le client affiche :

```
>
```

Quitter :

* `QUIT` (envoyé au serveur)
* `exit` (ferme uniquement le client)

---

## 6. Commandes supportées (protocole texte)

### 6.1. PING / PONG

Commande :

```
PING
```

Réponse :

```
PONG
```

---

### 6.2. SET

Usage :

```
SET <key> <value>
```

Exemple :

```
SET A 10
OK
```

Effet : met à jour ou crée la clé + journalisation.

---

### 6.3. GET

```
GET <key>
```

Exemple :

```
GET A
10
```

---

### 6.4. DEL

```
DEL <key>
```

Exemple :

```
DEL A
OK
```

---

### 6.5. INCR

```
INCR <key>
```

Exemple :

```
INCR compteur
OK
```

---

### 6.6. DECR

```
DECR <key>
```

---

### 6.7. EXISTS

```
EXISTS <key>
```

Exemples :

```
EXISTS A
ok
EXISTS Z
no
```

---

### 6.8. ECHO

```
ECHO <message...>
```

---

### 6.9. APPEND

```
APPEND <key> <texte...>
```

---

### 6.10. RPUSH

```
RPUSH <key> <val1> <val2> ...
```

Exemple :

```
RPUSH liste 1 2 3 4
OK
```

---

### 6.11. LRANGE

```
LRANGE <key>
```

---

### 6.12. RENAME

```
RENAME <oldkey> <newkey>
```

---

### 6.13. QUIT / exit

```
QUIT
exit
```

---

## 7. Fonctionnement interne du serveur

### 7.1. Structures de données

#### Entry

```c
struct Entry {
    char key[MAX_KEY_SIZE];
    char value[MAX_VALUE_SIZE];
};
```

#### Context

```c
struct Context {
    pthread_mutex_t mutex;
    struct Entry entries[MAX_ENTRIES];
};
```

---

### 7.2. Concurrence (threads)

* un thread par client
* mutex pour protéger la table en mémoire

```c
pthread_mutex_lock(&c->mutex);
/* opérations */
pthread_mutex_unlock(&c->mutex);
```

---

### 7.3. Fichier de persistance

```c
FILE *file = fopen(FICHIER, "a");
if (file != NULL) {
    fprintf(file, "%s %s\n", key, value);
    fclose(file);
}
```

---

## 8. Exemple de session complète

Serveur :

```bash
./server 5050
le serveur est en ecoute sur le port 5050...
client connecté
```

Client :

```
> PING
PONG
> SET A 10
OK
> GET A
10
> INCR A
OK
> GET A
11
> EXISTS A
ok
> APPEND A _units
OK
> GET A
11 _units
> RPUSH liste 1 2 3 4
OK
> LRANGE liste
1 2 3 4
> RENAME A B
OK
> GET B
11 _units
> DEL B
OK
> GET B
Cle non existante
> QUIT
```

Serveur :

```
client connecté
Client déconnecté
```

---

## 9. Résumé d’utilisation

1. **Compiler**

```bash
make
```

2. **Lancer le serveur**

```bash
./server 5050
```

3. **Lancer les clients**

```bash
./client 127.0.0.1 5050
```

4. **Envoyer des commandes :**

PING, SET, GET, DEL, INCR, DECR, EXISTS, ECHO, APPEND, RPUSH, LRANGE, RENAME, QUIT.

5. **Observer :**

* réponses côté client
* journalisation dans **Cle_Valeur.data**

---

README.md prêt à copier-coller.
