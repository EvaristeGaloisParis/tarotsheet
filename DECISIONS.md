# TarotSheet — Journal des décisions

Document de référence retraçant le **but**, les **décisions fonctionnelles**, les **choix
techniques**, les **services externes** et les **pistes écartées** du projet.

---

## 1. Le but (contexte d'origine)

Besoin exprimé : lors de longues soirées de tarot (3, 4 ou 5 joueurs, parties de 8-9 h),
une seule personne sait tenir les scores, et la fatigue rend difficile de jouer **et** compter
en même temps. On veut donc :

1. **Tenir les scores simplement** — une grille où l'on saisit chaque donne (preneur choisi par
   **prénom**, contrat, bouts, poignée, petit au bout, points de l'attaque), avec attribution
   automatique des points.
2. **Un résumé** — graphe d'évolution (temps → points) et statistiques par joueur.
3. **De l'interactivité** — les autres joueurs peuvent **suivre la grille sur leur téléphone**.

Hors périmètre initial : chelem, misères. (Les misères ont été **ajoutées ensuite** à la demande.)

---

## 2. Décisions fonctionnelles

- **Choix du preneur par prénom** (gros boutons colorés), pas « joueur 1/2/3 ».
- **Règles de score : FFT standard.**
  - Points à faire selon les bouts : 0→56, 1→51, 2→41, 3→36.
  - Multiplicateurs : Petite ×1, Garde ×2, Garde Sans ×4, Garde Contre ×6.
  - Formule par donne (par défenseur) : `mult × ((25 + écart)·signe + petit_au_bout) + poignée`.
  - Total des points du jeu = 91 → **la défense est affichée automatiquement** (91 − attaque)
    pour le contrôle croisé. Somme des points de tous les joueurs = 0 (vérifié par tests).
- **Poignée** : simple / double / triple (+20 / +30 / +40, non multipliées, vont au camp gagnant),
  avec **rappel du nombre d'atouts** selon le nombre de joueurs (3 j : 13/15/18 ; 4 j : 10/13/15 ;
  5 j : 8/10/13). Comptes FFT confirmés par l'utilisateur.
- **Petit au bout** : attaque / défense / aucun (±10 dans la parenthèse, donc multiplié).
- **5 joueurs** : gestion de l'**appelé** (partenaire) ou « seul ».
- **Misères** (ajout ultérieur) : misère de **tête** et misère d'**atout**, prime forfaitaire de
  **10 pts par misère**, payée par **chacun des autres joueurs** au porteur, **indépendante du
  résultat** et **non multipliée**. Section repliable (rare) pour ne pas encombrer.
- **Chelem** : volontairement **non géré**.
- **Effectif fixe** : une partie démarre avec N joueurs (3 à 5) qui jouent **toutes** les donnes
  jusqu'à la fin. (Décision de l'utilisateur : on a retiré la sélection « qui ne joue pas cette
  donne » qui existait au début.)
- **Prénoms** : capitalisation automatique (1re lettre de chaque mot en majuscule, gère les
  composés « jean-pierre » → « Jean-Pierre »), à la création et au renommage.
- **Renommer un joueur** à tout moment (corrige une faute de frappe) → se répercute sur **toutes**
  les donnes (preneur, appelé, participants, misères), sans perte de scores.
- **Formule de calcul affichée** sur chaque donne du résumé, sur la même ligne que la description
  (ex. `2×(25+3) + 20 = 76 /déf.`).
- **Résumé** : graphe cumulé (une courbe/couleur par joueur), classement en direct, et **stats
  détaillées** (prises, réussites, taux, points totaux, meilleure/pire donne).
- **Export / import JSON** pour sauvegarde/transfert.

---

## 3. Choix techniques

- **Application web en un seul fichier `index.html`**, **sans dépendance** ni étape de build :
  ouvrable partout (téléphone / laptop / tablette), déposable sur n'importe quel hébergeur statique.
  Motivation : « besoin rapide », robustesse, fonctionne hors-ligne.
