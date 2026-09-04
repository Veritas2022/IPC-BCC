# Collecte BCC — application Android

Projet prêt à compiler qui transforme le module enquêteur de l'applicatif IPC en une
application Android installable (**APK**) ou publiable sur le Play Store (**AAB**).

- Identifiant : `cd.bcc.collecteipc`
- Nom affiché : **Collecte BCC**
- Version : 1.0.0

---

## Voie 1 — obtenir l'APK sans rien installer (recommandée)

GitHub compile l'application pour vous, gratuitement, en environ six minutes. Aucun logiciel
à installer sur votre poste.

1. Créez un dépôt sur **github.com** — privé de préférence (bouton *New repository*).
2. Sur la page du dépôt vide, cliquez **« uploading an existing file »**, puis glissez-déposez
   **tout le contenu de ce dossier** (y compris les dossiers `www`, `resources`, `scripts`
   et `.github`). Validez avec *Commit changes*.
   *Si le dossier `.github` n'apparaît pas dans votre explorateur, activez l'affichage des
   fichiers cachés — c'est lui qui contient la recette de compilation.*
3. Ouvrez l'onglet **Actions** du dépôt, choisissez **« Construire l'APK »**, puis
   **Run workflow**.
4. Au bout de quelques minutes, la ligne du build affiche une pièce jointe
   **`Collecte-BCC-apk`**. Téléchargez-la : elle contient le fichier `app-debug.apk`.

**Installation sur les téléphones des enquêteurs** : transférez l'APK (câble, Bluetooth,
WhatsApp, carte SD), ouvrez-le sur le téléphone et autorisez l'installation depuis une source
inconnue lorsque Android le demande. Autorisez ensuite la localisation au premier relevé.

## Voie 2 — compiler sur un poste équipé

Nécessite **Node.js 20+**, **Android Studio** (SDK 34) et **JDK 17 ou 21**.

```bash
npm install
npx cap add android
node scripts/patch-manifest.mjs          # permissions de géolocalisation
npx capacitor-assets generate --android  # icône et écran de démarrage
npx cap sync android

cd android && ./gradlew assembleDebug    # → app/build/outputs/apk/debug/
```

`npm run android` ouvre le projet dans Android Studio si vous préférez l'interface graphique.

## Voie 3 — sans APK du tout

L'applicatif reste une **application web installable**. Ouvert dans Chrome sur le téléphone,
il propose « Ajouter à l'écran d'accueil » et s'exécute ensuite en plein écran, avec la même
icône et le même comportement. Ni compilation, ni signature, ni magasin d'applications — et une
correction se propage aux douze téléphones sans réinstallation.

L'APK ne devient nécessaire que pour une distribution par le Play Store, une gestion de parc
(MDM), ou des fonctions natives que le navigateur ne donne pas.

---

## Version signée pour le Play Store

Le Play Store exige un bundle **signé**. Créez une clé, une seule fois :

```bash
keytool -genkey -v -keystore bcc-collecte.keystore -alias bcc \
        -keyalg RSA -keysize 2048 -validity 10000
```

**Conservez ce fichier et ses mots de passe hors du dépôt.** Sans lui, aucune mise à jour de
l'application publiée ne sera possible — la clé n'est pas récupérable.

Puis, dans *Settings › Secrets and variables › Actions* du dépôt GitHub, créez quatre secrets :

| Secret | Contenu |
|---|---|
| `KEYSTORE_BASE64` | Le keystore encodé : `base64 -w0 bcc-collecte.keystore` |
| `KEYSTORE_PASSWORD` | Mot de passe du keystore |
| `KEY_ALIAS` | `bcc` |
| `KEY_PASSWORD` | Mot de passe de la clé |

Relancez ensuite le workflow : le second travail, **« Bundle signé pour le Play Store »**,
produit une pièce jointe `Collecte-BCC-aab`. Tant que les secrets sont absents, ce travail est
simplement ignoré et seul l'APK de test est produit.

Il reste à déclarer la signature dans `android/app/build.gradle` (`signingConfigs.release`)
si vous compilez localement ; en CI, le fichier `key.properties` est écrit automatiquement.

---

## Contenu du dossier

| Chemin | Rôle |
|---|---|
| `www/index.html` | L'application : un seul fichier autonome, aucune ressource externe |
| `capacitor.config.json` | Identifiant, nom, écran de démarrage marine, barre d'état |
| `package.json` | Dépendances Capacitor et raccourcis de compilation |
| `resources/icon.png` | Icône source 1024 × 1024, logo BCC sur fond marine |
| `resources/icon-foreground.png` | Calque avant de l'icône adaptative Android |
| `resources/splash.png` | Écran de démarrage 2732 × 2732 |
| `scripts/patch-manifest.mjs` | Ajoute les permissions de géolocalisation au manifeste |
| `.github/workflows/apk.yml` | Recette de compilation exécutée par GitHub |

## Mettre à jour l'application

1. remplacez `www/index.html` par la nouvelle version ;
2. incrémentez `versionCode` et `versionName` dans `android/app/build.gradle`
   (ou dans `package.json` avant de recréer le projet) ;
3. relancez le workflow.

## Permissions demandées

- `ACCESS_FINE_LOCATION` et `ACCESS_COARSE_LOCATION` — question A9 du formulaire, position du
  point de vente. Ajoutées automatiquement par `scripts/patch-manifest.mjs`.
- `INTERNET` — inutile tant que l'application fonctionne en mémoire locale ; nécessaire le jour
  où elle sera reliée à un serveur de la Banque.

Aucune autre permission n'est demandée : ni contacts, ni SMS, ni stockage externe.
