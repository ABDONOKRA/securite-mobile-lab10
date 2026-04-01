# securite-mobile-lab10


<img width="958" height="166" alt="image" src="https://github.com/user-attachments/assets/76d76a32-e0e4-48f7-9338-23009daeb92e" />  
1.2. Installer frida et frida-tools
# Commande générique (essayez d’abord)
pip install --upgrade frida frida-tools  
<img width="1624" height="337" alt="image" src="https://github.com/user-attachments/assets/af011386-f36b-474c-9868-060ae8269107" />

  # Vérifications CLI:
  <img width="1798" height="842" alt="image" src="https://github.com/user-attachments/assets/3e031a22-6cdf-4302-8201-152cdd29f052" />  
  # Étape 2 — Installation des outils Android (optionnel mais recommandé)

  <img width="912" height="265" alt="image" src="https://github.com/user-attachments/assets/2e3dc82c-f393-4c6b-95fa-b5d6eea00d1c" />  

  # 3.1. Identifier l’architecture CPU de l’appareil  
  
  <img width="1356" height="72" alt="image" src="https://github.com/user-attachments/assets/0c689529-2485-451b-8365-5a650a48f026" />  
  # 3.4. Copier frida-server vers l’appareil Android
Le binaire doit être transféré vers un répertoire accessible en écriture sur l’appareil, par exemple /data/local/tmp.
adb push frida-server /data/local/tmp/
 # 3.6. Lancer frida-server
Un premier test peut être réalisé en lançant frida-server au premier plan :
adb shell /data/local/tmp/frida-server -l 0.0.0.0
Dans certains cas, il est préférable de le lancer en arrière-plan :
adb shell "nohup /data/local/tmp/frida-server -l 0.0.0.0 >/dev/null 2>&1 &"
Une autre méthode consiste à ouvrir un shell interactif :
adb shell
cd /data/local/tmp
./frida-server >/dev/null 2>&1 &
exit
# 3.7. Vérifier que frida-server est bien actif
Pour vérifier que le serveur est bien en cours d’exécution sur l’appareil :
adb shell ps | grep frida
Si grep n’est pas disponible dans le shell Android, il est possible d’utiliser simplement :
adb shell ps  
<img width="801" height="812" alt="image" src="https://github.com/user-attachments/assets/c9958de6-8db2-48f0-a737-ff928a985911" />
# Étape 4 — Test de connexion depuis le PC  

frida-ps -U           # liste des processus USB
frida-ps -Uai         # liste apps (i = include icons/pkg info)  

  <img width="893" height="432" alt="image" src="https://github.com/user-attachments/assets/2d61c569-4bb7-486f-819c-d2d3d0222fb2" />

# Étape 5 — Injection minimale pour valider  
<img width="704" height="250" alt="image" src="https://github.com/user-attachments/assets/3301c36c-9169-45c6-afa1-4d0b1786726d" />      

Java.perform(function () {
  console.log("[+] Frida Java.perform OK");
});  

Java.perform(function() { ... })
C'est la fonction principale de Frida pour interagir avec Android.

Elle dit à Frida : "attends que la VM Java (Dalvik/ART) soit prête, puis exécute ce code"
Tout code qui manipule des classes Java doit être à l'intérieur de Java.perform
Sans ça, tu aurais des erreurs car la VM n'est pas encore initialisée  
# frida -U -f com.Gestionetudiant.app -l hello.js
<img width="912" height="465" alt="image" src="https://github.com/user-attachments/assets/a8707030-c54a-4272-871e-fccd9bd5375b" />  
<img width="750" height="290" alt="image" src="https://github.com/user-attachments/assets/1cf1044b-b5a6-4d58-a6aa-aeb7fe118576" />    

console.log("[+] Script chargé");

Interceptor.attach(Module.getExportByName(null, "recv"), {
  onEnter(args) {
    console.log("[+] recv appelée");
  }
});

  Ce script :
affiche un message dès son chargement ;
recherche l’adresse de la fonction native recv ;
intercepte chaque appel à cette fonction ;
affiche un message lorsqu’une réception de données réseau est détectée.

