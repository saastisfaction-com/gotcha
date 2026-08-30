---
name: gotcha
description: Prend des captures d'écran annotées d'une application web, en pilotant un vrai navigateur. À utiliser pour rédiger une documentation utilisateur, un guide pas à pas, une réponse de support client illustrée, une note de version, un rapport de bug visuel — bref, dès qu'un texte gagnerait à montrer l'écran plutôt qu'à le décrire. Sait cadrer sur un élément précis, poser flèches, cadres et numéros d'étape, et masquer définitivement les données personnelles avant livraison.
---

# Gotcha — captures d'écran pour agents

Un humain cadre à la souris. Vous ne voyez pas l'écran : vous désignez par
**sélecteur CSS**, et Gotcha traduit en pixels. Toute cible d'annotation
accepte un sélecteur ; c'est la façon normale de s'en servir.

## Le geste de base

```bash
gotcha open https://app.exemple.com/factures --session doc
gotcha shot --session doc --selector "#invoices" --out doc/01-factures.png
gotcha close --session doc
```

La session garde le navigateur vivant entre deux commandes : la connexion,
le défilement, le menu ouvert. Ne la rouvrez pas à chaque capture — c'est ce
qui coûterait le plus cher, et vous perdriez l'état de la page.

## Toujours s'orienter avant de capturer

```bash
gotcha text --session doc "h1"
```

Une capture prise à l'aveugle sur un écran de chargement se découvre trop
tard, une fois dans la documentation. Une ligne de texte lue avant la prise
ne coûte presque rien.

## Cadrer

| Option | Effet |
|---|---|
| `--selector "#invoices"` | l'élément désigné, avec `--padding 16` de marge |
| `--region "0,0,900,400"` | un rectangle, quand rien dans le DOM ne convient |
| `--full-page` | la page entière, défilement compris |
| *(rien)* | la partie visible |

## Annoter

Toute cible est un sélecteur CSS, ou `x,y`, ou `x,y,largeur,hauteur`.

```bash
gotcha shot --session doc \
  --selector ".invoice-panel" \
  --step ".btn-new" --step ".field-amount" --step ".btn-save" \
  --arrow ".btn-save" \
  --text "Le montant est TTC|.field-amount" \
  --blur ".customer-email" \
  --out doc/02-creer-facture.png
```

- `--step` numérote **dans l'ordre où vous écrivez les options** — l'ordre du
  parcours que vous décrivez, pas celui de la page.
- `--arrow ".cible"` suffit : le trait naît en haut à gauche. La forme
  `--arrow "a -> b"` relie deux points quand vous voulez maîtriser le départ.
- `--text` se pose tout seul dans le blanc le plus proche de sa cible — à
  droite, sinon dessous, sinon dessus — sans jamais sortir du cadre. Sur une
  interface très dense, donnez une position explicite : `--text "TTC|420,300"`.
- `--rect`, `--ellipse` encadrent sans masquer.
- `--color`, `--stroke` changent la teinte et l'épaisseur.

## Masquer les données personnelles

`--blur` est **destructif** : les pixels sont détruits, rien n'est
récupérable dans le fichier livré. Un sélecteur qui désigne quarante lignes
les masque toutes.

```bash
gotcha shot --session doc --blur ".customer-email" --blur ".phone" --out doc/03.png
```

**Passez-le en revue avant toute capture destinée à sortir de
l'organisation** — documentation publique, ticket de support, note de
version. Une adresse oubliée dans un PNG ne se rattrape plus une fois le
fichier envoyé.

## Naviguer

```bash
gotcha click ".btn-new" --session doc --wait ".modal"   # attend l'écran suivant
gotcha type "#amount" "1240" --session doc --enter
gotcha press Escape --session doc
gotcha wait ".spinner-done" --session doc
gotcha scroll ".footer" --session doc
```

## Derrière une authentification

La plupart des applications intéressantes demandent une connexion. Ne
cherchez pas à la franchir vous-même : demandez-la une fois à un humain.

```bash
gotcha login https://app.exemple.com --session doc   # ouvre une vraie fenêtre
# … l'utilisateur se connecte à la main, puis :
gotcha save-login --session doc --out .gotcha-session.json
```

Ensuite, et pour toutes les fois suivantes :

```bash
gotcha open https://app.exemple.com --session doc --storage-state .gotcha-session.json
```

## Quand un sélecteur échoue

