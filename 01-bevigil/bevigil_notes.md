# Synthèse BeVigil – InsecureBankv2

## Constats solides
- Note globale détectée : 7.4 / 10 (niveau moyen)
- Package analysé : com.android.insecurebankv2 (v1.0)
- Nombre d'issues signalées par BeVigil : 23 (niveau Low)
- Composants exportés non protégés : 4 occurrences dans le manifest
- Potentiel secret identifié : 1 entrée dans `res/values/strings.xml`
- Assets recensés : 144 (fichiers, URLs et noms d'hôte)
- Trackers tiers identifiés : AdMob, Google Analytics, Tag Manager
- Bibliothèques externes détectées : 15
- Permissions potentiellement dangereuses : 5

## Hypothèses utiles pour la remédiation
- L'élément marqué comme "secret" pourrait correspondre à une clé API ou un identifiant de test codé en dur.
- Les composants exportés peuvent exposer des interfaces internes à d'autres applications installées.
- Des connexions en clair observées (HTTP) laissent supposer un trafic non chiffré pour certains endpoints.

## Observations techniques
- Indicateurs de vulnérabilités cryptographiques : cas de CBC (padding oracle) détectés (3 occurrences)
- Données sensibles potentiellement stockées en clair dans SharedPreferences (2 occurrences)
- Informations sensibles présentes dans des logs (2 occurrences)
- Requêtes SQL non paramétrées détectées (1 occurrence)
- Vérification de root incomplète / absence d'utilisation correcte de SafetyNet

## Hôtes et domaines repérés (extraits)
- www.googleapis.com, pagead2.googlesyndication.com, www.google-analytics.com, www.googletagmanager.com
- plus.google.com, www.facebook.com, accounts.google.com, login.live.com, www.paypal.com
- Total d'hôtes relevés : 23

## API et URLs
- Endpoints REST identifiés : 2
- URLs extraites du binaire : 61
- Cas d'HTTP non chiffré : 4 occurrences liées à un client HTTP non sécurisé

## Données sensibles en clair
- Adresses email trouvées : 6
- Ressource potentiellement sensible : `res/values/strings.xml`

## Bibliothèques et technologies observées
- Android Support (v4 / v7), Google Mobile Services, Google Maps, Google Fit
- Utilisation de bibliothèques publicitaires/analytiques (Ads/Analytics/Tag Manager)
- Signes d'algorithmes faibles et de générateurs non cryptographiques (Weak Crypto, Insecure Random)
