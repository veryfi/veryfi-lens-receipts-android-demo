---
name: veryfi-lens-android-receipts-integrate
description: "Integrates the Veryfi Lens Android Receipts SDK (com.veryfi.lens:veryfi-lens-receipts-sdk:2.7.0.6) for Kotlin or Java: Maven repository, permissions, VeryfiLens.configure, VeryfiLensSettings for RECEIPT, INVOICE, and OTHER, VeryfiLensDelegate callbacks, showCamera, showGallery, showDocumentBrowser, and no-UI uploadImages. Use when adding Lens receipt or invoice capture to an Android app, choosing gallery or file import, wiring the delegate, or debugging receipt upload and extraction."
---

# Veryfi Lens Android — Receipts

Walk a host Android app from Gradle to a working first scan. Change the fewest flags. Do not invent SDK APIs. Facts were read from the Android SDK at version `2.7.0.6` (commit `7f33726c`). Anything not confirmed there is marked **Unconfirmed**.

**Coordinate:** `com.veryfi.lens:veryfi-lens-receipts-sdk:2.7.0.6`  
**Gradle flavor:** `lensReceipts`  
**Allowed document types:** `DocumentType.RECEIPT`, `DocumentType.OTHER`, and `DocumentType.INVOICE`.

Public entry point: `com.veryfi.lens.VeryfiLens` with `com.veryfi.lens.VeryfiLensDelegate`. Settings type: `com.veryfi.lens.helpers.VeryfiLensSettings`.

This folder is complete by itself. Duplicate guidance across products is intentional.

## Pick a workflow

- **Quick** — the host already asked for this product. Ask nothing else. Apply the settings in [settings.md](settings.md), wire [implementation.md](implementation.md), and state the assumptions below.
- **Guided** — something material is missing. Ask only the unknown questions (at most four, one message), then continue.

### Quick workflow

```
veryfi-lens-android-receipts-integrate quick:
- [ ] Confirm this artifact (install.md)
- [ ] Add the Maven repository and this one dependency
- [ ] Permissions (install.md)
- [ ] Credentials + settings from settings.md
- [ ] setDelegate, then configure { ok -> }, then showCamera() only when ok is true
- [ ] Handle the callbacks in callbacks.md
- [ ] Run the conflicts in troubleshooting.md
```

Assumptions when the host was specific about the product and silent on the rest: Lens camera UI, `DocumentType.RECEIPT` only, `VeryfiCloudAPI`, review-then-submit, gallery on, More menu off, no gallery backup.

### Guided workflow

1. **Types** — receipts only, or also invoices and other?
2. **Entry** — Lens camera, gallery, system files, or `uploadImages` for files the app already has?
3. **Extraction** — Veryfi Cloud (default) or decoupled (`ExtractionEngine.None`, customer runs Process a Document)?
4. **Size** — is the installed app size a constraint?

Defaults when the user does not answer: Lens camera, Cloud API, review-then-submit, gallery on, one document type `RECEIPT`.


## Rules

1. `documentTypes` must be a subset of RECEIPT, OTHER, INVOICE. `showCamera()` enforces that.
2. Apply the base hardening in [settings.md](settings.md) unless the host asked for the opposite. `moreMenuIsOn` defaults to true and, once a user has opened Lens, stored preferences replace host values of `autoDocDetectionAndCropIsOn`, `blurDetectionIsOn`, `glareDetectionIsOn`, `autoLightDetectionIsOn`, `autoCaptureIsOn`, `backupDocsToGallery`, `closeCameraOnSubmit`, and `batchUploadOnCamera`.
3. Call `VeryfiLens.setDelegate` before `VeryfiLens.configure(application, credentials, settings) { ok -> }`. Call `showCamera()`, `showGallery()`, or `showDocumentBrowser()` only when `ok` is true.
4. Extraction for receipts is `VeryfiCloudAPI`. `VeryfiInApp` returns a plain success and no receipt fields.
5. `VeryfiLens.uploadImages` is the no-UI file path on this same UI SDK. Headless receipt capture is a different Maven coordinate (`com.veryfi.lens.headless:receipts-sdk`) and a different type (`VeryfiLensHeadless`).


## Do not

- Copy a repository password, keystore, or credential from the SDK repo or the Demo app. Maven access comes from Veryfi. Keep it in `~/.gradle/gradle.properties` or CI secrets.
- Depend on a second Lens UI flavor, or on `com.veryfi.lens.core:veryfi-lens-core-sdk` (email and obsolete dictation; adds `WRITE_CONTACTS` and `RECORD_AUDIO`).
- Treat Demo types as the client contract: `SettingsHolder`, Demo analyzers, `ImageProxyUtils`, `VeryfiLensHelper`.
- Set `dictateIsOn = true` or call `VeryfiLensCore.showDictation()`.
- Set `anyDocumentType` (deprecated local regex OCR). A new template id is `anyDocumentTemplate`, and only after Veryfi support confirms it.
- Set `useLegacyModels` (default true; no partner contract).
- Quote AAR size as the installed app size. Compare the final app with the SDK linked and removed. [SDK binary sizes](https://docs.veryfi.com/lens/mobile/settings/sdk_binary_sizes/).
- Treat `allowSubmittingScreenshots` (rejects imported screenshots) and `enableScreenshots` (device screenshots of Lens screens, default true) as one switch.
- Put `LONG_RECEIPT`, `CREDIT_CARD`, `CODE`, `CHECK`, `W2`, `W9`, or `ANY_DOCUMENT` in `documentTypes` on this artifact.
- Promise on-device `documentFieldsToDetect` on `veryfi-lens-receipts-sdk`. That build uses `fieldDetectionStub`.
- Call `uploadImages(paths)` with PDFs. Use the `Uri` overload.
- Treat `browseIsOn` as a global import switch. It only fills the Import card inside the More menu.


## What to hand back

1. The repository block and `implementation "com.veryfi.lens:veryfi-lens-receipts-sdk:2.7.0.6"`.
2. Permission notes that match [install.md](install.md). Usually none beyond the merged manifest.
3. A `VeryfiLensDelegate`, `configure { ok -> }`, and one launcher inside the success branch.
4. `receiptsSettings()` with a one-line comment on every flag that is not an Android default.
5. Residual conflicts, fraud/field-detection limits, and any **Unconfirmed** item the host is relying on.


## Files in this folder

- [install.md](install.md) — Maven coordinate, requirements, permissions, app size
- [implementation.md](implementation.md) — configure, delegate, and launch or camera lifecycle
- [settings.md](settings.md) — `VeryfiLensSettings` for this product, with Android defaults
- [callbacks.md](callbacks.md) — delegate payloads
- [troubleshooting.md](troubleshooting.md) — symptom, cause, fix
- [clarifications.md](clarifications.md) — Unconfirmed items, Any-Documents, fraud, field detection
- [checklist.md](checklist.md) — validation checklist before handing code to the host