- **Vanilla JS + CSS**, pas de framework.
- **Persistance locale** via `localStorage` (sauvegarde automatique à chaque action).
- **Graphe** dessiné à la main en `<canvas>` (avec gestion du `devicePixelRatio`), **sans
  bibliothèque** de charting.
- **Design mobile-first**, thème sombre (confort sur 8-9 h), grandes cibles tactiles, saisie de
  donne en *bottom sheet* avec aperçu du résultat en direct.
- **Local d'abord** : l'app est pleinement fonctionnelle sans aucun réseau ; la synchronisation est
  une couche **optionnelle** ajoutée par-dessus (si Firebase non configuré → 100 % local).

### Architecture de synchronisation
- Modèle **teneur / spectateur** :
  - le **teneur** (celui qui compte) écrit et **pousse** l'état complet vers la base ;
  - les **spectateurs** ouvrent un lien `?room=<code>` et voient la grille **en lecture seule**,
    mise à jour en temps réel.
- **Salle** identifiée par un **code** court (ex. `T4821`) ; lien partagé + **QR code**.
- État poussé = l'objet complet de la partie (JSON), stocké sous `rooms/<code>`.

---

## 4. Services externes utilisés

| Service | Rôle | Détails |
|---|---|---|
| **Firebase Realtime Database** (Google) | Synchronisation temps réel | `databaseURL` codé dans `FIREBASE_CONFIG` de `index.html`. Région **europe-west1**. Règles : `rooms/$code` en lecture/écriture publiques. Palier **gratuit** (Spark). SDK chargé par **CDN** (`firebase-app-compat` + `firebase-database-compat` 10.12.2). |
| **Render** (Static Site) | Hébergement du fichier | https://tarotsheet.onrender.com — *Build Command* vide, *Publish Directory* `./`, branche `main`. Auto-déploiement depuis GitHub. Les sites statiques Render **ne se mettent pas en veille**. |
| **GitHub** | Dépôt de code | https://github.com/EvaristeGaloisParis/tarotsheet — `index.html`, `README.md`, `.gitignore` à la racine. Push via **HTTPS** (identifiants Windows mémorisés ; la clé SSH n'est pas autorisée). |
| **api.qrserver.com** | Génération du QR code du lien de partage | Appel d'image à la volée (le lien de partage n'est pas secret). |

---

## 5. Pistes écartées (et pourquoi)

- **Serveur temps réel toujours actif sur Render (Web Service)** : le palier gratuit se met en
  veille (démarrage à froid) et/ou est perçu comme payant → écarté au profit d'une base
  *serverless* sans serveur à maintenir.
- **Supabase** (au lieu de Firebase) : équivalent, mais le **projet gratuit se met en pause après
  ~1 semaine d'inactivité** (gênant pour un usage par week-ends espacés) et demande un schéma de
  table. Firebase Realtime DB, sans schéma et sans mise en pause, colle mieux au besoin
  « un blob JSON synchronisé ».
- **Serveur local en WiFi** (l'app tourne sur un appareil, les autres se connectent en réseau
  local) : envisagé, mais contrainte « même réseau + appareil allumé » ; on a préféré une URL
  publique.
- **Sélection des joueurs par donne** (donneur qui saute) : implémentée au début puis **retirée**
  sur décision de l'utilisateur (effectif fixe pour toute la partie).

---

## 6. Non fait / pistes futures

- Gestion du **chelem**.
- **Historique multi-parties** et statistiques inter-parties.
- **Durcissement de la sécurité** des salles (codes plus longs, expiration, séparation
  lecture/écriture, éventuellement authentification).
- Réglage en interface de la **valeur des misères** (aujourd'hui la constante `MISERE_PRIME`).

---

## 7. Repères de vérification

- Déploiement : `curl -s https://tarotsheet.onrender.com/ | grep firebasedatabase.app`
- Base joignable (REST) : `curl https://tarotsheet-default-rtdb.europe-west1.firebasedatabase.app/rooms/_healthcheck.json` → doit renvoyer `null`.
- Pour livrer une évolution : éditer `index.html` → `git add/commit/push origin main` → Render redéploie.
