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
<img width="944" height="477" alt="image" src="https://github.com/user-attachments/assets/f833027a-2e6e-4f1a-8d8f-3bf4b9c3e52b" />



