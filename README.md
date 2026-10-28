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

Le texte collé n'est jamais interprété : un collage, ou un clavier qui envoie le texte ligne par ligne,
laisse le code intact (les lignes `* `, `- `, `1.` d'un commentaire ou d'un YAML ne sont pas transformées).
La liste n'est continuée que 0,15 s après un Entrée tapé à la main, si rien d'autre n'a été saisi entre-temps.

Boutons **• Puces**, **1. Liste**, **⇥ ⇤** (décaler). Au clavier : Entrée continue la liste (numéro suivant),
Entrée sur un élément vide en sort, Tab / Maj+Tab décalent. C'est du texte brut en syntaxe Markdown
(`- ` et `1. `, retrait de 2 espaces) : il se colle tel quel dans un prompt Claude. **Copier** retire en plus
les éléments de liste vides et les espaces en fin de ligne.

## Synchronisation

- Flux en direct Firebase pour l'onglet affiché, plus une lecture toutes les 2 s
  (certains proxys d'entreprise retiennent le flux).
- Les événements du flux ne servent que de signal : le contenu est toujours relu auprès du serveur
  (un écho en retard ne peut plus être pris pour une modification d'un autre appareil).
  Les relectures utilisent `If-None-Match` : réponse « 304 inchangé » sans retélécharger le texte.
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

## Page de test (`test.html`)

Nouvelle version en essai sur <https://vian31.github.io/note/test.html>, avec le même lien secret.
Ses notes, noms et images sont rangés à part (`t1…`, `tn1…`, `ti1…`, `images/`) : l'appli
normale n'est pas touchée. Il faut publier les règles de `database.rules.json` (elles gardent
celles de l'appli normale).

- **Barre unique** : `•` `1.` `</>` `🖼` Effacer Copier Coller (⚙ réglages et point d'état en haut).
  Deux niveaux au plus : 1er appui sur `1.` → `1.`, 2e → `a)`, 3e → plus de liste (idem `•` → `-`,
  `•`, rien). Tab ne crée pas de 3e niveau. Retour arrière juste après la puce remonte d'un niveau.
- **Images** : Ctrl+V d'une capture (Win+Maj+S), glisser-déposer, bouton Coller, ou bouton 🖼
  (galerie / appareil photo : sur téléphone, les captures ne passent pas par le presse-papiers). Réduites à
  1568 px, en WebP (~50–120 Ko), découpées en morceaux de texte dans Firebase (même domaine, même
  format que les notes : passe les proxys) ; si un envoi échoue, les morceaux sont réduits de
  moitié au nouvel essai. Vignettes synchronisées ; toucher une vignette : agrandir, copier
  l'image, supprimer.
- **Copier** : en plusieurs fois quand il y a des images (Claude ne prend qu'un contenu à la fois
  dans le presse-papiers) : le texte, puis le bouton devient `🖼 1/2`, `🖼 2/2`… et chaque appui
  copie l'image suivante. Sur téléphone, **Partager tout** envoie texte et images d'un coup.
- **Coller** : texte au curseur, images ajoutées à l'onglet.
- **Code** : bouton `</>` (balises ``` autour de la sélection ou de la ligne ; dans un bloc, les
  retire) et détection automatique au collage. Le code s'affiche en couleur (bleu-vert, lignes ```
  atténuées) sur un bandeau ; pas de puces automatiques ni de renumérotation dans le code ; Tab y
  insère 2 espaces. Technique : la zone de texte devient transparente (curseur et sélection
  gardés) et une copie alignée au pixel affiche le texte ; la police reste la même (une taille
  différente décalerait le curseur).
- **Réglages ⚙** : thème (auto, clair, sombre), lien secret, **Supprimer les images** (toutes celles du bloc-notes, texte conservé)
  et **Tout effacer** (notes, noms et images du bloc-notes, appli normale comprise). Les règles
  n'autorisent sur un bloc-notes entier que sa suppression.
