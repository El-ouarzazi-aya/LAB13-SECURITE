# LAB13-SECURITE
#  Lab — Bypass Root Detection avec Frida & Objection

> **Objectif :** Contourner la détection root d'une application Android en utilisant Frida et Objection, et identifier les appels natifs avec frida-trace.

---

##  Environnement

| Composant | Version |
|---|---|
| OS | Windows 10 |
| Python | 3.11.9 |
| Frida | 17.9.1 |
| Objection | 1.12.4 |
| Appareil | Émulateur AVD (Android 11) |
| App cible | RootBeer Sample (`com.scottyab.rootbeer.sample`) |

---

## Étape 1 — Vérification des prérequis

Vérification que Python, pip, ADB et Frida sont correctement installés et que l'émulateur est bien reconnu.

```bash
python --version
pip --version
adb devices
frida --version
```

<img width="860" height="139" alt="prerequis" src="https://github.com/user-attachments/assets/66733758-e99c-4235-8a26-a896610121c3" />


---

## Étape 2 — Installation d'Objection

Installation d'Objection via pip avec Python 3.11 pour éviter le problème de compatibilité `tomllib` (absent en Python 3.10).

```bash
py -3.11 -m pip install --upgrade objection
```

<img width="537" height="417" alt="objection-install" src="https://github.com/user-attachments/assets/ce4d401e-dd99-4063-83cc-795ee2c2fdb7" />


Vérification de l'installation :

```bash
objection --help
```

<img width="452" height="422" alt="objection-verification" src="https://github.com/user-attachments/assets/a90c6845-684e-4ba0-9c39-25d056c4e509" />


---

## Étape 3 — Connexion Frida à l'émulateur

Démarrage de `frida-server` sur l'émulateur et vérification de la visibilité des processus Android.

```bash
adb -s emulator-5556 shell "/data/local/tmp/frida-server &"
frida-ps -Uai
```

<img width="325" height="223" alt="frida" src="https://github.com/user-attachments/assets/4e241d07-29f7-40ea-8774-0e80d911ef54" />


---

## Étape 4 — Root detection (avant bypass)

Lancement de l'app **RootBeer Sample** sans instrumentation. L'app détecte l'environnement rooté et affiche `*ROOTED*`.

```bash
adb -s emulator-5556 shell am start -n com.scottyab.rootbeer.sample/.MainActivity
```

<img width="189" height="384" alt="root-detected" src="https://github.com/user-attachments/assets/ab75b97a-7da0-4edd-8462-de422ead2fb4" />


---

## Étape 5 — Bypass avec Objection

Lancement d'Objection en mode spawn avec la commande `android root disable` appliquée au démarrage. Objection installe des hooks Java qui neutralisent les méthodes de détection root de RootBeer :

- `RootBeer->detectRootCloakingApps()` → `false`
- `RootBeer->detectTestKeys()` → `false`
- `RootBeer->checkForBinary()` → `false`
- `RootBeer->checkSuExists()` → `false`
- `RootBeer->checkForDangerousProps()` → `false`
- `RootBeerNative->checkForRoot()` → `0`

```bash
objection --serial emulator-5556 -n com.scottyab.rootbeer.sample start \
  --startup-command "android root disable"
```

<img width="769" height="286" alt="script--objection" src="https://github.com/user-attachments/assets/e4f8b06d-89da-4610-aee0-604173eaf428" />


---

## Étape 6 — Résultat du bypass (après)

Après instrumentation, l'app affiche `NOT ROOTED` — tous les checks sont neutralisés.

<img width="186" height="362" alt="root-not-detected" src="https://github.com/user-attachments/assets/f14a781e-f5a2-424a-9990-03b4deea457c" />


---

## Étape 7 — Bonus : Identification des appels natifs avec frida-trace

Utilisation de `frida-trace` pour intercepter les appels natifs de `libc.so` en temps réel. Ces fonctions sont utilisées par l'app pour vérifier l'existence de binaires suspects (`/system/xbin/su`, `/sbin/su`, etc.) au niveau C/C++.

```bash
frida-trace -U -i "open" -i "access" -i "stat" -f com.scottyab.rootbeer.sample
```

Résultat : frida-trace génère automatiquement des handlers JS pour chaque fonction interceptée et affiche les appels en temps réel (`open()`, `stat()`, `access()`).

<img width="511" height="427" alt="frida-trace" src="https://github.com/user-attachments/assets/e31d97d1-aff3-45fc-8a4b-61078a8c84b8" />


---

##  Récapitulatif des points

| Exercice | Points | Résultat |
|---|---|---|
| Prérequis : Python, ADB, Frida, Objection | 20 pts | ✅ |
| Connexion Objection + invite session | 20 pts | ✅ |
| Bypass root avant/après (RootBeer) | 40 pts | ✅ |
| Bonus natif avec frida-trace | 20 pts | ✅ |
| **Total** | **100 pts** | 🏆 |

---

## Réalisé par : EL OUARZAZI AYA
