# Veryfi Lens Android — Receipts — install

Source of truth: Veryfi Lens Android SDK **2.7.0.6**, commit `7f33726c`. Names below were read from that tree. Items not fully confirmed are marked **Unconfirmed**. Do not copy repository passwords, keystores, or Demo-only types into the host app.


## Requirements

| Item | Value |
|------|--------|
| minSdk | 23 |
| compileSdk / targetSdk used by the SDK | 36 |
| Language | Kotlin façade, callable from Java via `@JvmStatic` |
| SDK version | `2.7.0.6` (commit `7f33726c`) |

Camera stack: CameraX, bundled transitively by the Lens UI artifact. The host does not add a second CameraX stack for `showCamera()`.

## Maven

Repository: `https://nexus.veryfi.com/repository/maven-releases/`

```groovy
// settings.gradle — dependencyResolutionManagement.repositories
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven {
            url "https://nexus.veryfi.com/repository/maven-releases/"
            // Only when Veryfi issued repository credentials. Read them from Gradle properties.
            // Never commit them, and never copy a password out of the SDK repository.
            credentials {
                username = providers.gradleProperty("veryfiMavenUser").getOrNull()
                password = providers.gradleProperty("veryfiMavenPassword").getOrNull()
            }
        }
    }
}
```

```groovy
// app/build.gradle
dependencies {
    implementation "com.veryfi.lens:veryfi-lens-receipts-sdk:2.7.0.6"
}
```

Transitive `com.veryfi.lens.cpp` and `com.veryfi.lens.helpers` artifacts resolve with the Lens dependency. Do not add them by hand.


## Flavor check

`FlavorCheckerHelper` reads `BuildConfig.FLAVOR` (`lensReceipts`). Allowed types: `DocumentType.RECEIPT`, `DocumentType.OTHER`, and `DocumentType.INVOICE`.

`showCamera()` calls `checkLensReceipts`. Any other `DocumentType` shows Toast `Please use only valid document types, refer to hub docs: Lens for Receipts & Invoices` and does not open the camera.

`com.veryfi.lens:veryfi-lens-remote-models-sdk:2.7.0.6` and `com.veryfi.lens:veryfi-lens-sdk:2.7.0.6` also allow these three types (they use `checkLensFull`). They reject `CREDIT_CARD` and `CODE`. Use them only when the host truly needs that wider set or remote model download. This skill's dependency stays the receipts artifact unless install size forces that conversation — see app size below.


## Permissions

`veryfilens` manifest merges these into the host. Redeclaring them is unnecessary:

`CAMERA`, `INTERNET`, `ACCESS_NETWORK_STATE`, `ACCESS_FINE_LOCATION`, `ACCESS_COARSE_LOCATION`, `FLASHLIGHT`, `VIBRATE`, `FOREGROUND_SERVICE`, `FOREGROUND_SERVICE_DATA_SYNC`, `POST_NOTIFICATIONS`, `READ_EXTERNAL_STORAGE`, `WRITE_EXTERNAL_STORAGE`.

Also merged, and with no platform effect: `READ_INTERNAL_STORAGE`, `WRITE_INTERNAL_STORAGE`.

Camera `uses-feature` entries are `android:required="false"` (`camera`, `camera.any`, `camera.autofocus`, `camera.flash`, `sensor.light`).

Lens UI requests permissions from `CaptureActivity`:

| Permission | When |
|------------|------|
| `CAMERA` | When the camera opens |
| Location | Only when `locationServicesIsOn = true` |
| Storage (`READ`/`WRITE_EXTERNAL_STORAGE`, API < 33) | When `galleryIsOn`, `browseIsOn`, or `backupDocsToGallery` is true |
| `POST_NOTIFICATIONS` (API 33+) | Optional. Uploads run in `UploadDocumentsService` (`foregroundServiceType="dataSync"`) |

Host duties:

- `VeryfiLens.uploadImages(uris)` on API < 33 requires storage permission already granted. Otherwise `veryfiLensError` carries `storage_permission_missing`. The `paths` overload does not check permission.
- The AAR declares a `FileProvider` with authority `${applicationId}.veryfi.lens.fileprovider`. Do not declare another provider with that authority.
- New projects set `android:allowBackup="true"`. Lens manifests set `false`, so the merger fails: `Attribute application@allowBackup value=(true) ... is also present at [com.veryfi.lens:…] value=(false)`. On `<application>`, set `android:allowBackup="false"` and `tools:replace="android:allowBackup"`, with `xmlns:tools` on `<manifest>`. Append `android:allowBackup` when `tools:replace` already lists other attributes. Leave `dataExtractionRules` and `fullBackupContent` in place. Keep `allowBackup="true"` only when the host must enable Auto Backup.
- Stripping location or storage from the Play listing is a host manifest change (`tools:node="remove"`) plus `locationServicesIsOn = false`, `backupDocsToGallery = false`, and gallery/browse off on API < 33. That merger behavior is standard. Whether every Lens path tolerates the removal is **Unconfirmed**.


## App size

Compare the **installed app**, with this dependency linked and with it removed (release APK/AAB, Android Studio APK Analyzer, or the Play Console size report). The published AAR is a fat artifact. Do not quote AAR size as the user's install delta. Background: [SDK binary sizes](https://docs.veryfi.com/lens/mobile/settings/sdk_binary_sizes/).

`RemoteModelConfig.isRemoteModelEnabled()` is true only for flavors `lensDemo`, `lensRemoteModels`, and `lensLite`. The receipts flavor does not download models after install. When the allow-list can grow to the Full set and install size matters, `veryfi-lens-remote-models-sdk` is the size-oriented UI artifact. It still rejects `CREDIT_CARD` and `CODE`.



