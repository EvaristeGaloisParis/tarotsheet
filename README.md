# 🃏 TarotSheet

Feuille de score simple pour le **tarot français** (3 à 5 joueurs).

> **👉 Application en ligne : https://tarotsheet.onrender.com**

Application web légère, **sans dépendance** (un seul fichier `index.html`), qui fonctionne
**hors-ligne** et sauvegarde automatiquement sur l'appareil. Pensée pour tenir les scores
rapidement pendant une longue soirée, et permettre aux autres joueurs de **suivre la grille
en direct sur leur téléphone**.

## ✨ Fonctionnalités

- **Grille de score** par prénom (pas de « joueur 1, 2… ») :
  - preneur, contrat (Petite ×1 / Garde ×2 / Garde Sans ×4 / Garde Contre ×6),
  - nombre de bouts avec rappel des points à faire (56 / 51 / 41 / 36),
  - points de l'attaque avec **contrôle automatique de la défense** (total = 91),
  - **poignée** (simple / double / triple) avec rappel du nombre d'atouts selon le nombre de joueurs,
  - **petit au bout** (attaque / défense),
  - **misères** de tête et d'atout (prime forfaitaire).
- **Partenaire (appelé)** géré à 5 joueurs, y compris le cas « seul ».
- **Formule de calcul affichée** sur chaque donne (ex. `2×(25+3) + 20 = 76 /déf.`).
- **Résumé** : graphe d'évolution des scores, classement en direct, statistiques par joueur
  (prises, réussites, taux, points totaux, meilleure/pire donne).
- **Renommer un joueur** à tout moment (se répercute sur toutes les donnes).
- **Export / import** des parties en JSON.
- **Partage en direct** optionnel via Firebase : lien + QR code, vue spectateur en lecture seule.

> Règles de score : **FFT standard**. Le chelem n'est pas géré (volontairement).

## 🎮 Utilisation

1. Ouvrir l'application.
2. Saisir les prénoms des joueurs (3 à 5) et démarrer la partie.
3. Bouton **＋ Nouvelle donne** pour chaque donne jouée.
4. Tout est enregistré automatiquement. Sur mobile : « Ajouter à l'écran d'accueil » pour un usage type appli.

## 📡 Partage en direct (optionnel)

La synchronisation temps réel utilise **Firebase Realtime Database** :

- le teneur de score active **⚙ Réglages → 📡 Partager cette partie** ;
- il obtient un **lien + QR code** à donner aux autres joueurs ;
- les autres ouvrent le lien et voient la grille **se mettre à jour en direct** (lecture seule).

Sans configuration Firebase, l'application reste **100 % locale** (aucune synchronisation).
La configuration se fait dans le bloc `FIREBASE_CONFIG` en haut du script de `index.html` :

```js
const FIREBASE_CONFIG = {
  databaseURL: "https://<ton-projet>-default-rtdb.<region>.firebasedatabase.app",
};
```

Règles Realtime Database utilisées :

```json
{
  "rules": {
    "rooms": { "$code": { ".read": true, ".write": true } }
  }
}
```

## 🚀 Déploiement

C'est un site **statique** (un seul fichier) — hébergeable gratuitement partout :

- **Render (Static Site)** : dépôt GitHub → New Static Site → *Build Command* vide, *Publish Directory* `.`.
- ou Netlify, GitHub Pages, Cloudflare Pages…

> La synchronisation Firebase nécessite une vraie URL `https://` (elle ne fonctionne pas en
> ouvrant le fichier en local).

## 🛠️ Technique

- HTML / CSS / JavaScript purs, **un seul fichier**, aucune étape de build.
- Données locales en `localStorage`, graphe dessiné en `<canvas>` (sans bibliothèque).
- Synchronisation temps réel via le SDK Firebase chargé par CDN (facultatif).