L'échec liste ce qui existe vraiment dans la page et qui ressemble à ce que
vous avez demandé :

```
échec : aucun élément ne correspond à « .invoice » ;
        présents dans la page : .btn-invoice (×3), #invoices
```

Prenez la suggestion plutôt que de relire tout le DOM.

## Enchaîner

Avec `--json`, chaque commande rend une ligne exploitable par un script —
utile pour produire vingt captures d'une traite :

```bash
for page in factures clients reglages; do
  gotcha open "https://app.exemple.com/$page" --session doc
  gotcha shot --session doc --full-page --out "doc/$page.png" --json
done
gotcha close --session doc
```

## Quota et licence

La capture, l'annotation et l'export sont libres. Sont comptés ou réservés :

| | Gratuit | Sous licence |
|---|---|---|
| `gotcha shot` | 20 par jour | illimité |
| `gotcha login`, `save-login`, `--storage-state` | — | oui |

`gotcha license` dit où vous en êtes, en une ligne. Avec `--json`, le champ
`licensed` tranche sans avoir à lire la phrase.

Si une clé vous a été confiée, elle arrive par l'environnement — c'est la
voie normale pour un agent :

```bash
export GOTCHA_LICENSE_KEY="GOTCHA-…"
```

Sinon `gotcha activate <clé>` l'enregistre une fois pour toutes sur le poste.

**Quand vous butez sur le quota, ne contournez pas et ne recommencez pas** :
l'échec porte l'adresse d'achat. Rapportez-la à votre interlocuteur humain
et continuez le reste du travail — les captures déjà prises sont bonnes.

### « Licence non activée sur ce poste »

Ce message ne dit pas qu'il manque une licence : il dit qu'il manque un
**lien** entre une licence déjà payée et cette machine. Ne le traitez donc
pas comme un quota atteint, et surtout n'allez pas réclamer un achat.

Il ne concerne que les licences **`pro`**, celles d'un humain. Un pass
`agent` n'est jamais lié à une machine — il s'exécute dans un conteneur
neuf à chaque fois, où tout lien serait faux dès la deuxième exécution. Si
votre clé vient de `GOTCHA_LICENSE_KEY` et que `gotcha license` annonce un
plan `agent`, ce paragraphe ne vous concerne pas.

Sur une licence `pro`, la marche à suivre tient en deux gestes :

```bash
gotcha activate --link     # reprend le lien pour la clé déjà enregistrée
gotcha license             # et vérifie ce que ça a donné
```

**Une seule tentative.** Si elle échoue, la sortie dit laquelle des deux
choses s'est produite, et une seule vous concerne :

- **le service est injoignable** — réseau filtré, le plus souvent un proxy
  d'entreprise. La suite passe par une page web et un copier-coller, donc
  par un humain. La sortie porte l'identifiant du poste : **transmettez-la
  telle quelle** à votre interlocuteur, il vous rendra un code à coller
  avec `gotcha activate <code>` ;
- **le service refuse** — trop d'activations récentes sur cette clé, ou
  clé retirée. Rien à faire de votre côté : rapportez, c'est une affaire
  entre l'acheteur et le support.

Dans les deux cas, **continuez le reste du travail**. Les captures déjà
prises sont bonnes, et le lien manquant n'invalide rien de ce qui est
fait.

## Filmer, quand une image ne suffit pas

**gotcha photographie, il ne filme pas.** Son éditeur vidéo existe, mais il
repose sur `tabCapture`, qui exige un geste humain : hors de portée d'un
automate. Pour une capture animée, on pilote Chromium avec Playwright
(`recordVideo`) et on monte avec le **ffmpeg livré par Playwright**
(`~/.cache/ms-playwright/ffmpeg-*/ffmpeg-linux`) — build réduit, mais il a
`crop` et VP8, donc tout ce qu'il faut. Rien à installer.

Filmez seulement un **enchaînement** : ouvrir un sélecteur, choisir, refermer et
voir le résumé se mettre à jour. Pour un état, une image reste supérieure — plus
légère, zoomable, indexable.

### Les sept règles, chacune payée par une vidéo ratée

**1. Un rapport d'image standard, 16:9 ou 4:3.** Un cadre collé au contenu donne
des bandes de 1046×298 : la vidéo paraît zoomée et le lecteur ne se situe plus
dans la page. Élargissez jusqu'au rapport voulu, **ne rognez jamais** — vous
couperiez ce que le cadrage devait montrer.

