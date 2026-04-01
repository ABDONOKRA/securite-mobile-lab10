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


