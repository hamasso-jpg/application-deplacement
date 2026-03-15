
# Mobilité ONG (Google Maps)

Application Android minimale (Kotlin + Compose) avec **Google Maps** intégrée (MapView).

## Configuration de la clé Google Maps

1. Crée une **API Key** dans Google Cloud Console et active **Maps SDK for Android**.
2. Restreins la clé aux **applications Android** :
   - Package: `com.ngo.mobilite`
   - Empreinte **SHA-1 debug** : exécuter `./gradlew signingReport` dans Android Studio/terminal.
3. Ajoute la clé dans `local.properties` (ne pas la committer) :
   ```
   MAPS_API_KEY=VOTRE_CLE_ICI
   ```

## Build local (APK debug)

```bash
./gradlew clean :app:assembleDebug
# APK : app/build/outputs/apk/debug/app-debug.apk
```

## GitHub Actions (APK automatique)

Ajoute ce workflow `.github/workflows/android.yml` :

```yaml
name: Build APK (Debug)

on:
  workflow_dispatch:
  push:
    branches: [ main, master ]

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      MAPS_API_KEY: ${{ secrets.MAPS_API_KEY }}
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup JDK
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - name: Ensure Gradle wrapper exists
        run: |
          if [ ! -f ./gradlew ]; then
            GRADLE_VERSION=8.4
            curl -L https://services.gradle.org/distributions/gradle-${GRADLE_VERSION}-bin.zip -o gradle.zip
            unzip -q gradle.zip -d $HOME/gradle
            $HOME/gradle/gradle-${GRADLE_VERSION}/bin/gradle wrapper
          fi
          chmod +x ./gradlew

      - name: Build Debug APK
        run: ./gradlew clean :app:assembleDebug

      - name: Upload APK artifact
        uses: actions/upload-artifact@v4
        with:
          name: app-debug-apk
          path: app/build/outputs/apk/debug/app-debug.apk
```

Puis crée un **secret** `MAPS_API_KEY` dans *Settings → Secrets and variables → Actions*.

## Test rapide

- Installe l'APK sur un appareil Android 8+ (`adb install -r app-debug.apk`).
- Ouvre l'app : une carte centrée sur **Niamey** s'affiche avec un marqueur.
- Si la carte n'apparaît pas, vérifie la facturation Google Cloud, la restriction SHA-1 et le nom de package.
```


## Couche "Ma position"

- L'app demande `ACCESS_FINE_LOCATION` au lancement.
- Si accordée, le **point bleu** et le **bouton cible** sont activés (Google Maps `isMyLocationEnabled`).
- La caméra tente de se centrer sur la **dernière position connue**.