# Étape 6 - Explorer la console interactive Frida dans un contexte de sécurité

<img width="944" height="477" alt="image" src="https://github.com/user-attachments/assets/f833027a-2e6e-4f1a-8d8f-3bf4b9c3e52b" />

# 6.1 Vérifier l’architecture du processus

Commande :  

Process.arch  

Rôle :  

Cette commande affiche l’architecture du processus, par exemple "x64" ou "arm64". Cette information est importante pour comprendre le contexte d’exécution, choisir les bons outils d’analyse et interpréter correctement les bibliothèques natives chargées.

<img width="944" height="154" alt="image" src="https://github.com/user-attachments/assets/3e7763c4-0807-4034-8027-de07db6120f2" />

  # 6.2 Identifier le module principal de l’application
<img width="772" height="228" alt="image" src="https://github.com/user-attachments/assets/4fc50389-fbaf-4799-82fe-ded2f5e5369b" />

Commande :  

Process.mainModule  

Rôle :  

Cette commande retourne les informations sur le module principal du processus. Dans une perspective sécurité, elle permet de situer le point d’entrée natif principal et de distinguer le code applicatif du code système.


# 6.3 Inspecter une bibliothèque système critique
Commande :  

Process.getModuleByName("libc.so")

<img width="806" height="774" alt="image" src="https://github.com/user-attachments/assets/ea827e14-a619-412c-8ef5-df96fa9fb89f" />
Rôle :  

Cette commande affiche les informations du module libc.so, notamment son adresse de base, son chemin et sa taille. En sécurité applicative, libc.so est particulièrement importante car elle contient de nombreuses fonctions natives liées aux fichiers, au réseau, à la mémoire et aux processus.
# 6.4 Vérifier la présence d’une fonction sensible
Commande :  

Process.getModuleByName("libc.so").getExportByName("recv")  

Rôle :  

Cette commande retourne l’adresse mémoire de recv, une fonction liée à la réception de données réseau. Cette vérification permet de confirmer que la fonction ciblée peut être observée dans le processus.

<img width="794" height="832" alt="image" src="https://github.com/user-attachments/assets/07887508-9ac1-4cd5-8595-33cdd8d51c7e" />

# 6.5 Lister les bibliothèques chargées
Rôle :
Cette commande liste toutes les bibliothèques chargées dans le processus.
Intérêt sécurité :
repérer la présence de bibliothèques natives propres à l’application ;
identifier des composants liés à la cryptographie, au réseau ou au WebView ;
vérifier si l’application s’appuie sur des librairies tierces qui méritent une attention particulière.
Exemples de bibliothèques intéressantes à rechercher :
libssl.so
libcrypto.so
libc.so
bibliothèques propres à l’application, par exemple libnative-lib.so
# 6.6 Lister les threads actifs

<img width="806" height="529" alt="image" src="https://github.com/user-attachments/assets/d64d48a9-430a-4754-84fb-b05238fd4515" />
# 6.8 Vérifier si l’environnement Java est disponible
Commande :
Java.available
Rôle :
Cette commande indique si le runtime Java est disponible dans le processus.
Intérêt sécurité :
si le résultat est true, il est possible d’observer des classes Java ;
cela permet de compléter l’analyse native par une inspection côté Java. 
<img width="768" height="168" alt="image" src="https://github.com/user-attachments/assets/21bb031e-8aee-424e-ab9c-7ee50a088379" />
# 6.9 Énumérer quelques classes Java chargées  
Commande :
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
Rôle :
Cette commande énumère les classes Java chargées et filtre celles liées à l’application.
Intérêt sécurité :
repérer les classes métier ;
identifier les composants d’authentification, de réseau ou de stockage ;
mieux comprendre la surface fonctionnelle de l’application.  

<img width="970" height="376" alt="image" src="https://github.com/user-attachments/assets/cb4131f2-3290-43cd-befe-08d9f33c1977" />

