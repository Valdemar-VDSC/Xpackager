*[English version: [GUIDE.en.md](GUIDE.en.md)]*

# Prise en main de XPackager

Ce guide part de zéro : vous venez de télécharger XPackager, vous ne l'avez jamais ouvert,
et vous voulez au bout du compte un `.pkg` que n'importe qui puisse installer d'un
double-clic sans que macOS s'y oppose.

Il se lit dans l'ordre. La partie longue n'est pas XPackager — c'est Apple.

---

## 1. Installer et ouvrir

Double-cliquez `XPackager.pkg`. L'installateur pose l'application dans `/Applications` et
vous propose, en option, l'outil en ligne de commande `xpackagerbuild`. Prenez-le : il sert
plus tard, pour construire sans ouvrir l'interface.

Au premier lancement, XPackager n'affiche pas son projet vide tout de suite. Il ouvre
d'abord une feuille : **Vérification de l'environnement**.

Elle existe parce que tout ce qui peut vous bloquer est **en dehors** de XPackager — les
outils d'Apple, les certificats de votre trousseau, votre profil de notarisation — et parce
que ces manques, autrement, ne se voient qu'après coup : la construction part, tourne deux
minutes, et échoue à la toute fin sur un refus d'Apple.

Cinq lignes :

| Contrôle | Ce qu'il regarde |
|---|---|
| Outils Xcode | `notarytool` est-il installé |
| Outils de fabrication de paquets | `pkgbuild`, `productbuild`, `installer` |
| Certificat Developer ID Application | signe le logiciel |
| Certificat Developer ID Installer | signe le paquet |
| Profil de notarisation | vos identifiants Apple enregistrés dans le trousseau |

Sur une machine neuve, attendez-vous à cinq croix. C'est normal. Les sections suivantes les
transforment en coches, dans cet ordre.

La feuille reste disponible à tout moment par **Aide ▸ Vérifier l'environnement…**, et le
bouton **Revérifier** la relance sans quitter l'application. Gardez-la ouverte pendant tout
ce qui suit : chaque étape franchie s'y lit immédiatement.

---

## 2. Les outils d'Apple

Si la première ligne est rouge, installez **Xcode** depuis le Mac App Store. C'est gros
(plusieurs gigaoctets) mais c'est le chemin sûr : `notarytool` vit à l'intérieur.

Les outils en ligne de commande seuls suffisent parfois :

```bash
xcode-select --install
```

Après l'installation, **Revérifier**. Les deux premières lignes doivent passer au vert.
`pkgbuild`, `productbuild` et `installer` sont fournis par macOS et devraient déjà être là.

---

## 3. Les certificats

C'est l'étape la plus longue, et celle où l'on se trompe le plus souvent. Prenez le temps.

### 3.1 Le compte

Il faut un compte **Apple Developer Program** payant, renouvelé chaque année. Un identifiant
Apple gratuit ne donne pas droit aux certificats « Developer ID ».

Deuxième condition, souvent oubliée : seul le **titulaire du compte** (Account Holder) peut
créer un certificat Developer ID. Si vous êtes membre d'une équipe sans ce rôle, le bouton
existe mais le type de certificat ne vous sera pas proposé.

### 3.2 Quatre certificats qui se ressemblent

C'est ici que se joue la moitié des échecs de notarisation.

| Certificat | Ce qu'il fait | Notarisable |
|---|---|---|
| **Developer ID Application** | signe une `.app`, un binaire, une dylib | **oui** — c'est celui qu'il vous faut |
| **Developer ID Installer** | signe un `.pkg` distribué hors App Store | **oui** — celui qu'il vous faut aussi |
| Apple Development | débogage local sur vos machines | **non** |
| 3rd Party Mac Developer Installer | soumission au Mac App Store | sans objet |

Le piège : dans un menu de signature, `Apple Development: Votre Nom (XXXXXXXXXX)` et
`Developer ID Application: Votre Nom (XXXXXXXXXX)` se ressemblent beaucoup. Le premier
produit un logiciel qu'Apple **refusera** de notariser. La feuille de vérification le dit
explicitement quand elle ne trouve qu'un certificat de développement.

