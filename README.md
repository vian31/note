# Note

Bloc-note ultra simple synchronisé entre appareils (façon Framapad), en PWA.
Trois onglets renommables (toucher l'onglet affiché), un bouton **Effacer**, un bouton **Copier**. Aucune authentification.

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

## Puces et numérotation

Détection automatique : une ligne qui commence par `- `, `* `, `+ `, `• `, `– `, `1. `, `1) ` ou `a) `
est reconnue comme élément de liste, et Entrée ouvre l'élément suivant (même repli sur les claviers de
téléphone qui n'autorisent pas l'interception de la touche Entrée).

Un retrait ouvre un sous-niveau d'un autre type (chapitres) : `1.` → `a)` → `1)` pour les numéros,
`-` → `•` → `◦` pour les puces. En remontant d'un niveau, la numérotation reprend à la suite
(`1.`, `a)`, `b)`, puis `2.`). Insérer ou décaler un élément renumérote les suivants.

Boutons **• Puces**, **1. Liste**, **⇥ ⇤** (décaler). Au clavier : Entrée continue la liste (numéro suivant),
Entrée sur un élément vide en sort, Tab / Maj+Tab décalent. C'est du texte brut en syntaxe Markdown
(`- ` et `1. `, retrait de 2 espaces) : il se colle tel quel dans un prompt Claude. **Copier** retire en plus
les éléments de liste vides et les espaces en fin de ligne.

## Synchronisation

- Flux en direct Firebase pour l'onglet affiché, plus une lecture toutes les 2 s
  (certains proxys d'entreprise retiennent le flux).
- Envois conditionnels (ETag) : un appareil ne peut pas écraser une version qu'il n'a pas vue.
  En cas d'écritures simultanées, les modifications sont fusionnées si elles touchent des
  endroits différents ; sinon la version locale l'emporte.
- Le texte non envoyé est conservé sur l'appareil (hors ligne, fermeture) et envoyé dès que possible.

## Lien secret

Au premier lancement, l'appli propose de créer un bloc-notes : une clé aléatoire est générée
et placée dans l'adresse (`…/note/#clé`). Le bouton 🔗 affiche ce lien, à ouvrir sur les
autres appareils (ou à coller dans la fenêtre d'accueil). Sans la clé, impossible de lire les
notes : les règles Firebase interdisent de lister la base.

⚠️ Toute personne qui possède le lien peut lire et modifier les notes : ne le partagez pas.
