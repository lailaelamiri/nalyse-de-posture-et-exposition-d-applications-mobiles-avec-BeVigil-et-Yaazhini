# Notes d'analyse Yaazhini

**Application**: apk (`jakhar.aseem.diva`)  
**Version**: 1.0  
**Date de scan**: 15-MAY-2026  
**Min SDK**: 15 | **Target SDK**: 23  
**Taille**: 1.4 MB  

---

## Résumé des findings

| Risque | Nombre |
|--------|--------|
| High | 1 |
| Medium | 3 |
| Low | 1 |
| Warning | 1 |
| Information | 2 |
| **Total** | **8** |

---

## Éléments identifiés

### Élément 1: Communication non sécurisée (HTTP)
- **Localisation**: `jakhar\aseem\diva\APICreds2Activity.java` — Ligne 23
- **Description**: Une URL HTTP est utilisée dans le code source. Le trafic réseau n'est pas chiffré. Le message affiché à l'utilisateur contient une URL en clair vers un service d'enregistrement.
- **Impact potentiel**: Interception du trafic réseau par attaque Man-in-the-Middle (MitM). Les données sensibles (credentials, PIN) transitent en clair.
- **Remédiation suggérée**: Remplacer toutes les URLs HTTP par HTTPS. Utiliser SSL/TLS pour toutes les connexions transmettant des données sensibles.
- **CVSS**: 8.1 — **Sévérité: HIGH**

---

### Élément 2: Mode debug activé
- **Localisation**: `AndroidManifest.xml` — Ligne 7
- **Description**: La propriété `android:debuggable="true"` est présente dans le tag application. L'application peut être déboguée même sur un appareil physique en production.
- **Impact potentiel**: Un attaquant peut utiliser ADB pour accéder aux données internes, injecter du code ou extraire des informations sensibles via le mode debug.
- **Remédiation suggérée**: Supprimer ou définir `android:debuggable="false"` dans le manifeste pour les builds de production.
- **CVSS**: 4.9 — **Sévérité: MEDIUM**

---

### Élément 3: Vulnérabilité de backup Android
- **Localisation**: `AndroidManifest.xml` — Ligne 7
- **Description**: La propriété `android:allowBackup="true"` permet la sauvegarde des données internes de l'application (sous `/data/data/`) via ADB.
- **Impact potentiel**: Un attaquant avec accès physique au téléphone peut extraire toutes les données internes de l'application via `adb backup`, y compris credentials et données utilisateur.
- **Remédiation suggérée**: Définir `android:allowBackup="false"` dans `AndroidManifest.xml`.
- **CVSS**: 4.9 — **Sévérité: MEDIUM**

---

### Élément 4: Export incorrect de providers (ContentProvider)
- **Localisation**: `AndroidManifest.xml` — Ligne 36
- **Description**: Un ContentProvider est exporté avec `android:exported="true"`, le rendant accessible depuis d'autres applications Android sans restriction.
- **Impact potentiel**: D'autres applications malveillantes peuvent accéder aux données gérées par ce provider sans authentification.
- **Remédiation suggérée**: Définir `android:exported="false"` si le provider n'a pas besoin d'être partagé entre applications. Sinon, protéger l'accès avec une permission explicite.
- **CVSS**: 4.9 — **Sévérité: MEDIUM**

---

### Élément 5: JavaScript activé dans WebView
- **Localisation**: `jakhar\aseem\diva\InputValidation2URISchemeActivity.java` — Ligne 14
- **Description**: L'application active JavaScript dans une WebView via `setJavaScriptEnabled(true)`. Cela peut permettre l'exécution de code JavaScript arbitraire.
- **Impact potentiel**: Vulnérabilité Cross-Site Scripting (XSS) possible si du contenu web non fiable est chargé dans la WebView.
- **Remédiation suggérée**: Désactiver JavaScript si non nécessaire. Si requis, n'autoriser que du code JavaScript de confiance et auditer le contenu pour les attaques XSS.
- **CVSS**: 2.9 — **Sévérité: LOW**

---

### Élément 6: Utilisation du stockage externe
- **Localisation**: `jakhar\aseem\diva\InsecureDataStorage4Activity.java` — Ligne 24 (+ 8 autres occurrences dans les libs support)
- **Description**: L'application écrit des données dans le stockage externe (carte SD) via `Environment.getExternalStorageDirectory()`. Le fichier `.uinfo.txt` est créé sans protection. Le stockage externe est lisible et modifiable globalement.
- **Impact potentiel**: Toute application ou utilisateur peut lire, modifier ou supprimer les données stockées. Aucun mécanisme de sécurité n'est appliqué sur les fichiers externes.
- **Remédiation suggérée**: Utiliser le stockage interne pour les données sensibles. Si le stockage externe est nécessaire, chiffrer les données avant de les écrire.
- **Sévérité: WARNING**

---

### Élément 7: Absence de protection copier/coller dans les champs EditText
- **Localisation**: Multiple fichiers — 34 occurrences (layouts XML dont `activity_apicreds2.xml`, `activity_hardcode.xml`, `activity_insecure_data_storage*.xml`, etc.)
- **Description**: Les champs de saisie (EditText) ne désactivent pas le copier/coller. Des données sensibles saisies (PIN, mots de passe) peuvent être copiées dans le presse-papiers Android et accessibles par d'autres applications.
- **Impact potentiel**: Fuite de données sensibles via le clipboard, accessible par des applications tierces malveillantes.
- **Remédiation suggérée**: Désactiver les opérations copier/coller sur les champs contenant des données sensibles (PIN, numéros de carte, mots de passe).
- **Sévérité: INFORMATION**

---

### Élément 8: Absence de protection contre les captures d'écran
- **Localisation**: Application globale — 17 occurrences
- **Description**: L'application n'implémente pas de protection contre les captures d'écran. Des écrans contenant des données sensibles peuvent être capturés.
- **Impact potentiel**: Fuite de données sensibles via des screenshots, accessibles dans la galerie ou partagés par des applications tierces.
- **Remédiation suggérée**: Désactiver les captures d'écran sur les écrans sensibles en utilisant `FLAG_SECURE` dans les activités concernées.
- **Sévérité: INFORMATION**

---

## Observations comparatives BeVigil vs Yaazhini

| Aspect | BeVigil (externe) | Yaazhini (interne) |
|--------|-------------------|---------------------|
| Portée | Analyse externe (URLs, endpoints exposés) | Analyse interne (code source, manifeste) |
| Secrets | Endpoints et Firebase URLs découverts | Hardcoded HTTP URL dans APICreds2Activity |
| Configuration | Non applicable | Debug mode, backup, exported providers |
| Permissions | Visibles depuis l'extérieur | Analysées dans le contexte du code |

