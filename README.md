# LAB-12-Bypass-de-la-D-tection-de-Root-Android-avec-Medusa
# 🔓 LAB 12 — Bypass de la Détection de Root Android

> **Cours : Sécurité des Applications Mobiles**  
> **Outils : Frida · Medusa · ADB · Android Emulator**

---

## 📋 Objectif

Démontrer comment contourner les mécanismes de détection de root dans une application Android en utilisant deux approches d'instrumentation dynamique :

1. **Medusa** — framework d'instrumentation modulaire basé sur Frida
2. **Frida pur** — script JavaScript injecté directement via la CLI Frida

L'application cible utilisée est **OWASP UnCrackable Level 1** (`owasp.mstg.uncrackable1`).

---

## 🛠️ Prérequis

| Outil | Version utilisée |
|---|---|
| Python | 3.13.x |
| Frida | 17.9.1 |
| Frida-tools | 14.8.x |
| Medusa | dev |
| Android Emulator | API 29 (x86) |
| ADB | inclus dans Android Studio |

---

## ⚙️ Installation de l'environnement

### 1. Cloner Medusa

```bash
git clone https://github.com/Ch0pin/medusa.git
cd medusa
python -m pip install -r requirements.txt
```

### 2. Installer les dépendances manquantes

```bash
python -m pip install pyreadline3 cmd2 pyyaml pick frida frida-tools colorama requests
```

### 3. Démarrer l'émulateur Android

- Ouvrir **Android Studio → Device Manager**
- Lancer un AVD **x86** (API 27–29 recommandé)

### 4. Pousser et lancer le serveur Frida sur l'émulateur

```bash
# Vérifier l'architecture de l'émulateur
adb shell getprop ro.product.cpu.abi
# → x86

# Pousser le bon frida-server (x86, pas x86_64 !)
adb push frida-server-17.9.1-android-x86 /data/local/tmp/frida-server
adb shell chmod 755 /data/local/tmp/frida-server

# Lancer le serveur
adb shell
/data/local/tmp/frida-server &
```

> ⚠️ Bien s'assurer que l'architecture du `frida-server` correspond à celle de l'émulateur (`x86` ≠ `x86_64`).

### 5. Installer l'APK cible

```bash
adb install UnCrackable-Level1.apk
```

---

## 🚀 Méthode 1 — Bypass avec Medusa

```bash
cd medusa
python medusa.py -d usb -p owasp.mstg.uncrackable1
```

Dans la console Medusa :

```
# Sélectionner l'émulateur
Enter the index of the device to use: 3

# Sélectionner l'application
(emulator-5554) medusa➤ 0

# Charger le module de bypass root
(emulator-5554) medusa➤ use root_detection/universal_root_detection_bypass

# Compiler le script
Module list has been modified, do you want to recompile? (Y/n) Y

# Spawner l'application avec le bypass actif
(emulator-5554) medusa➤ run -f owasp.mstg.uncrackable1
```

**Résultat attendu dans la console :**

```
---------- LOADING ANTI ROOT DETECTION SCRIPT ------------------
Loaded 5041 classes!
Bypass return value for binary: su
Bypass test-keys check
Bypass return value for binary: Superuser.apk
```

---

## 🚀 Méthode 2 — Bypass avec Frida pur (Plan B)

```bash
frida -U -f owasp.mstg.uncrackable1 -l bypass_root.js
```

**Résultat attendu :**

```
[+] Build.TAGS -> release-keys
[+] Runtime.exec hooks installed
[+] Java bypass installed
[+] File.exists bypass for /sbin/su
[+] File.exists bypass for /system/bin/su
[+] File.exists bypass for /system/xbin/su
...
```

---

## ✅ Validation du bypass

| État | Comportement de l'app |
|---|---|
| ❌ Sans bypass | Popup **"Root detected! This is unacceptable. The app is now going to exit."** |
| ✅ Avec bypass | L'app s'ouvre normalement et affiche le champ **"Enter the Secret String"** |

### Captures d'écran

**Avant (Root detected) :**

> L'application détecte l'environnement rooté et affiche un popup bloquant.

**Après (Bypass actif) :**

> L'application s'ouvre normalement sans aucune alerte.

---

## 🔍 Mécanismes bypassés

| Vecteur de détection | Méthode de bypass |
|---|---|
| `Build.TAGS` contient `test-keys` | Retourne `release-keys` |
| `Runtime.exec("su")` | Bloque l'exécution et retourne une erreur |
| `File.exists("/system/bin/su")` | Retourne `false` |
| `File.exists("/system/app/Superuser.apk")` | Retourne `false` |
| RootBeer checks | Hooks sur les méthodes Java internes |

---

## 📁 Structure du projet

```
.
├── bypass_root.js          # Script Frida pur (Plan B)
├── README.md               # Ce fichier
└── screenshots/
    ├── root_detected.png   # Avant le bypass
    └── bypass_success.png  # Après le bypass
```

---

## 📚 Références

- [OWASP Mobile Security Testing Guide](https://owasp.org/www-project-mobile-security-testing-guide/)
- [Frida Documentation](https://frida.re/docs/home/)
- [Medusa — Ch0pin](https://github.com/Ch0pin/medusa)
- [UnCrackable Apps — OWASP](https://github.com/OWASP/owasp-mastg/tree/master/Crackmes)

---

## ⚠️ Avertissement légal

> Ce projet est réalisé dans un cadre **éducatif uniquement**.  
> L'utilisation de ces techniques sur des applications sans autorisation explicite est **illégale**.  
> Les auteurs déclinent toute responsabilité en cas d'utilisation abusive.