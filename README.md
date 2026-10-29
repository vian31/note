# Note

Bloc-notes synchronisé entre appareils (façon Framapad), en PWA, pensé pour préparer des prompts
sur un PC et les coller dans Claude sur le téléphone (ou l'inverse) : texte, listes, code et
captures d'écran. Aucune authentification : un lien secret donne accès au bloc-notes.

<https://vian31.github.io/note/> — « Ajouter à l'écran d'accueil » pour l'installer.

GitHub Pages ne sert que des fichiers statiques : la synchronisation passe par une
**Firebase Realtime Database** (gratuite), interrogée directement en REST, sans SDK.

## Utilisation

- **Onglets** : trois notes ; toucher l'onglet affiché pour le renommer.
- **Barre** : mise en forme `•` `1.` `</>` `🖼` Balise Modèle, puis actions Effacer Copier Coller
  (deux rangées sur téléphone, une seule à gauche/droite sur PC) ; ⚙ réglages et point d'état
  en haut. Sur PC, colonne centrée.
- **Listes** : `- ` et `1. ` (texte brut Markdown, que Claude comprend). Deux niveaux au plus :
  1er appui sur `1.` → `1.`, 2e → `a)`, 3e → plus de liste (idem `•` → `-`, `•`, rien).
  Entrée continue la liste (numéro suivant) ; Entrée sur un élément vide en sort ; Tab / Maj+Tab
  décalent ; Retour arrière juste après la puce remonte d'un niveau. Insérer ou décaler un élément
  renumérote les suivants. Les lignes commençant par `* `, `+ `, `• `, `– `, `1) `, `a) ` sont
  aussi reconnues.
- **Code** : bouton `</>` (balises ``` autour de la sélection ou de la ligne ; dans un bloc, les
  retire) et détection automatique au collage. Affiché plus petit, en police à chasse fixe et en
  couleur ; pas de listes automatiques dans le code ; Tab insère 2 espaces, Entrée garde le retrait.
- **Balise** : entoure la sélection (ou la ligne) de `<nom>…</nom>` : un nom à taper ou une
  suggestion (document, exemple, contexte, log, email). Sert à délimiter un texte collé (document,
  log, e-mail…) pour qu'il ne soit pas confondu avec vos consignes. Dans une balise, le bouton
  propose aussi de la retirer. Les balises s'affichent en violet.
- **Modèle** : insère un squelette de prompt avec des titres `##` (Prompt complet : Contexte, Tâche,
  Contraintes, Format de réponse attendu ; Texte à traiter ; Avec exemple). Les titres et balises
  **laissés vides ne sont pas copiés** (Copier indique « n parties vides ignorées »).
- **Images** : Ctrl+V d'une capture (Win+Maj+S), glisser-déposer, Coller, ou 🖼 (galerie /
  appareil photo). Vignettes synchronisées ; toucher une vignette : agrandir, copier, supprimer.
- **Copier** : texte nettoyé (sans éléments de liste vides ni espaces en fin de ligne ; le code est
  laissé tel quel). Avec des images, en plusieurs fois, car Claude ne prend qu'un contenu à la fois
  dans le presse-papiers : le texte, puis le bouton devient `🖼 1/2`, `🖼 2/2`… et chaque appui
  copie l'image suivante. Sur téléphone, **Partager tout** envoie texte et images d'un coup.
- **Coller** : texte au curseur, images ajoutées à l'onglet.
- **Effacer** : texte et images de l'onglet, sur tous les appareils.
- **Réglages ⚙** : thème (auto, clair, sombre), lien secret, **Supprimer les images** (toutes,
  texte conservé), **Tout effacer** (notes, noms et images du bloc-notes).

Un texte collé n'est jamais interprété (les lignes `* `, `- `, `1.` d'un code collé restent
intactes), y compris avec les claviers qui envoient un collage ligne par ligne.

## Mise en route (une seule fois)

1. Sur <https://console.firebase.google.com>, créer un projet (Analytics inutile).
2. Menu **Build › Realtime Database › Créer une base de données** (région Europe, mode verrouillé).
3. Onglet **Règles** : coller le contenu de `database.rules.json`, puis **Publier**.
4. Copier l'URL de la base (ex. `https://mon-projet-default-rtdb.europe-west1.firebasedatabase.app`)
   dans `config.js`.
5. Sur GitHub : **Settings › Pages › Deploy from a branch** → branche `main`, dossier `/ (root)`.

## Lien secret

Au premier lancement, l'appli propose de créer un bloc-notes : une clé aléatoire est générée
et placée dans l'adresse (`…/note/#clé`). ⚙ Réglages affiche ce lien, à ouvrir sur les autres
appareils (ou à coller dans la fenêtre d'accueil). Sans la clé, impossible de lire les notes :
les règles Firebase interdisent de lister la base.

⚠️ Toute personne qui possède le lien peut lire et modifier les notes : ne le partagez pas.

## Synchronisation

- Flux en direct Firebase pour l'onglet affiché, plus une lecture toutes les 2 s (certains proxys
  d'entreprise retiennent le flux). Les événements du flux ne servent que de signal : le contenu
  est toujours relu auprès du serveur. Les relectures utilisent `If-None-Match` (« 304 inchangé »).
- Envois conditionnels (ETag) : un appareil ne peut pas écraser une version qu'il n'a pas vue.
  En cas d'écritures simultanées, les modifications sont fusionnées si elles touchent des
  endroits différents ; sinon la version locale l'emporte.
- Le texte non envoyé est conservé sur l'appareil (hors ligne, fermeture) et envoyé dès que possible.
- Images : réduites à 1568 px en WebP (~50–120 Ko), découpées en morceaux de texte dans Firebase
  (même domaine et même format que les notes : passe les proxys) ; après un échec, morceaux
  deux fois plus petits. Un petit index par onglet (avec vignettes) est synchronisé ; les images
  complètes sont téléchargées une fois puis gardées sur l'appareil.

## Technique

- **Éditeur** : [CodeMirror 6](https://codemirror.net) (licence MIT, voir
  `lib/LICENSE-codemirror.txt`), empaqueté dans `lib/codemirror.js` et servi par le même site
  (rien à autoriser sur un proxy). Ctrl+Z ne défait jamais une modification venue d'un autre
  appareil ; historique par onglet.
- **Règles Firebase** (`database.rules.json`) : notes `notes/<clé>/{1,2,3}`, noms `n1…`, index
  d'images `i1…` ; images `images/<clé>/<id>/<morceau>`. Sur un bloc-notes entier, seule la
  suppression est autorisée.
- **Page de test** : <https://vian31.github.io/note/test.html>, même code avec ses données rangées
  à part (`t1…`, `tn1…`, `ti1…`), pour essayer une nouveauté sans toucher aux vraies notes.
