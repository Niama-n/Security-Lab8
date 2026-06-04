# Cartographie OWASP / MASVS – InsecureBankv2

## FIND-001 — Communications non chiffrées
- Catégorie : MASVS-NETWORK
- Référence : MASVS-NETWORK-1
- Motif : Absence de TLS pour certaines requêtes, exposant le trafic à l'interception.

## FIND-002 — Application débogable
- Catégorie : MASVS-CODE
- Référence : MASVS-CODE-2
- Motif : Flag `android:debuggable` présent en production ; facilite le reverse-engineering et l'injection.

## FIND-003 — Sauvegarde ADB autorisée
- Catégorie : MASVS-STORAGE
- Référence : MASVS-STORAGE-8
- Motif : `android:allowBackup=true` permet l'extraction de données via ADB backup.

## FIND-004 — Hachage obsolète (MD5 / SHA-1)
- Catégorie : MASVS-CRYPTO
- Référence : MASVS-CRYPTO-4
- Motif : Algorithmes de hachage faibles utilisés pour des données sensibles.

## FIND-005 — Content Providers exposés
- Catégorie : MASVS-PLATFORM
- Référence : MASVS-PLATFORM-2
- Motif : Providers déclarés publics sans permissions restrictives.

## FIND-006 — Générateur non sécurisé
- Catégorie : MASVS-CRYPTO
- Référence : MASVS-CRYPTO-6
- Motif : Usage de `java.util.Random` pour des opérations sécurisées.

## FIND-007 — Signature APK faible
- Catégorie : MASVS-CODE
- Référence : MASVS-CODE-1
- Motif : Schéma de signature ou algorithme insuffisamment robuste (SHA1withRSA).

## FIND-008 — Bibliothèques de suivi tiers
- Catégorie : MASVS-PLATFORM
- Référence : MASVS-PLATFORM-1
- Motif : Présence d'outils analytiques/publicitaires pouvant collecter des données utilisateurs.

## FIND-010 — Receivers exportés sans garde
- Catégorie : MASVS-PLATFORM
- Référence : MASVS-PLATFORM-2
- Motif : Receivers accessibles par d'autres applications sans contrôle d'accès.

## FIND-012 — JavaScript activé dans WebView
- Catégorie : MASVS-PLATFORM
- Référence : MASVS-PLATFORM-5
- Motif : JavaScript autorisé dans des WebViews ; risque XSS si contenu non fiable.
