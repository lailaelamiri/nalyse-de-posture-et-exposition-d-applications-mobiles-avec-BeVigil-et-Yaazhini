# Rapport d'analyse de sécurité mobile

## A. Informations générales

| Champ | Valeur |
|-------|--------|
| **Date** | 15-MAY-2026 |
| **Analyste** | LAILA ELAMIRI |
| **Cible** | jakhar.aseem.diva (DIVA Android) |
| **Version** | 1.0 — Min SDK 15 / Target SDK 23 — 1.4 MB |
| **Outils utilisés** | BeVigil (analyse externe), Yaazhini (analyse statique interne) |

---

## B. Résumé exécutif

L'analyse de l'application **DIVA Android** (`jakhar.aseem.diva`) a mis en évidence **12 constats de sécurité** couvrant 4 catégories OWASP MASVS : stockage, réseau, plateforme et résilience. Le niveau de risque global est **ÉLEVÉ**, avec un constat critique (communication HTTP non chiffrée, CVSS 8.1) et trois constats de sévérité moyenne (mode debug, backup Android, ContentProvider exporté). Les catégories les plus touchées sont MASVS-NETWORK (4 findings) et MASVS-PLATFORM (3 findings). En complément, l'analyse externe via BeVigil sur `com.cerdillac.filmmaker` révèle une surface d'attaque étendue (31 IP hardcodées, 50 endpoints REST, 1 Firebase URL — désactivée et à faible risque actuel).

---

## C. Top 5 constats

### 1. Communication HTTP non sécurisée — FIND-001

- **Sévérité**: 🔴 HIGH (CVSS 8.1)
- **Preuve**: `jakhar/aseem/diva/APICreds2Activity.java` — Ligne 23
- **Impact**: Les données sensibles (credentials, PIN) transitent en clair sur le réseau. Une attaque Man-in-the-Middle (MitM) permet leur interception sans effort technique significatif.
- **Remédiation**: Remplacer toutes les URLs `http://` par `https://`. Appliquer TLS sur toutes les connexions transmettant des données sensibles. Considérer l'utilisation de certificate pinning.
- **Référence OWASP**: MASVS-NETWORK-1

---

### 2. Mode debug activé en production — FIND-003

- **Sévérité**: 🟠 MEDIUM (CVSS 4.9)
- **Preuve**: `AndroidManifest.xml` — Ligne 7 (`android:debuggable="true"`)
- **Impact**: Un attaquant peut attacher un débogueur ADB sur un appareil physique, inspecter la mémoire à l'exécution, extraire des données sensibles ou injecter du code arbitraire.
- **Remédiation**: Supprimer l'attribut ou définir `android:debuggable="false"` dans le manifeste. S'assurer que le flag est automatiquement désactivé lors des builds de production via la configuration Gradle.
- **Référence OWASP**: MASVS-RESILIENCE-2

---

### 3. Backup Android activé — FIND-002

- **Sévérité**: 🟠 MEDIUM (CVSS 4.9)
- **Preuve**: `AndroidManifest.xml` — Ligne 7 (`android:allowBackup="true"`)
- **Impact**: Avec un accès physique au terminal, un attaquant peut extraire l'intégralité des données internes de l'application (sous `/data/data/`) via la commande `adb backup`, sans nécessiter de root.
- **Remédiation**: Définir `android:allowBackup="false"`. Si des sauvegardes sont nécessaires, utiliser l'API `BackupAgent` avec un contrôle explicite des données incluses.
- **Référence OWASP**: MASVS-STORAGE-4

---

### 4. ContentProvider exporté sans protection — FIND-004

- **Sévérité**: 🟠 MEDIUM (CVSS 4.9)
- **Preuve**: `AndroidManifest.xml` — Ligne 36 (`android:exported="true"` sans permission)
- **Impact**: Toute application installée sur l'appareil peut accéder aux données gérées par ce provider sans authentification, permettant lecture, modification ou suppression des données applicatives.
- **Remédiation**: Définir `android:exported="false"` si le provider est à usage interne. Sinon, protéger l'accès avec une permission personnalisée de niveau `signature` ou `signatureOrSystem`.
- **Référence OWASP**: MASVS-PLATFORM-2

---

### 5. Stockage de données sensibles sur stockage externe — FIND-006

- **Sévérité**: ⚠️ WARNING
- **Preuve**: `jakhar/aseem/diva/InsecureDataStorage4Activity.java` — Ligne 24 (fichier `.uinfo.txt` sur carte SD)
- **Impact**: Le fichier créé sur le stockage externe est lisible et modifiable par toute application disposant de la permission `READ_EXTERNAL_STORAGE`. Les données utilisateur (informations personnelles) ne bénéficient d'aucune protection.
- **Remédiation**: Stocker les données sensibles dans le stockage interne (`getFilesDir()` ou `getDataDir()`). Si le stockage externe est indispensable, chiffrer les données avec AES-256 avant écriture.
- **Référence OWASP**: MASVS-STORAGE-2

---

## D. Faux positifs notables

| Finding | Outil | Justification |
|---------|-------|---------------|
| Firebase URL exposée (`bff-test.firebaseio.com`) | BeVigil | Test d'accès confirmé : la base de données Firebase est **désactivée** (`{"error": "The Firebase database 'bff-test' has been deactivated."}`). Aucune donnée accessible. Risque résiduel faible (historique d'usage confirmé). |
| Adresses email dans le code source | BeVigil | Les 8 adresses email trouvées sont vraisemblablement des contacts développeurs ou des adresses de support intégrées pour les retours d'erreur. Sans confirmation de données sensibles exposées, le risque est faible. |
| Chemins de shaders GLSL (ChenXinHeng0430/...) | BeVigil | Les ressources tierces détectées correspondent à des assets graphiques (shaders vidéo OpenGL ES), non à des données sensibles ou des endpoints d'API exploitables. |

---

## E. Recommandations prioritaires

1. **Migrer immédiatement toutes les communications vers HTTPS** — Identifier et remplacer l'ensemble des URLs HTTP dans le code source, en commençant par `APICreds2Activity.java`. Mettre en place un Network Security Config Android pour bloquer le trafic en clair.

2. **Durcir la configuration du manifeste Android** — Désactiver le mode debug (`android:debuggable="false"`), désactiver les sauvegardes ADB (`android:allowBackup="false"`) et restreindre l'export du ContentProvider avec une permission explicite. Ces trois corrections peuvent être appliquées en une seule itération de build.

3. **Implémenter une politique de protection des données sensibles en stockage** — Déplacer le stockage des données utilisateur du stockage externe vers le stockage interne chiffré, activer `FLAG_SECURE` sur les 17 activités identifiées pour prévenir les captures d'écran, et désactiver le copier/coller sur les 34 champs EditText contenant des données sensibles.

---

## F. Annexes

- **Rapport Yaazhini** : `yaazhini_notes.md` — 8 findings (1 High, 3 Medium, 1 Low, 1 Warning, 2 Information)
- **Analyse BeVigil** : `bevigil_notes.md` — Surface d'attaque externe (50 endpoints, 31 IPs, 1 Firebase URL)
- **Mapping OWASP MASVS** : `owasp_mapping.md` — 12 findings mappés sur MASVS v2
- **Référence OWASP MASVS v2** : https://github.com/OWASP/owasp-masvs
- **Référence OWASP MASTG** : https://github.com/OWASP/owasp-mastg

---

*Rapport généré le 15-MAY-2026 — Basé sur l'analyse statique Yaazhini et l'analyse externe BeVigil de l'application DIVA Android.*
