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

## À retenir

- **Une session par parcours**, fermée à la fin (`gotcha close --all` au besoin).
- **Sélecteurs, pas coordonnées** : une annotation ancrée sur `.save-btn`
  survit à une refonte de la mise en page ; `620,410` désigne un jour le
  bouton, le lendemain le vide à côté.
- **Vérifier avant de capturer**, masquer avant de livrer.