Vous avez besoin des **deux** Developer ID : l'un pour le logiciel, l'autre pour le paquet.

### 3.3 Les créer — par Xcode

Le chemin le plus court :

1. Xcode ▸ **Settings…** ▸ **Accounts**
2. Ajoutez votre identifiant Apple s'il n'y est pas
3. Sélectionnez votre équipe, puis **Manage Certificates…**
4. Bouton **+** en bas à gauche → **Developer ID Application**
5. Recommencez : **+** → **Developer ID Installer**

Xcode génère la clé privée, demande le certificat à Apple et l'installe dans votre trousseau
de session. Rien d'autre à faire.

### 3.4 Les créer — par le portail

Si vous préférez passer par le site, ou si Xcode refuse :

1. **Trousseau d'accès** ▸ menu *Trousseau d'accès* ▸ *Assistant de certification* ▸
   *Demander un certificat à une autorité de certification…*
2. Saisissez votre adresse de courrier, laissez l'adresse de l'autorité vide, cochez
   **Enregistré sur le disque**. Vous obtenez un fichier `.certSigningRequest`.
3. Sur [developer.apple.com](https://developer.apple.com/account/resources/certificates),
   section **Certificates**, bouton **+**, choisissez **Developer ID Application**, envoyez
   le `.certSigningRequest`, téléchargez le `.cer` obtenu et double-cliquez-le.
4. Recommencez pour **Developer ID Installer**.

Deux avertissements :

- **La clé privée ne quitte jamais la machine qui a produit la demande.** Un certificat
  téléchargé sur un autre Mac ne servira à rien. Pour travailler sur plusieurs machines,
  exportez le couple certificat + clé en `.p12` depuis Trousseau d'accès.
- Le nombre de certificats Developer ID que votre compte peut détenir est **limité**, et
  révoquer un certificat invalide les signatures faites avec. N'en créez pas à la légère et
  ne révoquez rien pour « faire propre ».

### 3.5 Vérifier

Dans XPackager : **Revérifier**. Les deux lignes de certificat passent au vert et affichent
le nom complet trouvé.

Au Terminal, si vous voulez voir la liste brute — c'est exactement ce que lit XPackager :

```bash
security find-identity -v
```

Vous devez y lire deux lignes contenant `Developer ID Application:` et
`Developer ID Installer:`, suivies de votre nom et de votre identifiant d'équipe entre
parenthèses. Notez cet identifiant à dix caractères : c'est votre **Team ID**, il sert à
l'étape suivante.

---

## 4. Le profil de notarisation

Signer ne suffit plus. Depuis macOS 10.15, un logiciel téléchargé doit aussi être
**notarisé** : envoyé à Apple, analysé, puis muni d'un ticket. Sans cela, votre installateur
s'ouvre sur un avertissement qui décourage la plupart des gens.

`notarytool` a besoin de vos identifiants. On ne les tape pas à chaque fois : on les range
une fois pour toutes dans le trousseau, sous un nom — le **profil**.

### 4.1 Un mot de passe spécifique

Votre vrai mot de passe Apple ne convient pas. Il faut un mot de passe dédié :

1. Ouvrez [account.apple.com](https://account.apple.com/account/manage) — XPackager propose
   le lien **Générer un mot de passe spécifique…** dans ses Réglages.
2. Section **Connexion et sécurité** ▸ **Mots de passe pour application**
3. Créez-en un, nommez-le par exemple « XPackager »
4. **Copiez-le tout de suite** : il ne s'affiche qu'une fois, sous la forme
   `abcd-efgh-ijkl-mnop`.

### 4.2 Enregistrer le profil

Dans XPackager : **Réglages** (⌘,) ▸ onglet **Notarisation**.

| Champ | Ce qu'on y met |
|---|---|
| Nom du profil | ce que vous voulez — « XPackager » par défaut |
| Identifiant Apple | l'adresse de votre compte développeur |
| Team ID | les dix caractères entre parenthèses de vos certificats |
| Mot de passe spécifique | celui que vous venez de créer |

Bouton **Créer / mettre à jour le profil**. XPackager appelle
`xcrun notarytool store-credentials` : le mot de passe part dans le trousseau et n'est
**pas** conservé par l'application.

Le bouton **Vérifier** interroge Apple pour confirmer que le profil fonctionne.

Le nom du profil est celui que vous indiquerez dans chaque projet. Revérifiez la feuille
d'environnement : la cinquième ligne passe au vert.

---

## 5. Par l'exemple : livrer une application

Tout est en place. Faisons un installateur réel, celui d'une application qui doit atterrir
dans `/Applications`.

### 5.1 Partir d'un modèle

**Fichier ▸ Nouveau à partir d'un modèle…** puis **Application dans /Applications**.

Cinq modèles sont fournis : *Paquet vide*, *Application dans /Applications*, *Distribution à
2 composants*, *Ligne de commande*, *Plug-in*. Le modèle ne fait que préremplir — tout reste
modifiable.

La fenêtre s'ouvre sur quatre pages, dans la barre latérale : **Réglages**, **Composants**,
**Prérequis**, **Présentation**.

### 5.2 Page Réglages

| Champ | Exemple |
|---|---|
| Nom du paquet | `MonApplication` |
| Identité de signature | `Developer ID Installer: Votre Nom (XXXXXXXXXX)` |

Le menu d'identité ne propose que les certificats trouvés dans votre trousseau. XPackager
affiche sous le champ un rappel de ce que chaque type implique : `Developer ID Installer`
pour un paquet installable par double-clic, `3rd Party Mac Developer Installer` pour une
soumission au Mac App Store.

Puis, plus bas :

- **Re-signer le payload en Hardened Runtime avant de packager** : cochez.
- **Identité (Developer ID Application)** : votre certificat d'application.
- **Notariser après la construction** : cochez. Le profil utilisé est celui des Réglages.

> **Pourquoi la case « Re-signer » n'est pas optionnelle en pratique.** Beaucoup
> d'applications arrivent signées de façon ad hoc, sans Hardened Runtime, et parfois avec
> l'entitlement de débogage `com.apple.security.get-task-allow`. C'est le cas par défaut des
> applications produites par Xojo. Apple refuse de notariser tout cela. XPackager re-signe
> donc une **copie** du payload avec `codesign --options runtime --timestamp` — vos fichiers
> d'origine ne sont pas touchés — et n'y laisse aucun entitlement de débogage.

### 5.3 Page Composants

Un composant, c'est un choix proposé à l'installation, et un emplacement.

| Champ | Exemple |
|---|---|
| Nom (titre du choix) | `Application` |
| Identifiant | `fr.exemple.monapplication` |
| Version | `1.0` |
| Emplacement d'installation | `/Applications` |

L'identifiant suit la convention inverse du nom de domaine et doit rester **stable d'une
version à l'autre** : c'est lui qui permet à macOS de reconnaître une mise à jour.

En dessous, la zone de contenu : **glissez votre `.app` depuis le Finder**. Une application
est copiée telle quelle ; un dossier est importé avec toute son arborescence. Sélectionnez un
élément pour ajuster ses **Permissions** — `755` pour un exécutable, `644` pour un fichier
ordinaire.

Les **Options d'installation** (*Modifiable par l'utilisateur*, *Sélectionné par défaut*,
*Visible dans la liste personnalisée*) ne prennent leur sens qu'avec plusieurs composants :
c'est ce qui donne les cases à cocher de l'installation personnalisée. Pour un composant
unique, laissez-les tranquilles.

### 5.4 Page Prérequis

- **Version minimale de macOS** : `15.0` par exemple. Vide, aucune version n'est imposée.
- **Architectures autorisées** : Apple Silicon, Intel, ou les deux.

Vous pouvez aussi ajouter des conditions — présence d'un fichier, mémoire disponible — qui
bloquent l'installation avec un message.

### 5.5 Page Présentation

Ce sont les quatre écrans que verra la personne qui installe : **Bienvenue**, **Lisez-moi**,
**Licence**, **Conclusion**. Un sélecteur bascule de l'un à l'autre ; l'éditeur en dessous
est un éditeur riche — gras, italique, listes — et accepte aussi l'import d'un `.rtf` ou d'un
`.txt` existant.

Si votre installateur doit parler plusieurs langues, le menu à globe, à droite du sélecteur,
ajoute une langue et bascule l'éditeur dessus. La **langue de référence** est obligatoire :
c'est elle qui sert de secours quand une traduction manque. Le README détaille ce mécanisme.

### 5.6 Construire

**Fichier ▸ Construire…** (⌘B). Une fenêtre de journal montre chaque commande lancée :
re-signature du payload, `pkgbuild` par composant, `productbuild` pour l'assemblage et la
signature, `notarytool submit --wait`, puis `stapler staple`.

Comptez une à deux minutes : c'est Apple qui prend ce temps, pas XPackager. La ligne
`status: Accepted` puis `The staple and validate action worked!` concluent une construction
réussie.

### 5.7 Vérifier avant de diffuser

Trois commandes, sur le `.pkg` produit :

```bash
spctl -a -vvv -t install MonApplication.pkg
```

Doit répondre `accepted` et `source=Notarized Developer ID`. C'est le verdict de Gatekeeper,
celui qui compte.

```bash
xcrun stapler validate MonApplication.pkg
```

Confirme que le ticket est agrafé — donc que l'installation marchera même hors ligne.

```bash
pkgutil --check-signature MonApplication.pkg
```

Affiche la chaîne de certificats et la ligne `Notarization: trusted by the Apple notary
service`.

Pour regarder à l'intérieur sans installer :

```bash
pkgutil --expand-full MonApplication.pkg /tmp/verif
```

Enfin, le seul essai qui vaut vraiment : copiez le paquet sur une autre machine, par un
chemin qui pose la quarantaine (courrier, téléchargement), et installez-le pour de bon.

---

## 6. La même chose sans l'interface

L'outil en ligne de commande construit le même paquet à partir du même fichier de projet :

```bash
xpackagerbuild MonProjet.xpackager
```

Options utiles : `-o` pour la destination, `--notarize <profil>` pour forcer un profil,
`--no-notarize` pour sauter la notarisation pendant les essais — ce qui ramène la
construction à quelques secondes.

C'est ce qu'il faut employer depuis un script de livraison ou un pas de construction.

---

## 7. Quand ça échoue

| Message | Cause | Remède |
|---|---|---|
| `The signature of the binary is invalid` | signature ad hoc, non Developer ID | cochez **Re-signer le payload en Hardened Runtime** |
| `The executable does not have the hardened runtime enabled` | même cause | idem |
| `The executable requests the com.apple.security.get-task-allow entitlement` | entitlement de débogage, laissé par Xojo notamment | idem — XPackager ne le conserve jamais |
| `The signature does not include a secure timestamp` | signé sans `--timestamp` | idem |
| Profil introuvable | le nom du profil ne correspond à rien dans le trousseau | Réglages ▸ Notarisation ▸ **Vérifier** |
| Aucune identité dans le menu | certificat absent, ou clé privée sur une autre machine | section 3 |
| `Team is not yet configured for notarization` | contrat développeur non signé | developer.apple.com ▸ Agreements |

Pour lire le détail d'un refus d'Apple :

```bash
xcrun notarytool log <identifiant-de-soumission> --keychain-profile <profil>
```

L'identifiant de soumission figure dans le journal de construction, juste après
`Submission ID received`.

---

## En résumé

1. Installer Xcode
2. Créer les certificats **Developer ID Application** *et* **Developer ID Installer**
3. Créer un mot de passe spécifique et enregistrer le profil de notarisation
4. Vérifier que les cinq lignes de **Aide ▸ Vérifier l'environnement…** sont vertes
5. Construire

Les étapes 1 à 3 ne se font qu'une fois. Ensuite, un paquet signé et notarisé demande un
⌘B et deux minutes d'attente.
