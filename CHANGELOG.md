*[English version: [CHANGELOG.en.md](CHANGELOG.en.md)]*

# Journal des versions

## 1.2.1

Trente-deux libellés n'avaient aucune traduction et s'affichaient en français
quelle que soit la langue du système : le menu **Fichier**, les cinq modèles de projet, la
question d'enregistrement à la fermeture, les trois encarts qui expliquent quel certificat
choisir, et quelques intitulés de la fenêtre des réglages. Ils parlent désormais les six
langues de l'application.

Le guide de prise en main existe aussi en anglais.

## 1.2.0

Première version publique.

### L'essentiel

- Composition du contenu à installer par glisser-déposer, avec permissions par élément.
- Installation personnalisée à plusieurs composants : cases à cocher, choix présélectionnés,
  scripts d'avant et d'après installation.
- Prérequis d'installation : version minimale de macOS, architectures autorisées, présence
  d'un fichier, mémoire disponible.
- Les quatre écrans de l'installateur dans un éditeur riche, image de fond comprise, avec
  import de `.rtf` et de texte brut.
- **Installateur multilingue** : écrans, titre et intitulés de choix traduits langue par
  langue, avec langue de référence obligatoire comme secours, indicateur d'avancement des
  traductions et normalisation des codes régionaux — `pt-BR`, `zh-Hans`.
- Signature `Developer ID Installer`, re-signature du contenu en Hardened Runtime,
  notarisation et agrafage du ticket.
- Outil en ligne de commande `xpackagerbuild` : le même moteur, le même fichier de projet,
  le même paquet.

### Premier démarrage

Une feuille de vérification de l'environnement s'ouvre au premier lancement et reste
rappelable par **Aide ▸ Vérifier l'environnement…**. Elle contrôle les outils d'Apple, les
deux certificats Developer ID et le profil de notarisation — tout ce qui est hors de
l'application et ne se voyait, autrement, qu'au bout d'une construction échouée.

Un certificat de développement y reçoit son propre message : il ressemble à un Developer ID
dans un menu de signature, et Apple refuse la notarisation avec.

### Donationware

XPackager est gratuit. Un bouton de don figure dans la fenêtre **À propos** et dans le menu
**Aide ▸ Faire un don…**

### Notes

- Exige **macOS 15**.
- Le paquet de cette version est fabriqué par XPackager lui-même.
