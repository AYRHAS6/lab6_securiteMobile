#  Rapport d'Analyse Statique de Sécurité Mobile
## Application : DIVA (Damn Insecure and Vulnerable App)

---

## 📋 Informations Générales

| Champ | Valeur |
|---|---|
| **Nom de l'application** | Diva |
| **Package** | `jakhar.aseem.diva` |
| **Version** | 1.0 (code : 1) |
| **Fichier analysé** | `DivaApplication.apk` |
| **Taille** | 1.43 MB |
| **SHA-256** | `5cefc51fce9bd760b92ab2340477f4dda84b4ae0c5d04a8c9493e4fe34fab7c5` |
| **MD5** | `82ab8b2193b3cfb1c737e3a786be363a` |
| **Date d'analyse** | 27 mai 2026 |
| **Outil utilisé** | MobSF v4.5.0 |
| **Target SDK** | 23 (Android 6.0) |
| **Min SDK** | 15 (Android 4.0.3) |

---

##  Score de Sécurité

```
Score global : 36 / 100  →  Grade C  →  RISQUE ÉLEVÉ (HIGH RISK)
```

<img width="733" height="454" alt="1" src="https://github.com/user-attachments/assets/ec4366e2-c284-41fe-baa9-e4896487d498" />


| Sévérité | Nombre |
|---|---|
|  HIGH | 5 |
|  MEDIUM/WARNING | 7 |
|  INFO | 1 |
|  SECURE | 1 |
|  HOTSPOT | 1 |

---

##  Résumé Exécutif

L'analyse statique de l'application **DIVA v1.0** révèle un niveau de risque **élevé** (score : 36/100). Cette application est intentionnellement vulnérable et conçue à des fins pédagogiques — elle ne doit en aucun cas être déployée en production.

Les principales vulnérabilités concernent le **stockage non sécurisé de données sensibles**, des **composants Android exportés sans protection**, l'utilisation d'un **certificat de débogage en production**, et l'absence de protections binaires dans les bibliothèques natives (NX bit, Stack Canary). L'application demande **2 permissions dangereuses** (stockage externe en lecture/écriture) qui correspondent à des vecteurs d'attaque fréquemment exploités par des malwares connus.

---

##  Top 5 Vulnérabilités

---

### 1.  Application signée avec un certificat de débogage

| Attribut | Détail |
|---|---|
| **Sévérité** | HIGH |
| **Référence** | OWASP MASVS : MSTG-RESILIENCE-2 |
| **Fichier** | `jakhar/aseem/diva/BuildConfig.java` |
| **CWE** | CWE-919 |

**Description :** L'APK est signé avec un certificat Android Debug (`CN=Android Debug`) valable jusqu'en 2045. Un certificat de débogage est public et partagé entre tous les développeurs Android, ce qui permet à n'importe qui de signer une version modifiée de l'application.

**Impact :** Usurpation d'identité de l'application, attaques de type Janus (CVE-2017-13156) sur Android 5.0–8.0 permettant d'injecter du code malveillant sans invalider la signature v1.

**Recommandation :** Signer toute build de production avec un keystore privé dédié. Désactiver `android:debuggable="true"` dans le `AndroidManifest.xml`.

<img width="1027" height="247" alt="2" src="https://github.com/user-attachments/assets/75151739-ca5f-49db-b408-2695c7ac5265" />


---

### 2.  Mode Debug activé (`android:debuggable=true`)

| Attribut | Détail |
|---|---|
| **Sévérité** | HIGH |
| **Référence** | OWASP MASVS : MSTG-RESILIENCE-2 / OWASP Top 10 M1 |
| **Fichier** | `AndroidManifest.xml` |
| **CWE** | CWE-919 |

**Description :** Le flag `android:debuggable=true` est activé dans le manifeste, permettant à un attaquant ayant un accès physique ou via ADB d'attacher un débogueur au processus de l'application.

**Impact :** Extraction de données sensibles en mémoire, contournement des contrôles d'accès, dumping de stack trace, reverse engineering facilité.

**Recommandation :** Retirer ce flag pour toute build de production. Les build systems modernes (Gradle) le gèrent automatiquement via les variantes `debug` / `release`.

<img width="991" height="70" alt="3" src="https://github.com/user-attachments/assets/a0411562-7d52-4c9e-a262-6a6ecc32efb4" />


---

### 3.  Composants Android exportés sans protection

| Attribut | Détail |
|---|---|
| **Sévérité** | HIGH / WARNING |
| **Référence** | OWASP MASVS : MSTG-PLATFORM-1 |
| **Fichiers** | `APICredsActivity.java`, `APICreds2Activity.java`, `NotesProvider.java` |

