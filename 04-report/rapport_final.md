# Rapport final — Analyse de sécurité mobile

## 1) Métadonnées
- Date d'analyse : 2026-06-04
- Analyste : NAFTAOUI NIAMA
- Application : InsecureBankv2 (`com.android.insecurebankv2`)
- Hash de l'artefact (SHA-256) : B18AF2A0E44D7634BBCDF93664D9C78A2695E050393FCFBB5E8B91F902D194A4
- Outils principaux : BeVigil (analyse externe), Yaazhini (analyse statique/décompilation)

## 2) Synthèse exécutive
L'évaluation a identifié douze constats consolidés, dont cinq classés à haute criticité. Les problèmes récurrents concernent :
- des communications non chiffrées,
- des flags de debug/backups dangereux dans le manifest,
- l'emploi d'algorithmes de hachage obsolètes,
- des composants Android (providers/receivers) exposés,
- l'utilisation de générateurs aléatoires non cryptographiques.

Ces lacunes rendent l'application inadaptée à un usage bancaire en production ; il est impératif de corriger prioritairement les éléments à sévérité élevée.

## 3) Principaux constats et actions recommandées
- FIND-001 — Communications non chiffrées (High)
  - Preuves : extraits BeVigil + Yaazhini (requêtes HTTP détectées)
  - Risque : interception/MITM des données sensibles
  - Action : migrer tout le trafic vers TLS 1.2+ et envisager certificate pinning

- FIND-002 — Flag `debuggable` actif (High)
  - Preuve : manifest avec `android:debuggable="true"`
  - Risque : exposition des logs et accès via ADB
  - Action : désactiver pour les builds de production

- FIND-003 — `allowBackup` activé (High)
  - Preuve : manifest `android:allowBackup="true"`
  - Risque : exportation des données via backup ADB
  - Action : désactiver ou restreindre via `fullBackupContent`

- FIND-004 — Hachage MD5/SHA-1 (High)
  - Preuve : code source (ChangePassword.java)
  - Risque : facilité d'obtention des valeurs originales
  - Action : utiliser des fonctions modernes (SHA-256, PBKDF2/bcrypt/Argon2)

- FIND-005 — Content Providers exposés (High)
  - Preuve : manifest avec `exported=true` sur des providers
  - Risque : accès par d'autres applications
  - Action : restreindre `exported` et/ou ajouter des permissions

## 4) Observations additionnelles
- Trackers et bibliothèques analytiques présents (AdMob / Analytics / Tag Manager) — vérifier conformité RGPD et nécessité.
- Adresses email et petits éléments d'information de contact identifiés dans le code — retirer du repository.

## 5) Recommandations prioritaires (ordre d'exécution)

### Approche Standard (Criticité)
1. Corriger les flags manifest critiques (`debuggable`, `allowBackup`, `exported`).
2. Remplacer les primitives cryptographiques faibles et sécuriser les fonctions de hachage.
3. Forcer le chiffrement des communications et appliquer des contrôles réseau (Network Security Config).

### Approche Alternative (Praticité & Impact)
**Justification critique** : L'ordre standard suit la sévérité CVSS, mais dans un contexte réel de déploiement, la praticité et l'impact métier doivent primer.

**Ordre révisé par praticité** :
1. **TLS/Communications d'abord** (HIGH impact, déploiement rapide) — Protège immédiatement les données en transit sans modification du code applicatif profond.
2. **Manifest flags** (HIGH impact, effort trivial) — Changements rapides dans XML ; améliore la posture globale sans recompilation majeure.
3. **Cryptographie** (HIGH impact, effort important) — Nécessite refactoring du code métier, tests approfondis, peut introduire des régressions.

**Raison du changement** : Le déploiement de TLS sécurise les données bancaires immédiatement (enjeu client maximal), tandis que la correction des flags manifest ne requiert qu'une recompilation.

---

## 6) Analyse Personnelle et Interprétation Critique

### Contexte Pédagogique vs Production
L'application **InsecureBankv2** est un **cas d'école volontairement vulnérable**. Certains constats demandent une contextualisation :

- **FIND-008 (Trackers)** : Bien que détecté comme risque MASVS-PLATFORM-1, la présence de Google Analytics est courante dans les applications légitimes. Dans un contexte pédagogique, c'est une démonstration intentionnelle de data leakage, pas une faille exploitable.
  
- **FIND-009 (Emails exposés)** : Les 6 adresses email trouvées appartiennent probablement aux développeurs du projet DIVA. C'est une exposition informative, non une vulnérabilité critique en contexte académique.

### Observations Analytiques

**Point 1 — Clustering de vulnérabilités** :
Les 5 findings HIGH se concentrent sur 3 domaines :
- Manifest misconfiguration (FIND-002, FIND-003, FIND-005)
- Cryptographie obsolète (FIND-001, FIND-004)
- Absence de hardening (FIND-005, FIND-006)

**Implication** : Un plan de remédiation peut être **consolidé en 3 streams parallèles**, réduisant le temps d'implémentation.

**Point 2 — Faux positif potentiel** :
BeVigil signale 23 vulnérabilités LOW dans le code. Après décompilation par Yaazhini, seules **13 sont confirmées**. Cela suggère un **taux de faux positifs ~40%** dans l'analyse externe, justifiant l'approche multi-outils.

**Point 3 — Architecture sous-jacente** :
L'absence d'implementation correcte de SafetyNet ou d'attestation Android indique que l'application **ne bénéficie d'aucun hardening runtime**. Cela amplifie l'impact des failles identifiées.

### Évaluation de la Sévérité Alternative

| Finding | Sévérité Standard | Sévérité Revisitée | Justification |
|---------|------|------|---|
| FIND-001 | HIGH | CRITICAL | Impact bancaire + accessibilité (aucune chiffrement) |
| FIND-002 | HIGH | HIGH | Dépend du modèle de menace (accès physique requis) |
| FIND-003 | HIGH | MEDIUM | Nécessite USB + unlock du device + consentement utilisateur |
| FIND-004 | HIGH | MEDIUM | Brute force coûteux sans accès aux hashes |
| FIND-005 | HIGH | HIGH | Attaque intra-device par malware voisin |

**Conclusion** : FIND-001 devrait être **CRITICAL**, pas HIGH, en contexte bancaire.

---

## 7) Considérations d'Implémentation

### Risques de Remédiation
- **Modifier TLS** : Risque de breakage sur serveurs legacy (tester d'abord avec feature flag)
- **Changer crypto** : Risque de rendre les anciens tokens invalides (migration progressive requise)
- **Désactiver flags** : Impact sur le debugging en développement (maintenir debug builds séparés)

### Métriques de Succès Post-Remédiation
- BeVigil score passe de 7.4 à > 8.5
- 0 findings HIGH restants
- Conformité MASVS L2 atteinte (> 85% des contrôles)
- Audit pen-test externe confirme absence d'exploitation rapide

---

## 8) Annexes
- [01-bevigil/bevigil_export_summary.txt](01-bevigil/bevigil_export_summary.txt)
- [01-bevigil/bevigil_notes.md](01-bevigil/bevigil_notes.md)
- [02-yaazhini/yaazhini_report.html](02-yaazhini/yaazhini_report.html)
- [02-yaazhini/yaazhini_notes.md](02-yaazhini/yaazhini_notes.md)
- [03-triage/triage.csv](03-triage/triage.csv)
- [03-triage/owasp_mapping.md](03-triage/owasp_mapping.md)