# 6.10 Vérifier la présence de bibliothèques liées au chiffrement ou au TLS
Commande :
Process.enumerateModules().filter(m => m.name.indexOf("ssl") !== -1 || m.name.indexOf("crypto") !== -1)
Rôle :
Cette commande filtre les modules chargés pour afficher ceux liés à SSL ou à la cryptographie.
Intérêt sécurité :
confirmer si l’application utilise des bibliothèques de chiffrement natives ;
orienter l’analyse vers les composants manipulant des données sensibles ;
préparer l’étude des flux réseau sécurisés.
<img width="1920" height="784" alt="image" src="https://github.com/user-attachments/assets/8154d1c1-9010-4b75-bd38-6d67ba46ac35" />
# 6.11 Observer les accès réseau de manière pédagogique  
Si un script de hook a déjà été injecté sur recv, send ou connect, la console Frida permet d’observer les appels effectués lorsque l’application communique.
Exemple de script déjà chargé :  
console.log("[+] Script chargé");

const recvPtr = Module.getGlobalExportByName("recv");
console.log("[+] recv trouvée à : " + recvPtr);

Interceptor.attach(recvPtr, {
  onEnter(args) {
    console.log("[+] recv appelée");
  }
});  
Rôle :
Ce test permet de confirmer qu’une activité réseau native existe dans le processus.
Intérêt sécurité :
observer les phases de communication ;
détecter si l’application appelle directement des fonctions natives réseau ;
préparer ensuite une analyse plus détaillée du trafic au niveau applicatif.
<img width="946" height="373" alt="image" src="https://github.com/user-attachments/assets/3c8927cc-e135-4a68-a24b-613bf8b7aac5" />

# 6.12 Observer les accès au système de fichiers  
Dans un contexte d’analyse de sécurité, il est également utile de vérifier si l’application ouvre des fichiers sensibles.
Exemple d’idée de test :
surveiller les fonctions open et read ;
observer si l’application accède à des fichiers de configuration, certificats, préférences locales ou bases SQLite.
Intérêt sécurité :
comprendre comment l’application stocke ou lit ses données ;
repérer les emplacements potentiellement sensibles ;
relier le comportement natif au stockage local.
# 6.13 Vérifier les informations de base sur le processus

ommandes :
Process.id
Process.platform
Rôle :
Process.id retourne l’identifiant du processus ;
Process.platform indique la plateforme d’exécution.
Intérêt sécurité :
documenter précisément l’environnement testé ;
faciliter la traçabilité dans un rapport de lab.  
<img width="644" height="153" alt="image" src="https://github.com/user-attachments/assets/1ae30307-925e-4696-a648-43b4345bc512" />
# 6.14 Quitter proprement la session
la commande :  
exit
## Résumé des commandes utiles en analyse de sécurité
Process.arch
Affiche l’architecture du processus.
Process.mainModule
Affiche le module principal.
Process.getModuleByName("libc.so")
Affiche les informations de libc.so.
Process.getModuleByName("libc.so").getExportByName("recv")
Retourne l’adresse de la fonction recv.
Process.enumerateModules()
Liste les modules chargés.
Process.enumerateThreads()
Liste les threads actifs.
Process.enumerateRanges('r-x')
Liste les zones mémoire exécutables.
Java.available
Vérifie la disponibilité du runtime Java.
Process.id
Process.platform
Affiche l’identifiant et la plateforme du processus.


# Étape 7 - Observer les bibliothèques de chiffrement, le stockage local et les appels réseau sensibles
# 7.1 Repérer les bibliothèques liées au chiffrement

Dans la console Frida, exécuter la commande suivante :
Process.enumerateModules().filter(m =>
  m.name.indexOf("ssl") !== -1 ||
  m.name.indexOf("crypto") !== -1 ||
  m.name.indexOf("boring") !== -1
)
Rôle :
Cette commande filtre les modules chargés dans le processus afin d’afficher les bibliothèques potentiellement liées au chiffrement ou au protocole TLS.
Intérêt sécurité :
identifier si l’application utilise des composants natifs liés à SSL/TLS ;
repérer l’usage de bibliothèques cryptographiques ;
préparer une analyse plus poussée des communications sécurisées.


