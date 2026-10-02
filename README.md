*[English version: [README.en.md](README.en.md)]*

# XPackager

Fabrique des **installateurs macOS** — des `.pkg` signés, notarisés et agrafés — à partir
d'un projet que l'on règle dans une interface, sans écrire une ligne de `pkgbuild` ni de
`productbuild`.

Cible **macOS 15** et au-delà.

Ce dépôt porte la **documentation** et les **versions distribuées**. Le source de
l'application n'y figure pas.

---

## Télécharger

Tout est dans la [dernière version](../../releases/latest) :

| Fichier | Ce que c'est |
|---|---|
| `XPackager.pkg` | L'application et, en option, son outil en ligne de commande |

Le paquet s'installe d'un double-clic. Il est lui-même fabriqué par XPackager, signé et
notarisé — c'est sa propre démonstration.

## Ce qu'il fait

- **Le contenu à installer** se compose par glisser-déposer depuis le Finder : une `.app`,
  un dossier avec son arborescence, un outil en ligne de commande. Permissions ajustables
  par élément.
- **Plusieurs composants** donnent une installation personnalisée, avec ses cases à cocher,
  ses choix présélectionnés et ses scripts d'avant et d'après installation.
- **Les prérequis** bloquent l'installation hors de leur domaine : version minimale de
  macOS, architectures autorisées, présence d'un fichier, mémoire disponible.
- **Les quatre écrans de l'installateur** — Bienvenue, Lisez-moi, Licence, Conclusion — se
  rédigent dans un éditeur riche, ou s'importent depuis un `.rtf` existant. Image de fond
  comprise.
- **Un installateur multilingue** se décrit dans le même projet : chaque écran, le titre et
  les intitulés de choix se traduisent langue par langue, avec une langue de référence qui
  sert de secours et un indicateur d'avancement des traductions.
- **La signature et la notarisation** se font dans la foulée de la construction. Le contenu
  est re-signé en Hardened Runtime sur une copie — les originaux ne sont pas touchés — puis
  le paquet part chez Apple et revient avec son ticket agrafé.
- **Un outil en ligne de commande**, `xpackagerbuild`, construit le même paquet à partir du
  même fichier de projet, pour un script de livraison ou une intégration continue.

## Premiers pas

Le [guide](GUIDE.md) part de zéro : l'application vient d'être téléchargée et n'a jamais été
ouverte. Il couvre la feuille de vérification du premier démarrage, l'obtention des
certificats chez Apple — la partie longue, et celle où l'on se trompe le plus —,
l'enregistrement d'un profil de notarisation, puis un exemple complet jusqu'au paquet
vérifié.

## Avec quoi c'est fait

XPackager est écrit en **Xojo**, au-dessus de **[VDSTools](https://github.com/Valdemar-VDSC/VDSTools-dist)**
— chrome de fenêtre, barre latérale, barre d'outils et contrôles macOS natifs, en Xojo pur.

## Assistance

Une question, une anomalie : **support@vdsc.fr**, ou les
[tickets](../../issues) de ce dépôt.
