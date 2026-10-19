# PokéQuiz — Application Kotlin Multiplatform (KMP)

[![Kotlin](https://img.shields.io/badge/Kotlin-2.0+-7F52FF.svg?logo=kotlin)](https://kotlinlang.org/)
[![Compose Multiplatform](https://img.shields.io/badge/Compose_Multiplatform-1.6+-blue.svg)](https://www.jetbrains.com/lp/compose-multiplatform/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

**PokéQuiz** est une application multiplateforme (Android, Desktop, iOS) développée en **Kotlin Multiplatform (KMP)** et **Compose Multiplatform** dans le cadre des travaux pratiques de Master.

---

## Fonctionnalités & Contenu

- **UI Partagée** : Interface utilisateur réutilisable sur l'ensemble des cibles grâce à Compose Multiplatform.
- **Gestion de plateforme (Expect / Actual)** : Détection et affichage dynamique des informations système propres à chaque cible (OS, version Java/Android).
- **Compteur interactif** : Gestion de l'état UI en code commun (`remember`, `mutableIntStateOf`).
- **Support Compose Hot Reload** : Développement fluide sur la cible Desktop.

---

## 📂 Structure du projet

```text
PokéQuiz/
├── androidApp/      # Module applicatif spécifique Android
├── desktopApp/      # Module applicatif spécifique Desktop (JVM)
├── iosApp/          # Projet Xcode pour l'application iOS
├── shared/          # Module KMP contenant le code partagé
│   └── src/
│       ├── commonMain/   # Logique métier et UI Compose partagée
│       ├── androidMain/  # Implémentations spécifiques Android (actual)
│       ├── jvmMain/      # Implémentations spécifiques Desktop (actual)
│       └── iosMain/      # Implémentations spécifiques iOS (actual)
└── docs/
    └── lab01.md     # Réponses aux questions d'exploration et de réflexion
```
---

## Prérequis & Configuration
- **JDK :** Version 17 minimum (JDK 21 ou JetBrains Runtime recommandé).

- **Android SDK :** API Level 36.

- **Android Studio :** Avec le plugin Kotlin Multiplatform.

- **Xcode (Optionnel, macOS uniquement) :** Pour la compilation et l'exécution sur iOS.

---

## Exécution de l'application
Via l'IDE (Android Studio)
Sélectionnez la configuration de lancement souhaitée (androidApp ou desktopApp) puis cliquez sur Run.

Via le Terminal (Gradle CLI)
- Lancer la version Desktop :

    ```Bash
    ./gradlew :desktopApp:run
    ```
- Installer l'application Android sur un émulateur / appareil connecté :

    ```Bash
    ./gradlew :androidApp:installDebug
    ```

---

## Livrables & Versionning
Le projet est taggué selon la progression du TP :

Tag lab01 : Initialisation du projet, exploration de l'arborescence, écran d'accueil en français et intégration du compteur.