<img width="944" height="850" alt="image" src="https://github.com/user-attachments/assets/29514d71-c903-419d-8ff1-de783e043504" />
# 7.2 Vérifier la présence d’une fonction réseau sensible

<img width="1190" height="130" alt="image" src="https://github.com/user-attachments/assets/9f46bfce-00a8-4e0c-bb75-0b532bf86754" />

# 7.3 Installer un hook sur connect
## Créer un fichier hook_connect.js :


<img width="896" height="478" alt="image" src="https://github.com/user-attachments/assets/e918f9df-fa13-49f0-a443-4513989b4660" />
<img width="1170" height="468" alt="image" src="https://github.com/user-attachments/assets/db2fb681-3374-4758-8219-6a7ac9d1299f" />
# 7.4 Installer un hook sur send et recv

Créer un fichier hook_network.js :
<img width="1070" height="760" alt="image" src="https://github.com/user-attachments/assets/df0d0873-383d-4891-aef7-03e6e3311a8e" />
<img width="1084" height="441" alt="image" src="https://github.com/user-attachments/assets/4bb53b1d-0cbf-432b-afb5-c12da6a19f81" />
frida -U -n "app1" -l hook_network.js
Rôle :
Ce script permet d’observer l’envoi et la réception de données dans le processus.
Intérêt sécurité :
détecter l’existence d’une activité réseau native ;
relier les opérations réseau à certaines actions dans l’application ;
mieux comprendre le comportement de communication du programme.
# 7.5 Observer les accès au système de fichiers
Créer un fichier hook_file.js :
<img width="1132" height="672" alt="image" src="https://github.com/user-attachments/assets/9ab9ebdc-ac1e-4573-8423-5a0315db2d66" />  
<img width="1132" height="948" alt="image" src="https://github.com/user-attachments/assets/c5adae14-2ca9-4d9e-abb8-8aa58543da1c" />

frida -U -n "Gestionetudiant" -l hook_file.js
Rôle :
Ce script permet d’observer les ouvertures de fichiers et certaines lectures effectuées par l’application.
Intérêt sécurité :
repérer l’accès à des fichiers internes ;
identifier la lecture de données locales, de préférences ou de ressources sensibles ;
relier le comportement natif à des mécanismes de stockage.

# 7.6 Vérifier la disponibilité du runtime Java
Commande :
Java.available
Rôle :
Cette commande indique si le processus dispose d’un environnement Java accessible par Frida.
Intérêt sécurité :
déterminer s’il est possible d’étendre l’analyse au niveau Java ;
préparer l’exploration des classes liées à l’authentification, au stockage ou au réseau.  

<img width="682" height="129" alt="image" src="https://github.com/user-attachments/assets/a9609b52-ac54-4f68-905b-31dd5dfe62b2" />

# 7.7 Rechercher des classes Java liées au stockage ou à la sécurité  
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
Rôle :
Cette commande aide à repérer des classes Java dont le nom suggère un lien avec la sécurité, la cryptographie ou le stockage.
Intérêt sécurité :
localiser des composants applicatifs sensibles ;
préparer le hooking de méthodes Java ;
mieux cibler l’analyse dynamique.
<img width="905" height="874" alt="image" src="https://github.com/user-attachments/assets/09548f77-d07b-4fa3-b366-a1ccf3b9dc96" />  
<img width="905" height="874" alt="image" src="https://github.com/user-attachments/assets/27186268-2f60-4cc7-b2e3-92d1e99831ca" />  
<img width="905" height="874" alt="image" src="https://github.com/user-attachments/assets/97e7600a-f0e9-4d93-828b-4b8112c43bae" />
# 7.8 Vérifier les zones mémoire exécutables
Commande :
Process.enumerateRanges('r-x')
Rôle :
Cette commande liste les plages mémoire marquées comme lisibles et exécutables.
Intérêt sécurité :
observer l’organisation mémoire du processus ;
identifier les régions susceptibles de contenir du code natif ;
compléter la compréhension du chargement des bibliothèques.










