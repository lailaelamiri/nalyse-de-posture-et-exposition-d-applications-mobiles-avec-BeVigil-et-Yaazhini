# Mapping OWASP MASVS

**Standard de référence**: OWASP Mobile Application Security Verification Standard (MASVS) v2  
**Application analysée**: jakhar.aseem.diva (DIVA Android)  
**Date**: 15-MAY-2026  

---

## FIND-001: Communication HTTP non sécurisée
- **Catégorie OWASP**: MASVS-NETWORK
- **Référence spécifique**: MASVS-NETWORK-1
- **Justification**: L'application utilise HTTP au lieu de HTTPS dans `APICreds2Activity.java`, exposant les données en transit à une interception. MASVS-NETWORK-1 exige que toutes les communications réseau soient chiffrées via TLS.

---

## FIND-002: Backup Android activé
- **Catégorie OWASP**: MASVS-STORAGE
- **Référence spécifique**: MASVS-STORAGE-4
- **Justification**: `android:allowBackup="true"` permet l'extraction des données internes via ADB. MASVS-STORAGE-4 exige que les sauvegardes automatiques ne révèlent pas de données sensibles.

---

## FIND-003: Mode debug activé
- **Catégorie OWASP**: MASVS-RESILIENCE
- **Référence spécifique**: MASVS-RESILIENCE-2
- **Justification**: `android:debuggable="true"` en production permet à un attaquant d'attacher un débogueur et d'inspecter la mémoire de l'application. MASVS-RESILIENCE-2 exige que le débogage soit désactivé sur les builds de production.

---

## FIND-004: ContentProvider exporté sans protection
- **Catégorie OWASP**: MASVS-PLATFORM
- **Référence spécifique**: MASVS-PLATFORM-2
- **Justification**: Un ContentProvider avec `android:exported="true"` sans permission expose les données de l'application à toute autre application installée. MASVS-PLATFORM-2 exige que les composants IPC ne soient accessibles qu'aux applications autorisées.

---

## FIND-005: JavaScript activé dans WebView
- **Catégorie OWASP**: MASVS-PLATFORM
- **Référence spécifique**: MASVS-PLATFORM-5
- **Justification**: `setJavaScriptEnabled(true)` dans une WebView peut permettre l'exécution de scripts malveillants si du contenu non fiable est chargé. MASVS-PLATFORM-5 exige que JavaScript soit désactivé dans les WebViews à moins d'être strictement nécessaire.

---

## FIND-006: Stockage de données sur stockage externe
- **Catégorie OWASP**: MASVS-STORAGE
- **Référence spécifique**: MASVS-STORAGE-2
- **Justification**: Le fichier `.uinfo.txt` est écrit sur la carte SD sans chiffrement, accessible en lecture/écriture par toute application. MASVS-STORAGE-2 exige que les données sensibles ne soient pas stockées en clair sur le stockage externe.

---

## FIND-007: Endpoints REST API exposés
- **Catégorie OWASP**: MASVS-NETWORK
- **Référence spécifique**: MASVS-NETWORK-2
- **Justification**: Les endpoints REST découverts via BeVigil peuvent révéler la surface d'attaque de l'API backend. MASVS-NETWORK-2 exige que les endpoints soient protégés par authentification et que la surface d'attaque soit minimisée.

---

## FIND-008: URLs Firebase exposées
- **Catégorie OWASP**: MASVS-STORAGE
- **Référence spécifique**: MASVS-STORAGE-1
- **Justification**: Les URLs Firebase exposées combinées à des règles de sécurité mal configurées peuvent permettre un accès non autorisé à la base de données. MASVS-STORAGE-1 exige que les données sensibles ne soient pas accessibles sans authentification appropriée.

---

## FIND-009: Adresses IP exposées dans l'application
- **Catégorie OWASP**: MASVS-NETWORK
- **Référence spécifique**: MASVS-NETWORK-1
- **Justification**: Les adresses IP hardcodées dans le code révèlent la topologie de l'infrastructure backend et facilitent les attaques ciblées. MASVS-NETWORK-1 recommande d'éviter l'exposition d'informations réseau sensibles dans le code client.

---

## FIND-010: Absence de protection copier/coller sur champs sensibles
- **Catégorie OWASP**: MASVS-PLATFORM
- **Référence spécifique**: MASVS-PLATFORM-4
- **Justification**: 34 champs EditText sans protection clipboard permettent la fuite de données sensibles via le presse-papiers Android. MASVS-PLATFORM-4 exige que les données sensibles ne soient pas exposées via les mécanismes IPC de la plateforme, incluant le clipboard.

---

## FIND-011: Absence de protection contre les captures d'écran
- **Catégorie OWASP**: MASVS-RESILIENCE
- **Référence spécifique**: MASVS-RESILIENCE-4
- **Justification**: L'absence de `FLAG_SECURE` sur 17 activités permet la capture d'écrans contenant des données sensibles. MASVS-RESILIENCE-4 exige que l'application empêche la capture de données sensibles affichées à l'écran.

---

## FIND-012: URLs relatives exposées
- **Catégorie OWASP**: MASVS-NETWORK
- **Référence spécifique**: MASVS-NETWORK-2
- **Justification**: Les chemins d'accès relatifs exposés dans le code client révèlent la structure interne de l'API et peuvent être utilisés pour du fuzzing ou de l'énumération. MASVS-NETWORK-2 exige que la surface d'attaque de l'API soit minimisée.

---

## Résumé des catégories MASVS identifiées

| Catégorie | Findings | Description |
|-----------|---------|-------------|
| MASVS-STORAGE | FIND-002, FIND-006, FIND-008 | Stockage non sécurisé des données |
| MASVS-NETWORK | FIND-001, FIND-007, FIND-009, FIND-012 | Communications réseau non sécurisées |
| MASVS-PLATFORM | FIND-004, FIND-005, FIND-010 | Interactions non sécurisées avec la plateforme |
| MASVS-RESILIENCE | FIND-003, FIND-011 | Manque de résistance aux attaques |

---

## Références
- OWASP MASVS v2: https://github.com/OWASP/owasp-masvs
- OWASP MASTG: https://github.com/OWASP/owasp-mastg
- OWASP Mobile Top 10: https://owasp.org/www-project-mobile-top-10/
