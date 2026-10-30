*[English version: [CHANGELOG.en.md](CHANGELOG.en.md)]*

# Journal des versions

## 1.5.0

### Des paquets plus légers d'un tiers

Les composants sont désormais compressés avec l'algorithme le plus récent de `pkgbuild`. Le
paquet de XPackager lui-même passe de 22,0 à 15,1 Mo, pour un contenu identique. L'option est
active par défaut et se règle dans les prérequis. Elle réclame une version minimale de macOS
valide au-dessus : sans elle `pkgbuild` refuse de construire, et XPackager s'abstient alors
d'y toucher plutôt que de faire échouer la construction. La légende sous l'interrupteur le
dit.

### Applications à quitter

Chaque composant accepte une liste d'**applications à quitter** : l'installateur demande de
les fermer avant d'installer. C'est la réponse au remplacement d'une application en cours
d'exécution — le cas de toute mise à jour. Le bouton **Choisir…** ouvre un sélecteur et lit
l'identifiant dans l'application que vous désignez, inutile de le connaître. L'installateur
ne s'en occupe que si ce composant est retenu.

### Un script de désinstallation

**Fichier ▸ Générer un script de désinstallation…** écrit un script `.sh` à joindre à votre
diffusion. macOS n'offre aucun désinstalleur, mais il tient le compte exact de ce que chaque
paquet a posé : le script relit ces reçus, enlève les fichiers et les liens, puis les
dossiers **seulement s'ils sont restés vides**, et oublie les reçus. Les dossiers partagés —
`/Applications`, `/usr/local/bin`, `~/Library/Preferences` et leurs semblables — sont épargnés
même vides. Un essai à blanc affiche ce qui partirait sans rien toucher, et le script refuse
de s'exécuter sans `sudo`.

### Le refus d'Apple, en clair

Quand la notarisation échoue, la fenêtre de construction affichait le JSON brut renvoyé par
`notarytool`. Elle en donne maintenant un résumé : la raison en une phrase, puis chaque
problème avec ses emplacements et le lien vers la page d'Apple qui l'explique. Les problèmes
identiques sont regroupés — Apple répète la même phrase pour chaque binaire et chaque
architecture, ce qui transformait un seul défaut en deux cents lignes.

### Les téléchargements mis en quarantaine

XPackager pose désormais la marque de quarantaine sur le paquet qu'il télécharge, comme le
fait un navigateur. C'est à l'application qui télécharge de la poser ; sans elle, macOS
n'examine rien à l'ouverture. Le paquet est donc contrôlé deux fois : par l'analyse XProtect
et Gatekeeper si vous l'ouvrez vous-même depuis le Finder, et par XPackager lui-même — signé,
notarisé, signé par le bon éditeur — avant qu'il ne soit remis à l'installateur. La marque ne
traverse pas le paquet : l'application installée n'en hérite pas, il n'y a rien à nettoyer
ensuite.

Merci à Arnaud Nicolet de l'avoir signalé sur le forum Xojo. Les notes de la 1.4.0
présentaient le contrôle interne comme le seul recours possible : poser la marque ne coûte
rien, et macOS en fait plus que vérifier une signature.

### Le volet de détail défile

Les quatre pages du volet — réglages, composants, prérequis, présentation — défilent quand
elles ne tiennent pas dans la fenêtre. Le minimum de la fenêtre descend de 780 à 600 points :
les pages restent atteignables sur un affichage mis à l'échelle ou dans une fenêtre à
demi-écran.

## 1.4.0

### Rechercher les mises à jour

Une entrée **Rechercher les mises à jour…** dans le menu de l'application interroge la
dernière version publiée, affiche la version installée en regard de la dernière, et donne
ses nouveautés — rendues, et non en Markdown brut.

Le bouton **Télécharger** dépose le paquet dans vos *Téléchargements*, avec une barre de
progression. **Avant de le remettre à l'installateur d'Apple, XPackager le contrôle
lui-même** : un fichier téléchargé par une application ne porte pas la marque de
quarantaine, Gatekeeper ne l'examinerait donc pas à l'ouverture. XPackager vérifie qu'il
est signé, notarisé, et signé par l'éditeur attendu. Si l'un des trois manque, il ne
l'ouvre pas et affiche ce qu'il a trouvé.

La recherche se fait aussi au démarrage, **en silence** : rien ne s'affiche quand vous êtes
à jour. Une case au bas de la fenêtre l'arrête.

## 1.3.0

### Hiérarchie de choix

Un composant peut désormais être un **groupe de choix** : il n'installe rien et réunit sous
une seule case à cocher les composants qui le désignent dans leur menu **Sous**. Les groupes
s'emboîtent, ce qui donne les listes à plusieurs niveaux des installateurs volumineux. Une
parenté qui bouclerait sur elle-même est écartée à la construction plutôt que de produire un
installateur illisible.

### Propriétaire et groupe par élément

Chaque élément du contenu accepte un **propriétaire** et un **groupe**. Laissés vides — le
cas normal — il appartient à `root:wheel`, comme avant. Renseignés, ils sont posés à la fin
de l'installation, sur l'élément et sur tout ce qu'il contient ; un élément placé en dessous
reprend la main sur son propre sous-arbre. De quoi livrer un démon, ou un dossier qu'un
service doit pouvoir écrire.

### Installer sur le volume de démarrage uniquement

Une case dans les prérequis interdit l'installation sur un autre disque et dans le dossier de
départ de l'utilisateur. Indispensable pour tout ce qui vit dans `/Library`.

### Corrections

- Changer de composant rafraîchissait les valeurs de la page sans la reposer : des lignes de
  l'ancien composant restaient à l'écran.
- L'interrupteur « Groupe de choix » n'était pas cliquable — il appartenait au mauvais cadre.
- Deux listes vides s'écrivaient `{}` au lieu de `[]` dans le fichier de projet.

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
