# Note

Bloc-note ultra simple synchronisé entre appareils (façon Framapad), en PWA.
Trois onglets (trois notes), un bouton **Effacer**, un bouton **Copier**. Aucune authentification.

GitHub Pages ne sert que des fichiers statiques : la synchronisation passe par une
**Firebase Realtime Database** (gratuite), interrogée directement en REST + flux temps réel,
sans SDK.

## Mise en route (une seule fois)

1. Sur <https://console.firebase.google.com>, créer un projet (Analytics inutile).
2. Menu **Build › Realtime Database › Créer une base de données** (région Europe, mode verrouillé).
3. Onglet **Règles** : coller le contenu de `database.rules.json`, puis **Publier**.
4. Copier l'URL de la base (ex. `https://mon-projet-default-rtdb.europe-west1.firebasedatabase.app`)
   dans `config.js`.
5. Sur GitHub : **Settings › Pages › Deploy from a branch** → branche `main`, dossier `/ (root)`.

L'appli est alors sur <https://vian31.github.io/note/> ; « Ajouter à l'écran d'accueil » pour l'installer.

## Lien secret

Au premier lancement, l'appli propose de créer un bloc-notes : une clé aléatoire est générée
et placée dans l'adresse (`…/note/#clé`). Le bouton 🔗 affiche ce lien, à ouvrir sur les
autres appareils (ou à coller dans la fenêtre d'accueil). Sans la clé, impossible de lire les
notes : les règles Firebase interdisent de lister la base.

⚠️ Toute personne qui possède le lien peut lire et modifier les notes : ne le partagez pas.
