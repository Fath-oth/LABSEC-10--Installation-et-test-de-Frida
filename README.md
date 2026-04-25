
**Auteur :** Othmane Fath  
**Sujet :** Installation, configuration et validation de l'environnement Frida sur Android.

---

## 1. Installation et Preuve de l'Environnement

Dans cette section, nous validons que les outils Frida sont correctement installés sur la machine hôte (Windows) et que le pont ADB est opérationnel.

### 1.1 Versions de Frida CLI
| Outil | Commande | Version détectée |
| :--- | :--- | :--- |
| Frida | `frida --version` | 17.9.1 |
| Frida-PS | `frida-ps --version` | 17.9.1 |

> **Capture d'écran :**
<img width="627" height="101" alt="Screenshot 2026-04-25 164355" src="https://github.com/user-attachments/assets/c1a92419-dd56-4d63-a545-34cf66a8d069" />


### 1.2 Validation de l'API Python
Vérification que la bibliothèque Python `frida` est synchronisée avec les outils CLI.
* **Commande :** `py -c "import frida; print(frida.__version__)"`
* **Résultat :** `17.9.1`

> **Capture d'écran :**

<img width="905" height="85" alt="Screenshot 2026-04-25 171433" src="https://github.com/user-attachments/assets/471d7abd-1462-4e35-878a-03ba323f4f45" />

### 1.3 État du périphérique Android (ADB)
Vérification de la connexion avec l'émulateur.
* **Device ID :** `emulator-5556`

> **Capture d'écran :**
<img width="443" height="100" alt="Screenshot 2026-04-25 164431" src="https://github.com/user-attachments/assets/af44b492-7678-4d67-848c-4b08a0da69e8" />

---

## 2. Déploiement sur Android

### 2.1 Lancement du serveur Frida
Le binaire `frida-server` est déployé dans `/data/local/tmp/`. Il est lancé avec l'option `-l 0.0.0.0` pour permettre l'écoute sur toutes les interfaces.

* **Commande :** `adb shell "/data/local/tmp/frida-server -l 0.0.0.0 &"`

### 2.2 Liste des applications (frida-ps -Uai)
Extraction des processus et des identifiants de packages installés sur le périphérique.

> **Capture d'écran :**

---<img width="892" height="513" alt="Screenshot 2026-04-25 164413" src="https://github.com/user-attachments/assets/455bfe55-f473-4151-bf58-1b4f349c924b" />


## 3. Injection de Script (Validation)

### 3.1 Script d'injection : `hello.js`
Le script suivant est utilisé pour confirmer que Frida peut interagir avec la VM Java d'Android :
```javascript
Java.perform(function () {
  console.log("[+] Frida Java.perform OK");
});
````
<img width="1089" height="404" alt="Screenshot 2026-04-25 172448" src="https://github.com/user-attachments/assets/62e756e0-0151-4063-8916-bc4a91d7df9d" />

## 4. Dépannage (Troubleshooting)

Cette section documente la résolution d'un incident technique simulé pour valider la compréhension du flux de communication entre l'hôte Windows et l'agent Frida sur Android.

### 4.1 Description de l'incident
Lors de la tentative d'énumération des processus via `frida-ps -U`, le terminal retourne l'erreur suivante :
`Failed to enumerate devices: express session closed` ou `device not found`.

### 4.2 Méthodologie de Diagnostic
Pour isoler la cause racine, la check-list suivante a été appliquée :

1.  **Vérification de la couche physique (ADB) :**
    * Commande : `adb devices`
    * *Résultat :* Le périphérique `emulator-5556` est bien listé. Le tunnel ADB est opérationnel.
2.  **Vérification de l'état du serveur distant :**
    * Commande : `adb shell "ps | grep frida"`
    * *Constat :* Aucun retour. Le binaire `frida-server` n'est pas en cours d'exécution sur le mobile.
3.  **Vérification des privilèges :**
    * Tentative de relance manuelle : `/data/local/tmp/frida-server`.
    * *Erreur :* `Permission denied`. Le serveur nécessite les droits root pour s'attacher aux processus système.

### 4.3 Résolution et Correction
Pour rétablir le service, les étapes de correction suivantes ont été effectuées :

* **Étape 1 :** Attribution des permissions d'exécution :
    ```powershell
    adb shell "chmod 755 /data/local/tmp/frida-server"
    ```
* **Étape 2 :** Lancement du serveur en arrière-plan avec les privilèges `root` :
    ```powershell
    adb shell "su -c '/data/local/tmp/frida-server &'"
    ```
* **Étape 3 :** Test de persistance :
    ```powershell
    frida-ps -U
    ```

### 4.4 Conclusion du diagnostic
L'erreur était due à un arrêt du processus démon sur le périphérique cible. La procédure de relance via `su -c` est indispensable sur Android pour permettre à Frida d'accéder à la mémoire des applications tierces.

---
*Livrable réalisé dans le cadre du module Sécurité Mobile - ENSA Marrakech.*