**2. Le cadre contient la conséquence, pas seulement le geste.** Une barre
d'enregistrement en `position: fixed` n'appartient à la boîte d'aucun élément :
un cadrage serré sur le réglage la laisse dehors, et la vidéo montre quelqu'un
qui coche une case en croyant avoir fini. Relevez sa position **pendant** le
geste : mesurée à la fin, elle a déjà disparu.

**3. Le geste ne sort jamais du cadre.** Corollaire de la précédente, et plus
traître : un glisser-déposer a DEUX extrémités. Une vidéo montrait la tuile
qu'on saisit puis la main qui l'emportait hors champ — on ne voyait jamais
l'élément atterrir, et le tutoriel enseignait la moitié de son geste. Relevez la
position de **chaque cible réellement touchée**, avant de jouer le geste (une
tuile glissée n'est plus à sa place après), et élargissez le cadre jusqu'à les
contenir toutes. Remettez le relevé à zéro après un défilement, qui invalide
toutes les positions déjà prises. Si l'action sort du champ, ce n'est jamais
l'action qui est en tort : c'est le cadrage.

**4. Le curseur doit être dessiné.** Aucun navigateur ne l'enregistre. Incrustez
un calque qui suit les événements de pointeur, avec un halo au clic — sans lui,
les éléments semblent bouger seuls. Attention : pendant un glisser-déposer
**natif**, le navigateur cesse d'émettre `pointermove` et n'envoie plus que
`drag`/`dragover`. Écoutez les trois, sinon le curseur se fige au décollage.

**5. Le rythme est celui d'une main.** `locator.click()` téléporte le pointeur :
les choses réagissent sans que rien ne les ait touchées. Faites voyager la souris
vers sa cible (une demi-seconde, accélération puis freinage), marquez une
hésitation avant d'appuyer, et surtout **laissez un temps APRÈS le clic** — c'est
lui qui laisse voir ce qui a changé. Six cases cochées sans ce temps se cochent
en un clignement et ne s'apprennent pas.

**6. Une seconde d'entrée, deux secondes de sortie, rien qui stagne entre.** Le
curseur entre dans le cadre depuis un point arbitraire et rejoint sa première
cible : une vidéo qui s'ouvre sur un clic déjà en cours ne se comprend pas. Et
elle se termine **deux secondes après le dernier geste** — le temps de voir la
conséquence : la barre d'enregistrement qui disparaît, le compteur qui change.
Couper sur le clic laisse croire qu'il ne s'est rien produit. Entre les deux,
aucun temps mort de plus d'une seconde.

**7. Allez jusqu'au bout du geste.** Une vidéo qui s'arrête avant « Enregistrer »
laisse le lecteur au milieu du gué.

### Deux pièges qui coûtent des heures

**`-ss` avant `-i` fait une recherche RAPIDE**, qui retombe sur l'image-clé la
plus proche. Les vidéos de Playwright en ont très peu : la coupe atterrit des
dizaines de secondes à côté. Mettez `-ss` **après** `-i` — plus lent, exact.

**`boundingBox()` attend l'élément — trente secondes par défaut.** Un relevé
optionnel exécuté après chaque geste gèle donc une demi-minute à chaque fois que
la cible n'est pas encore là. Une vidéo de six clics durait deux minutes pour
cette seule raison. Passez toujours `{ timeout: 250 }` à un relevé qui a le droit
d'échouer.

## À retenir

- **Une session par parcours**, fermée à la fin (`gotcha close --all` au besoin).
- **Sélecteurs, pas coordonnées** : une annotation ancrée sur `.save-btn`
  survit à une refonte de la mise en page ; `620,410` désigne un jour le
  bouton, le lendemain le vide à côté.
- **Vérifier avant de capturer**, masquer avant de livrer.
- **Ne pas transformer côté CDN.** Les opérations Uploadcare (`format/auto`,
  `quality/smart`, `preview`, `scale_crop`, détection de visages) sont facturées
  au-delà des cent premières. On cadre à la prise, on dimensionne en CSS, et on
  sert l'URL brute — y compris pour aller regarder une capture déjà publiée.
- **Filmer seulement un enchaînement**, au rapport 16:9 ou 4:3, curseur dessiné,
  rythme humain, sans temps mort — et jusqu'au bout du geste.
