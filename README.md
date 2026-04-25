Instrumentation Dynamique avec Frida

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
> ![Versions Frida](Screenshots/Screenshot_2026-04-25_164355.png)

### 1.2 Validation de l'API Python
Vérification que la bibliothèque Python `frida` est synchronisée avec les outils CLI.
* **Commande :** `py -c "import frida; print(frida.__version__)"`
* **Résultat :** `17.9.1`

> **Capture d'écran :**
> ![Version Python Frida](Screenshots/Screenshot_2026-04-25_171433.png)

### 1.3 État du périphérique Android (ADB)
Vérification de la connexion avec l'émulateur.
* **Device ID :** `emulator-5556`

> **Capture d'écran :**
> ![ADB Devices](Screenshots/Screenshot_2026-04-25_164431.png)

---

## 2. Déploiement sur Android

### 2.1 Lancement du serveur Frida
Le binaire `frida-server` est déployé dans `/data/local/tmp/`. Il est lancé avec l'option `-l 0.0.0.0` pour permettre l'écoute sur toutes les interfaces.

* **Commande :** `adb shell "/data/local/tmp/frida-server -l 0.0.0.0 &"`

### 2.2 Liste des applications (frida-ps -Uai)
Extraction des processus et des identifiants de packages installés sur le périphérique.

> **Capture d'écran :**
> ![Liste des Apps](Screenshots/Screenshot_2026-04-25_164413.png)

---

## 3. Injection de Script (Validation)

### 3.1 Script d'injection : `hello.js`
Le script suivant est utilisé pour confirmer que Frida peut interagir avec la VM Java d'Android :
```javascript
Java.perform(function () {
  console.log("[+] Frida Java.perform OK");
});