**Description :** Deux Activities (`APICredsActivity`, `APICreds2Activity`) et un Content Provider (`NotesProvider`) sont exportés sans permission requise. Toute application installée sur l'appareil peut les invoquer directement.

**Impact :** Accès non autorisé aux données de l'application (notes, credentials API), possibilité de déclencher des écrans sensibles sans authentification.

**Recommandation :** Ajouter `android:exported="false"` sur les composants qui ne doivent pas être accessibles externement, ou définir une `android:permission` personnalisée de niveau `signature`.

<img width="970" height="214" alt="4" src="https://github.com/user-attachments/assets/f5c922fb-2f44-4de3-b057-30fccc829ad8" />


---

### 4.  Injection SQL possible

| Attribut | Détail |
|---|---|
| **Sévérité** | WARNING |
| **Référence** | OWASP Top 10 M7 / OWASP MASVS : MSTG-STORAGE-2 |
| **Fichiers** | `InsecureDataStorage2Activity.java`, `NotesProvider.java`, `SQLInjectionActivity.java` |
| **CWE** | CWE-89 |

**Description :** L'application utilise des requêtes SQL brutes (`rawQuery`) sans paramétrage. Les entrées utilisateur sont directement concaténées dans les requêtes.

**Impact :** Un attaquant peut extraire, modifier ou supprimer l'ensemble des données de la base SQLite locale (ex. : `' OR '1'='1`).

**Recommandation :** Utiliser exclusivement des requêtes paramétrées (`?` avec `selectionArgs`). Envisager l'utilisation de Room ORM qui protège contre ce vecteur par défaut.

<img width="1000" height="151" alt="5" src="https://github.com/user-attachments/assets/1ddfcf13-9f16-415e-bd71-d5e860e369e8" />

---

### 5.  Protections binaires absentes (NX bit & Stack Canary)

| Attribut | Détail |
|---|---|
| **Sévérité** | HIGH |
| **Bibliothèques concernées** | `libdivajni.so` (armeabi, arm64-v8a, mips, mips64, x86_64, armeabi-v7a) |

**Description :** 12 des 14 bibliothèques natives analysées ne disposent ni du **NX bit** (protection contre l'exécution de code en zone mémoire non-exécutable) ni de **Stack Canary** (détection de dépassements de tampon sur la pile). Seules les variantes x86 présentent un Stack Canary.

**Impact :** Exploitation facilitée de vulnérabilités de type buffer overflow, attaques Return-Oriented Programming (ROP) possibles.

**Recommandation :** Compiler avec les options `-z noexecstack` (NX) et `-fstack-protector-all` (canary). Ajouter `-D_FORTIFY_SOURCE=2` pour la protection des fonctions libc.


---

##  Recommandations Priorisées

| Priorité | Action | Impact |
|---|---|---|
| 🔴 1 | Désactiver `android:debuggable="true"` dans le manifeste de production | Critique |
| 🔴 2 | Remplacer le certificat de débogage par un keystore de production privé | Critique |
| 🔴 3 | Protéger ou ne pas exporter `APICredsActivity`, `APICreds2Activity` et `NotesProvider` | Élevé |
| 🟠 4 | Remplacer les requêtes SQL brutes par des requêtes paramétrées ou Room ORM | Élevé |
| 🟠 5 | Recompiler les bibliothèques natives avec NX, Stack Canary et FORTIFY_SOURCE | Moyen |

---

## 📎 Annexes Techniques

### Permissions Déclarées

| Permission | Niveau | Usage légitime ? |
|---|---|---|
| `WRITE_EXTERNAL_STORAGE` |  Dangerous | À justifier |
| `READ_EXTERNAL_STORAGE` |  Dangerous | À justifier |
| `INTERNET` | Normal | Oui |

> Ces 3 permissions correspondent à des vecteurs abusés dans **3/25 catégories de malwares** recensées par MobSF.

---

### Composants Exportés

| Type | Composant | Protégé ? |
|---|---|---|
| Activity | `jakhar.aseem.diva.APICredsActivity` |  Non |
| Activity | `jakhar.aseem.diva.APICreds2Activity` |  Non |
| Content Provider | `jakhar.aseem.diva.NotesProvider` |  Non |

---

### Secret Hardcodé Détecté

```
"pkey" : "notespin"
```

<img width="1017" height="165" alt="6" src="https://github.com/user-attachments/assets/3ffd27d2-f577-44b7-bcd1-5c7ef65ae153" />

---

### Domaines & Endpoints Détectés

| Domaine | Statut | Localisation |
|---|---|---|
| `payatu.com` |  Clean | San Francisco, US (IP: 172.67.72.183) |

---

*Rapport généré à partir de MobSF v4.5.0 — Analyse réalisée le 27 mai 2026*
*Application analysée à des fins strictement éducatives (DIVA est un lab intentionnellement vulnérable)*
