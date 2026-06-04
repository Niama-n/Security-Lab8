# Synthèse Yaazhini – InsecureBankv2

### 1) Communications en clair
- Localisation : Findings Yaazhini (#1) – code/source/network
- Observation : certaines requêtes utilisent HTTP sans chiffrement (protocole fixé en dur).
- Risque : interception et modification des échanges (MITM) pouvant divulguer identifiants et informations sensibles.
- Mesures recommandées : imposer TLS 1.2+ systématiquement et envisager le pinning des certificats.

### 2) Application débogable
- Localisation : Findings Yaazhini (#3) – AndroidManifest.xml (android:debuggable="true")
- Observation : build livré avec le flag debuggable actif.
- Risque : accès via ADB permettant extraction de données ou manipulation runtime.
- Mesures recommandées : désactiver `android:debuggable` sur les builds destinés à la production.

### 3) Sauvegarde ADB activée
- Localisation : Findings Yaazhini (#4) – AndroidManifest.xml (android:allowBackup="true")
- Observation : la fonctionnalité de backup est autorisée sans contrôle.
- Risque : récupération complète des données applicatives via connexion USB.
- Mesures recommandées : désactiver `allowBackup` ou définir un fichier `fullBackupContent` restrictif.

### 4) Hachage obsolète (MD5 / SHA-1)
- Localisation : Findings Yaazhini (#7/#8) – ChangePassword.java
- Observation : utilisation de MD5 et SHA-1 pour des opérations de hachage.
- Risque : vulnérabilité aux attaques par tables arc-en-ciel et collisions.
- Mesures recommandées : migrer vers SHA-256+ ou utiliser des fonctions adaptatives (PBKDF2, bcrypt, Argon2) avec sel.

### 5) Content Providers exposés
- Localisation : Findings Yaazhini (#5) – AndroidManifest.xml (exported=true)
- Observation : providers déclarés publics sans permissions protectrices.
- Risque : fuite ou modification des données internes par des applications tierces.
- Mesures recommandées : limiter `android:exported` ou imposer `readPermission`/`writePermission`.

### 6) Générateur non sécurisé
- Localisation : Findings Yaazhini (#2)
- Observation : usage de `java.util.Random` pour des tâches sensibles.
- Risque : prédictibilité des valeurs (tokens, nonces).
- Mesures recommandées : remplacer par `java.security.SecureRandom` pour les usages cryptographiques.

### 7) Schéma de signature faible
- Localisation : Findings Yaazhini (#9)
- Observation : APK signée avec `SHA1withRSA`.
- Risque : réduction de la résistance aux collisions et à la falsification.
- Mesures recommandées : signer avec `SHA256withRSA` et utiliser le scheme de signature v2+ d'Android.
