# Atelier Calcul


**[Ouvrir l’application Atelier calcul](https://atelier-calcul-famille.wisezebra2048.chatgpt.site)**

L’application est privée et accessible aux personnes invitées. Ce dépôt contient actuellement sa documentation ; le transfert du code source reste à terminer.

Application familiale d’entraînement au calcul : de petites séances générées à partir de modèles, une correction expliquée et un suivi des progrès.

## Fonctionnalités

- Six familles d’exercices sur les nombres relatifs : additions et soustractions, produits et quotients, signe d’un produit, priorités, calculs à trous, programmes de calcul.
- Séances de 5, 10, 15 ou 20 questions, trois difficultés et plages de nombres configurables.
- Réponses entières, décimales ou fractionnaires ; correction par calcul, sans appel à une IA.
- Indices, correction immédiate ou bilan en fin de séance, reprise d’une séance interrompue.
- Espace enfant limité à son profil et à ses résultats.
- Espace parent : profils, réglages, statistiques par enfant, période et notion, détail des tentatives.
- Exports CSV et JSON, import de sauvegardes de la première version.
- Sauvegarde partagée et reprise des envois après une interruption du réseau.

Il n’y a pas d’envoi automatique de rapports par e-mail ni de lecture automatique de nouvelles photos. Un nouveau chapitre demande d’ajouter des modèles au moteur.

## Architecture de la version hébergée

| Élément | Rôle |
| --- | --- |
| HTML, CSS et JavaScript | Interface et génération des exercices dans le navigateur |
| Serveur du site (Vinext / Workers) | Vérification de l’identité et des autorisations, API de sauvegarde |
| Base Cloudflare D1 | Profils, membres, réglages et séances partagés entre appareils |
| `localStorage` | Copie locale de l’état et liste des séances en attente d’envoi |
| Authentification Sites | Connexion des visiteurs invités ; identité validée transmise au serveur |

**Supabase n’est plus nécessaire dans cette version.** Les premières explications du projet évoquaient un mode exclusivement local, puis une synchronisation Supabase ; la version actuelle utilise le stockage intégré au site.

L’accès au code sur GitHub et l’accès au site hébergé sont distincts. Publier le code ne rend pas publics les résultats de la famille.

## Connexion, logins et mots de passe

L’application ne crée pas de comptes avec un mot de passe propre à Atelier Calcul. La connexion est assurée par la plateforme qui héberge le site privé et gère les invitations.

Après identification, le serveur reçoit notamment un identifiant stable pour ce site et l’adresse e-mail vérifiée du visiteur. L’e-mail établit le premier lien avec le parent ou le profil enfant ; les accès suivants utilisent l’identifiant stable enregistré dans les membres de la famille.

Dans la version actuelle :

- aucun mot de passe n’est stocké dans le code, la base de progression ou le stockage local de l’application ;
- la session de connexion est gérée par la plateforme d’authentification, pas par un formulaire de mot de passe dans l’application ;
- le rôle parent ou enfant est contrôlé côté serveur ;
- chaque lecture enfant est limitée à son profil et à ses séances ;
- les modifications des profils et des réglages sont réservées au parent ;
- un enfant ne peut pas modifier une séance appartenant à un autre profil.

Le code de l’application ne permet pas d’affirmer comment le fournisseur d’identité stocke ou chiffre ses propres secrets : il ne met en œuvre ni ne vérifie son mécanisme interne de stockage des mots de passe.

**Pour un autre hébergement :** les en-têtes d’identité doivent provenir d’une passerelle d’authentification de confiance. Accepter directement des en-têtes fournis par un client sur un serveur public permettrait d’usurper une identité. L’intégration d’authentification doit donc être conservée ou remplacée par une solution équivalente.

## Où sont stockés les progrès ?

### 1. Dans la base partagée du site

La sauvegarde durable entre appareils utilise Cloudflare D1. Le schéma comporte quatre tables :

| Table | Contenu |
| --- | --- |
| `family_profiles` | Identifiant du profil, prénom et éventuelle adresse de l’accès enfant |
| `family_members` | Identité du site, rôle parent/enfant et profil associé |
| `family_runs` | Séances et réponses détaillées, avec le profil et la date de mise à jour |
| `family_settings` | Réglages des prochaines séances |

Une séance conserve les questions, les réponses attendues, chaque tentative, son résultat, les indices ou corrections consultés, l’état de la séance et des durées indicatives. Les statistiques sont calculées à partir de ces données.

Les réponses sont envoyées après les modifications, avec un court délai de regroupement. Les statistiques sont actualisées environ toutes les 20 secondes lorsque la page est visible, ainsi qu’à l’ouverture du suivi. L’affichage d’une sauvegarde en attente signifie que l’envoi n’est pas encore confirmé.

### 2. Dans le navigateur de chaque appareil

« En local » signifie ici dans **`localStorage`**, un espace de stockage du navigateur pour l’adresse du site. Ce n’est ni un fichier Excel ni un dossier du PC.

L’application conserve :

- une copie des profils, réglages et séances accessibles à l’utilisateur connecté ;
- une liste des identifiants des séances restant à envoyer.

Les clés sont séparées par identité : `atelier-family-v2:<identifiant>` et `atelier-family-v2:<identifiant>:pending`. La première version peut aussi avoir laissé une sauvegarde sous `atelier-v1`.

Conséquences pratiques :

- la copie locale n’est pas partagée automatiquement entre navigateurs ou appareils ; c’est la base du site qui assure ce partage ;
- effacer les données du site supprime la copie locale, mais pas les séances déjà sauvegardées dans la base ;
- des réponses encore en attente peuvent être perdues si les données du navigateur sont effacées avant leur envoi ;
- les données locales ne sont pas chiffrées par l’application : une personne ayant accès au profil du navigateur ou aux outils de développement peut les consulter ;
- le cache de l’espace parent peut contenir les séances de tous les enfants, tandis que celui d’un enfant ne reçoit que ses propres données.

Un appareil partagé doit donc être utilisé avec un profil de navigateur et une session adaptés. La sauvegarde JSON permet de conserver une copie supplémentaire.

### 3. Ce qui ne part pas dans GitHub

Le dépôt doit contenir le code, les migrations du schéma, les tests et la documentation. Les résultats, bases locales, caches du navigateur, mots de passe, jetons et fichiers d’environnement privés ne doivent pas être versionnés.

Les adresses personnelles et la configuration de la famille doivent être retirées ou remplacées par des exemples dans toute publication publique du code. Le chiffrement de la base ou les garanties propres à l’hébergeur ne doivent pas être déduits de ce README : aucun chiffrement applicatif supplémentaire des progrès n’est implémenté.

## Structure du code

| Chemin | Fonction |
| --- | --- |
| `public/engine.js` | Génération et correction des exercices |
| `public/app.js` | Parcours, statistiques, exports et synchronisation |
| `public/index.html`, `public/style.css` | Interface et styles adaptatifs |
| `app/api/family/route.ts` | API protégée des profils, séances et réglages |
| `app/chatgpt-auth.ts` | Intégration de l’identité fournie par Sites |
| `lib/family.ts` | Validation et règles d’accès |
| `db/schema.ts`, `drizzle/` | Schéma et migrations |
| `tests/check.cjs`, `tests/server.cjs` | Vérifications du moteur et de l’API |

## Installation et vérification

Le développement utilise Node.js (au moins 22.13), pnpm et la version du gestionnaire indiquée dans `package.json`. Le navigateur des enfants n’a besoin d’aucune installation.

Dans une copie du code complet :

```sh
pnpm install
node tests/check.cjs
node tests/server.cjs
pnpm exec tsc --noEmit
pnpm run build
```

Les tests couvrent notamment les calculs, les fractions, les tentatives, la persistance, les restrictions parent/enfant et la lecture par le parent d’une séance sauvegardée par un enfant.

Le déploiement complet dépend d’une base D1, de ses migrations et d’une authentification compatible. Un simple hébergement de fichiers statiques ne suffit pas à fournir le partage des résultats entre appareils. Les adresses d’invitation et la configuration d’instance sont à adapter ; les paramètres de la famille réelle n’ont pas vocation à figurer dans le dépôt public.

## Calcul des statistiques

La réussite au premier essai correspond aux réponses justes dès la première validation, divisées par les questions travaillées. Une question passée compte comme non réussie ; une question non abordée est exclue. La réussite après essais compte chaque question au plus une fois. La consultation d’un indice est enregistrée, sans retrait de point.

## Limites connues

- Les durées sont indicatives, sans chronomètre imposé.
- Éviter de modifier la même séance simultanément sur plusieurs appareils ; la version la plus récente est conservée.
- Une coupure réseau peut retarder l’arrivée des résultats sur l’ordinateur du parent.
- Le contrôle des rôles a été testé dans l’API ; il ne constitue pas un audit de sécurité de toute la plateforme.
