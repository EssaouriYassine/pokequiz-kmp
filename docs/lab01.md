# Lab 01 - Étape 5 : Exploration de l'arborescence du projet PokéQuiz

### 1. Nombre de modules Gradle et inclusion d'iosApp
Le projet contient **3 modules Gradle** :
- `:shared`
- `:androidApp`
- `:desktopApp`

Ces modules sont déclarés dans le fichier **`settings.gradle.kts`** situé à la racine du projet.
**`iosApp` ne fait pas partie des modules Gradle** car il s'agit d'un projet natif Xcode (`.xcodeproj` / `.xcworkspace`) qui utilise le système de build d'Apple (`xcodebuild`). Le code Kotlin du module `:shared` lui est transmis sous forme de framework binaire via un script Gradle appelé directement par Xcode lors du build.

---

### 2. Versions des outils et bibliothèques utilisées
Les versions des dépendances principales sont centralisées dans le catalogue **`gradle/libs.versions.toml`** :
- **Kotlin** : `2.4.20`
- **Compose Multiplatform** : `1.12.1`
- **Android Gradle Plugin (AGP)** : `9.1.1`
- **Gradle** : `9.5.1` déclarée dans le fichier **`gradle/wrapper/gradle-wrapper.properties`** (`distributionUrl`)

---

### 3. Plugins appliqués dans `shared/build.gradle.kts` et `androidApp/build.gradle.kts`
- **`shared/build.gradle.kts`** applique :
    - `org.jetbrains.kotlin.multiplatform` : pour la configuration du code multiplateforme.
    - `com.android.library` : pour compiler la partie Android sous forme de bibliothèque (AAR).
    - `org.jetbrains.compose` : pour activer Compose Multiplatform.
- **`androidApp/build.gradle.kts`** applique :
    - `com.android.application` : pour générer l'application exécutable Android (APK/AAB).
    - `org.jetbrains.kotlin.plugin.compose` (ou plugin Android Compose).

*Pourquoi `androidApp` n'applique-t-il pas `org.jetbrains.kotlin.android` ?*  
Parce que l'application Android agit comme une enveloppe d'exécution native qui délègue toute sa logique multiplateforme au module `:shared`. Elle applique ainsi uniquement le plugin applicatif Android standard (`com.android.application`).

---

### 4. Configuration du `minSdk`
- **Bibliothèque partagée (`shared`)** : configurée dans `shared/build.gradle.kts` dans le bloc `android { defaultConfig { minSdk = ... } }`.
- **Application Android (`androidApp`)** : configurée dans `androidApp/build.gradle.kts` dans le bloc `android { defaultConfig { minSdk = ... } }`.
- **Forme** : Les valeurs sont définies via le version catalog `gradle/libs.versions.toml` en faisant référence à `libs.versions.android.minSdk.get().toInt()` (ou équivalent catalog).

---

### 5. Arborescence de `shared/src/` et gestion de `iosArm64Main`
- **Dossiers de `shared/src/`** :
    - `commonMain` (code partagé universel)
    - `commonTest`
    - `androidMain`
    - `jvmMain` (ou `desktopMain`)
    - `iosMain`
- *Pourquoi n'y a-t-il pas de dossier `iosArm64Main` ?*  
  Le plugin Kotlin Multiplatform utilise la hiérarchie de sources (Target Shortcuts) : les cibles `iosArm64()`, `iosX64()` et `iosSimulatorArm64()` sont toutes rattachées au sourceSet intermédiaire global **`iosMain`**.

---

### 6. Déclaration et implémentations de `getPlatform()`
- **Déclaration `expect`** : déclarée via `expect fun getPlatform(): Platform` dans le fichier **`shared/src/commonMain/kotlin/fr/m2/pokequiz/Platform.kt`**.
- **Implémentations `actual`** : il existe **3** implémentations `actual fun getPlatform(): Platform` :
    1. `shared/src/androidMain/kotlin/fr/m2/pokequiz/Platform.android.kt`
    2. `shared/src/jvmMain/kotlin/fr/m2/pokequiz/Platform.jvm.kt` (ou `desktopMain`)
    3. `shared/src/iosMain/kotlin/fr/m2/pokequiz/Platform.ios.kt`

---

### 7. Identification d'`applicationId` et de `mainClass`
- **`applicationId` Android** : déclaré dans **`androidApp/build.gradle.kts`** (valeur : `fr.m2.pokequiz` ou `fr.m2.pokequiz.android`).
- **`mainClass` Desktop** : déclarée dans **`desktopApp/build.gradle.kts`** (valeur : `fr.m2.pokequiz.MainKt` ou `fr.m2.pokequiz.main`).
- *Pourquoi se termine-t-elle par `Kt` ?*  
  En Kotlin, les fonctions globales / de niveau fichier (comme `fun main()` définies en dehors de toute classe) sont compilées par la JVM dans une classe générée automatiquement portant le nom du fichier suivi du suffixe `Kt` (ex. `main.kt` -> `MainKt`).

---

### 8. Définition et appels de la fonction `App()`
- **Définition** : définie dans **`shared/src/commonMain/kotlin/fr/m2/pokequiz/App.kt`**.
- **Appels** : elle est appelée depuis **3** endroits différents :
    1. `androidApp/src/androidMain/.../MainActivity.kt`
    2. `desktopApp/src/jvmMain/.../main.kt` (ou `mainApp.kt`)
    3. `iosApp/iosApp/ContentView.swift` (via le contrôleur de vue exposé dans `shared/src/iosMain/.../MainViewController.kt`)

---

### 9. Contenu du `.gitignore` et importance de `local.properties`
- **Contenu du `.gitignore`** : il exclut les fichiers de build et de configuration locale : `.gradle/`, `build/`, `local.properties`, `.idea/`, `*.iml`, `captures/`, ainsi que `xcuserdata/` pour iOS.
- *Pourquoi `local.properties` doit-il être ignoré ?*  
  Ce fichier contient des chemins absolus propres à la machine locale du développeur (notamment le chemin du SDK Android via `sdk.dir=...`), ainsi que d'éventuelles clés d'API locales. Le commiter casserait la compilation sur d'autres machines ou sur un serveur d'intégration continue (CI).

---

### 10. Ajout temporaire de `val f = java.io.File("x")` dans `App()`
- **Comportement de l'IDE** : L'IDE signale une erreur de compilation (`Unresolved reference: java` / `Cannot find package java.io`).
- **Explication** : La fonction `App()` se trouve dans le dossier `commonMain`. Le code écrit dans `commonMain` doit être strictement multiplateforme et ne peut pas faire référence à des bibliothèques spécifiques à la JVM Java (`java.io.*`), sous peine d'échouer lors de la compilation pour les cibles natives (comme iOS).