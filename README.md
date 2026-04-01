# 🔬 Lab 10 — Sécurité Mobile : Analyse dynamique avec Frida

> **Objectif** : Installer et configurer **Frida** sur un environnement Android, puis réaliser une série d'observations dynamiques (processus, modules, réseau, fichiers, classes Java) afin de mieux comprendre le comportement interne d'une application mobile.

---

## 📑 Table des matières

1. [Étape 1 — Pré-requis Python](#étape-1--pré-requis-python)
2. [Étape 2 — Installation de Frida et Frida-Tools](#étape-2--installation-de-frida-et-frida-tools)
3. [Étape 3 — Installation des outils Android](#étape-3--installation-des-outils-android)
4. [Étape 4 — Configuration de Frida-Server sur l'appareil](#étape-4--configuration-de-frida-server-sur-lappareil)
5. [Étape 5 — Test de connexion depuis le PC](#étape-5--test-de-connexion-depuis-le-pc)
6. [Étape 6 — Injection minimale pour valider](#étape-6--injection-minimale-pour-valider)
7. [Étape 7 — Explorer la console interactive Frida](#étape-7--explorer-la-console-interactive-frida)
8. [Étape 8 — Observer les bibliothèques de chiffrement, le stockage local et les appels réseau](#étape-8--observer-les-bibliothèques-de-chiffrement-le-stockage-local-et-les-appels-réseau)
9. [Résumé des commandes utiles](#-résumé-des-commandes-utiles-en-analyse-de-sécurité)

---

## Étape 1 — Pré-requis Python

Vérifier que Python et `pip` sont correctement installés :

<img width="958" height="166" alt="Vérification Python" src="https://github.com/user-attachments/assets/76d76a32-e0e4-48f7-9338-23009daeb92e" />

---

## Étape 2 — Installation de Frida et Frida-Tools

```bash
# Installation / mise à jour de Frida et ses outils CLI
pip install --upgrade frida frida-tools
```

<img width="1624" height="337" alt="Installation Frida" src="https://github.com/user-attachments/assets/af011386-f36b-474c-9868-060ae8269107" />

### Vérifications CLI

```bash
frida --version
frida-ps --help
```

<img width="1798" height="842" alt="Vérifications CLI" src="https://github.com/user-attachments/assets/3e031a22-6cdf-4302-8201-152cdd29f052" />

---

## Étape 3 — Installation des outils Android

*(Optionnel mais recommandé)*

<img width="912" height="265" alt="Outils Android" src="https://github.com/user-attachments/assets/2e3dc82c-f393-4c6b-95fa-b5d6eea00d1c" />

---

## Étape 4 — Configuration de Frida-Server sur l'appareil

### 4.1 Identifier l'architecture CPU de l'appareil

```bash
adb shell getprop ro.product.cpu.abi
```

<img width="1356" height="72" alt="Architecture CPU" src="https://github.com/user-attachments/assets/0c689529-2485-451b-8365-5a650a48f026" />

### 4.2 Copier frida-server vers l'appareil Android

Le binaire doit être transféré vers un répertoire accessible en écriture sur l'appareil, par exemple `/data/local/tmp` :

```bash
adb push frida-server /data/local/tmp/
```

### 4.3 Lancer frida-server

**Option 1 — Premier plan (test rapide) :**

```bash
adb shell /data/local/tmp/frida-server -l 0.0.0.0
```

**Option 2 — Arrière-plan :**

```bash
adb shell "nohup /data/local/tmp/frida-server -l 0.0.0.0 >/dev/null 2>&1 &"
```

**Option 3 — Shell interactif :**

```bash
adb shell
cd /data/local/tmp
./frida-server >/dev/null 2>&1 &
exit
```

### 4.4 Vérifier que frida-server est bien actif

```bash
adb shell ps | grep frida
# Si grep n'est pas disponible :
adb shell ps
```

<img width="801" height="812" alt="Frida-server actif" src="https://github.com/user-attachments/assets/c9958de6-8db2-48f0-a737-ff928a985911" />

---

## Étape 5 — Test de connexion depuis le PC

```bash
frida-ps -U           # liste des processus USB
frida-ps -Uai         # liste des apps installées (avec infos packages)
```

<img width="893" height="432" alt="Test de connexion" src="https://github.com/user-attachments/assets/2d61c569-4bb7-486f-819c-d2d3d0222fb2" />

---

## Étape 6 — Injection minimale pour valider

<img width="704" height="250" alt="Injection minimale" src="https://github.com/user-attachments/assets/3301c36c-9169-45c6-afa1-4d0b1786726d" />

### 6.1 Script `hello.js` — Test basique

```javascript
Java.perform(function () {
  console.log("[+] Frida Java.perform OK");
});
```

> **Explication :**
> - `Java.perform(function() { ... })` est la fonction principale de Frida pour interagir avec Android.
> - Elle attend que la VM Java (Dalvik/ART) soit prête, puis exécute le code.
> - Tout code manipulant des classes Java **doit** être à l'intérieur de `Java.perform`.
> - Sans cela, des erreurs surviendraient car la VM n'est pas encore initialisée.

#### Exécution :

```bash
frida -U -f com.Gestionetudiant.app -l hello.js
```

<img width="912" height="465" alt="Exécution hello.js" src="https://github.com/user-attachments/assets/a8707030-c54a-4272-871e-fccd9bd5375b" />
<img width="750" height="290" alt="Résultat hello.js" src="https://github.com/user-attachments/assets/1cf1044b-b5a2-4d58-a6aa-aeb7fe118576" />

### 6.2 Script d'interception réseau — `recv`

```javascript
console.log("[+] Script chargé");

Interceptor.attach(Module.getExportByName(null, "recv"), {
  onEnter(args) {
    console.log("[+] recv appelée");
  }
});
```

> **Ce script :**
> - Affiche un message dès son chargement.
> - Recherche l'adresse de la fonction native `recv`.
> - Intercepte chaque appel à cette fonction.
> - Affiche un message lorsqu'une réception de données réseau est détectée.

---

## Étape 7 — Explorer la console interactive Frida

<img width="944" height="477" alt="Console interactive" src="https://github.com/user-attachments/assets/f833027a-2e6e-4f1a-8d8f-3bf4b9c3e52b" />

### 7.1 Vérifier l'architecture du processus

```javascript
Process.arch
```

> Affiche l'architecture du processus (ex. `"x64"`, `"arm64"`). Important pour choisir les bons outils d'analyse et interpréter les bibliothèques natives.

<img width="944" height="154" alt="Process.arch" src="https://github.com/user-attachments/assets/3e7763c4-0807-4034-8027-de07db6120f2" />

### 7.2 Identifier le module principal de l'application

```javascript
Process.mainModule
```

> Retourne les informations sur le module principal du processus. Permet de situer le point d'entrée natif principal et de distinguer le code applicatif du code système.

<img width="772" height="228" alt="Process.mainModule" src="https://github.com/user-attachments/assets/4fc50389-fbaf-4799-82fe-ded2f5e5369b" />

### 7.3 Inspecter une bibliothèque système critique

```javascript
Process.getModuleByName("libc.so")
```

> Affiche les informations de `libc.so` (adresse de base, chemin, taille). Cette bibliothèque contient des fonctions natives liées aux fichiers, au réseau, à la mémoire et aux processus.

<img width="806" height="774" alt="libc.so" src="https://github.com/user-attachments/assets/ea827e14-a619-412c-8ef5-df96fa9fb89f" />

### 7.4 Vérifier la présence d'une fonction sensible

```javascript
Process.getModuleByName("libc.so").getExportByName("recv")
```

> Retourne l'adresse mémoire de `recv`, une fonction liée à la réception de données réseau. Permet de confirmer que la fonction ciblée peut être observée dans le processus.

<img width="794" height="832" alt="recv address" src="https://github.com/user-attachments/assets/07887508-9ac1-4cd5-8595-33cdd8d51c7e" />

### 7.5 Lister les bibliothèques chargées

```javascript
Process.enumerateModules()
```

> **Intérêt sécurité :**
> - Repérer la présence de bibliothèques natives propres à l'application.
> - Identifier des composants liés à la cryptographie, au réseau ou au WebView.
> - Vérifier si l'application utilise des librairies tierces sensibles.
>
> **Exemples de bibliothèques à rechercher :** `libssl.so`, `libcrypto.so`, `libc.so`, `libnative-lib.so`

### 7.6 Lister les threads actifs

```javascript
Process.enumerateThreads()
```

<img width="806" height="529" alt="Threads" src="https://github.com/user-attachments/assets/d64d48a9-430a-4754-84fb-b05238fd4515" />

### 7.7 Vérifier si l'environnement Java est disponible

```javascript
Java.available
```

> - Si le résultat est `true`, il est possible d'observer des classes Java.
> - Cela permet de compléter l'analyse native par une inspection côté Java.

<img width="768" height="168" alt="Java.available" src="https://github.com/user-attachments/assets/21bb031e-8aee-424e-ab9c-7ee50a088379" />

### 7.8 Énumérer les classes Java chargées

```javascript
Java.perform(function () {
  Java.enumerateLoadedClasses({
    onMatch: function (name) {
      if (name.indexOf("app1") !== -1) {
        console.log(name);
      }
    },
    onComplete: function () {
      console.log("Fin de l'énumération");
    }
  });
});
```

> **Intérêt sécurité :**
> - Repérer les classes métier.
> - Identifier les composants d'authentification, de réseau ou de stockage.
> - Mieux comprendre la surface fonctionnelle de l'application.

<img width="970" height="376" alt="Classes Java" src="https://github.com/user-attachments/assets/cb4131f2-3290-43cd-befe-08d9f33c1977" />

### 7.9 Vérifier les bibliothèques liées au chiffrement / TLS

```javascript
Process.enumerateModules().filter(m =>
  m.name.indexOf("ssl") !== -1 || m.name.indexOf("crypto") !== -1
)
```

> Confirme si l'application utilise des bibliothèques de chiffrement natives et oriente l'analyse vers les composants manipulant des données sensibles.

<img width="1920" height="784" alt="SSL/Crypto modules" src="https://github.com/user-attachments/assets/8154d1c1-9010-4b75-bd38-6d67ba46ac35" />

### 7.10 Observer les accès réseau

```javascript
console.log("[+] Script chargé");

const recvPtr = Module.getGlobalExportByName("recv");
console.log("[+] recv trouvée à : " + recvPtr);

Interceptor.attach(recvPtr, {
  onEnter(args) {
    console.log("[+] recv appelée");
  }
});
```

> Permet de confirmer qu'une activité réseau native existe dans le processus, d'observer les phases de communication et de détecter les appels directs à des fonctions réseau natives.

<img width="946" height="373" alt="Accès réseau" src="https://github.com/user-attachments/assets/3c8927cc-e135-4a68-a24b-613bf8b7aac5" />

### 7.11 Observer les accès au système de fichiers

> Dans un contexte de sécurité, il est utile de :
> - Surveiller les fonctions `open` et `read`.
> - Observer si l'application accède à des fichiers de configuration, certificats, préférences locales ou bases SQLite.
> - Repérer les emplacements potentiellement sensibles.

### 7.12 Vérifier les informations de base sur le processus

```javascript
Process.id        // Identifiant du processus
Process.platform  // Plateforme d'exécution
```

> Permet de documenter précisément l'environnement testé et de faciliter la traçabilité dans un rapport de lab.

<img width="644" height="153" alt="Process info" src="https://github.com/user-attachments/assets/1ae30307-925e-4696-a648-43b4345bc512" />

### 7.13 Quitter proprement la session

```
exit
```

---

## Étape 8 — Observer les bibliothèques de chiffrement, le stockage local et les appels réseau

### 8.1 Repérer les bibliothèques liées au chiffrement

```javascript
Process.enumerateModules().filter(m =>
  m.name.indexOf("ssl") !== -1 ||
  m.name.indexOf("crypto") !== -1 ||
  m.name.indexOf("boring") !== -1
)
```

> **Intérêt sécurité :**
> - Identifier l'usage de composants natifs liés à SSL/TLS.
> - Repérer les bibliothèques cryptographiques.
> - Préparer une analyse plus poussée des communications sécurisées.

<img width="944" height="850" alt="Crypto modules" src="https://github.com/user-attachments/assets/29514d71-c903-419d-8ff1-de783e043504" />

### 8.2 Vérifier la présence d'une fonction réseau sensible

<img width="1190" height="130" alt="Fonction réseau" src="https://github.com/user-attachments/assets/9f46bfce-00a8-4e0c-bb75-0b532bf86754" />

### 8.3 Installer un hook sur `connect`

Créer un fichier **`hook_connect.js`** :

<img width="896" height="478" alt="hook_connect.js" src="https://github.com/user-attachments/assets/e918f9df-fa13-49f0-a443-4513989b4660" />
<img width="1170" height="468" alt="Résultat hook_connect" src="https://github.com/user-attachments/assets/db2fb681-3374-4758-8219-6a7ac9d1299f" />

### 8.4 Installer un hook sur `send` et `recv`

Créer un fichier **`hook_network.js`** :

<img width="1070" height="760" alt="hook_network.js" src="https://github.com/user-attachments/assets/df0d0873-383d-4891-aef7-03e6e3311a8e" />
<img width="1084" height="441" alt="Résultat hook_network" src="https://github.com/user-attachments/assets/4bb53b1d-0cbf-432b-afb5-c12da6a19f81" />

#### Exécution :

```bash
frida -U -n "app1" -l hook_network.js
```

> **Intérêt sécurité :**
> - Détecter l'existence d'une activité réseau native.
> - Relier les opérations réseau à certaines actions dans l'application.
> - Mieux comprendre le comportement de communication du programme.

### 8.5 Observer les accès au système de fichiers

Créer un fichier **`hook_file.js`** :

<img width="1132" height="672" alt="hook_file.js" src="https://github.com/user-attachments/assets/9ab9ebdc-ac1e-4573-8423-5a0315db2d66" />
<img width="1132" height="948" alt="Résultat hook_file" src="https://github.com/user-attachments/assets/c5adae14-2ca9-4d9e-abb8-8aa58543da1c" />

#### Exécution :

```bash
frida -U -n "Gestionetudiant" -l hook_file.js
```

> **Intérêt sécurité :**
> - Repérer l'accès à des fichiers internes.
> - Identifier la lecture de données locales, de préférences ou de ressources sensibles.
> - Relier le comportement natif à des mécanismes de stockage.

### 8.6 Vérifier la disponibilité du runtime Java

```javascript
Java.available
```

> Détermine s'il est possible d'étendre l'analyse au niveau Java et de préparer l'exploration des classes liées à l'authentification, au stockage ou au réseau.

<img width="682" height="129" alt="Java available" src="https://github.com/user-attachments/assets/a9609b52-ac54-4f68-905b-31dd5dfe62b2" />

### 8.7 Rechercher des classes Java liées au stockage ou à la sécurité

```javascript
Java.perform(function () {
  Java.enumerateLoadedClasses({
    onMatch: function (name) {
      if (
        name.toLowerCase().indexOf("security") !== -1 ||
        name.toLowerCase().indexOf("crypto") !== -1 ||
        name.toLowerCase().indexOf("prefs") !== -1 ||
        name.toLowerCase().indexOf("sqlite") !== -1 ||
        name.toLowerCase().indexOf("storage") !== -1
      ) {
        console.log(name);
      }
    },
    onComplete: function () {
      console.log("Fin de l'énumération");
    }
  });
});
```

> **Intérêt sécurité :**
> - Localiser des composants applicatifs sensibles.
> - Préparer le hooking de méthodes Java.
> - Mieux cibler l'analyse dynamique.

<img width="905" height="874" alt="Classes sécurité 1" src="https://github.com/user-attachments/assets/09548f77-d07b-4fa3-b366-a1ccd3b9dc96" />
<img width="905" height="874" alt="Classes sécurité 2" src="https://github.com/user-attachments/assets/27186268-2f60-4cc7-b2e3-92d1e99831ca" />
<img width="905" height="874" alt="Classes sécurité 3" src="https://github.com/user-attachments/assets/97e7600a-f0e9-4d93-828b-4b8112c43bae" />

### 8.8 Vérifier les zones mémoire exécutables

```javascript
Process.enumerateRanges('r-x')
```

> **Intérêt sécurité :**
> - Observer l'organisation mémoire du processus.
> - Identifier les régions susceptibles de contenir du code natif.
> - Compléter la compréhension du chargement des bibliothèques.

---

## 📋 Résumé des commandes utiles en analyse de sécurité

| Commande | Rôle |
|---|---|
| `Process.arch` | Affiche l'architecture du processus |
| `Process.mainModule` | Affiche le module principal |
| `Process.getModuleByName("libc.so")` | Affiche les informations de `libc.so` |
| `Process.getModuleByName("libc.so").getExportByName("recv")` | Retourne l'adresse de la fonction `recv` |
| `Process.enumerateModules()` | Liste les modules chargés |
| `Process.enumerateThreads()` | Liste les threads actifs |
| `Process.enumerateRanges('r-x')` | Liste les zones mémoire exécutables |
| `Java.available` | Vérifie la disponibilité du runtime Java |
| `Process.id` | Affiche l'identifiant du processus |
| `Process.platform` | Affiche la plateforme du processus |
