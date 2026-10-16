# Veryfi Lens Android — Receipts — settings

Source of truth: Veryfi Lens Android SDK **2.7.0.6**, commit `7f33726c`. Names below were read from that tree. Items not fully confirmed are marked **Unconfirmed**. Do not copy repository passwords, keystores, or Demo-only types into the host app.


Always construct a fresh `VeryfiLensSettings()`. Android has no `VeryfiLens*Settings` subclasses. Pass that instance to `configure`. The SDK mutates it.

```kotlin
import com.veryfi.lens.helpers.DocumentType
import com.veryfi.lens.helpers.VeryfiLensSettings

fun receiptsSettings() = VeryfiLensSettings().apply {
    ignoreRemoteSettings = true            // default false; dashboard lens_settings overwrite this object during configure
    backupDocsToGallery = false            // default true; copies every scan into the device gallery
    moreMenuIsOn = false                   // default true; stored user prefs then replace host capture flags
    moreSettingsMenuIsOn = false           // default true
    autoCaptureOtherIsOn = false           // default true; OTHER auto-captures even when autoCaptureIsOn is false
    showNoDocumentDetectedWarning = false  // default true
    dictateIsOn = false                    // default true; obsolete voice capture, not a partner feature
    locationServicesIsOn = false           // default false; true raises a runtime location request
    glareDetectionIsOn = false             // default true; the glare gate is a weak algorithmic check

    dataExtractionEngine = VeryfiLensSettings.ExtractionEngine.VeryfiCloudAPI // default
    documentTypes = arrayListOf(DocumentType.RECEIPT) // null is rejected by showCamera()
    showDocumentTypes = false               // set true when INVOICE or OTHER is also listed
    autoSubmitDocumentOnCapture = false     // default; review screen stays on
    imagePreviewIsOn = true                 // default
    galleryIsOn = true                      // default
    autoRotateIsOn = true                   // default false
    autoSkewCorrectionIsOn = true           // default false
    cleanBorderIsOn = true                  // default false
    stitchIsOn = true                       // default true
    multipleDocumentsIsOn = true            // default true; requires stitchIsOn
    multiplePagesCaptureIsOn = true         // default true
    returnStitchedPDF = true                // default false; success may include img_stitched_pdf_path
    packageMaxScans = 10                    // default 5
    sameFileUploadPreventionPolicy = VeryfiLensSettings.SameFileUploadPreventionPolicy.Session // default None
    webhookIsOn = false                     // default
}

```

## Defaults versus this recipe

Android initializers below are the property defaults in `VeryfiLensSettings`.

| Property | Android default | This recipe | Why |
|----------|-----------------|-------------|-----|
| `ignoreRemoteSettings` | false | true | Dashboard `lens_settings` would overwrite this object during configure |
| `backupDocsToGallery` | true | false | Default copies every scan into the device gallery |
| `moreMenuIsOn` | true | false | Stored user prefs would replace host capture flags on later configure calls |
| `moreSettingsMenuIsOn` | true | false | Settings sheet inside More |
| `autoCaptureOtherIsOn` | true | false | `OTHER` auto-captures even when `autoCaptureIsOn` is false |
| `showNoDocumentDetectedWarning` | true | false | Skip the no-document alert |
| `dictateIsOn` | true | false | Obsolete voice control; default is on |
| `glareDetectionIsOn` | true | false | Android default glare gate is a weak check |
| `locationServicesIsOn` | false | false | Leave off unless the host needs lat/lng on the package |

| `documentTypes` | `null` | `RECEIPT` | `showCamera()` refuses a null list |
| `autoRotateIsOn` | false | true | Rotate the captured page |
| `autoSkewCorrectionIsOn` | false | true | Deskew after capture |
| `cleanBorderIsOn` | false | true | Clean-border post process |
| `returnStitchedPDF` | false | true | Ask for `img_stitched_pdf_path` |
| `packageMaxScans` | 5 | 10 | Cap pages in one package |
| `sameFileUploadPreventionPolicy` | `None` | `Session` | Block re-selecting the same file in this Lens session |


Add `DocumentType.INVOICE` and `DocumentType.OTHER` only when the host needs them, and set `showDocumentTypes = true` plus `defaultSelectedDocumentType = DocumentType.RECEIPT`. `LONG_RECEIPT` is rejected by this flavor.

`barcodeExtractionIsOn` (default false) asks the **Cloud API** to read barcodes printed on a receipt. That is separate from the Barcodes artifact, which reads barcodes on device with `DocumentType.BARCODE` and `VeryfiInApp`.

Decoupled capture, when the customer runs Process a Document: set `dataExtractionEngine = VeryfiLensSettings.ExtractionEngine.None`. Keep `uploadWithoutProcessingIsOn = false` and `autoDeleteLocalResourcesAfterProcessing = false` for a local zip (`data.local_package_path`). Set `uploadWithoutProcessingIsOn = true` only together with `None` for a remote package (`bucket`, `package_path`). Cloud response flags (`boundingBoxesIsOn`, `confidenceDetailsIsOn`, `parseAddressIsOn`, `detectBlurResponseIsOn`, `boostModeIsOn`, `computeIsOn`, `webhookIsOn`) do not shape the customer's later API call.

`VeryfiInApp` on a receipt returns a plain success and no extracted fields. Keep `VeryfiCloudAPI` for extraction.


## Suggestions

- Leave `moreMenuIsOn` false unless end users should permanently override capture toggles.
- Turn on `showHelpButton` and `showGuideShortCounter = 2` when users need the short-receipt guide. `showDefaultScanningGuide = true` replays the built-in guide; `false` calls `onHelpButtonClicked(parentFragment)`.
- Set `autoDeleteLocalResourcesAfterProcessing = false` when the host shows images after the callback.
- For installed-size pressure, measure the app, then consider `com.veryfi.lens:veryfi-lens-remote-models-sdk:2.7.0.6` (same Full allow-list, including these receipt types). Stay on the receipts artifact when the product must stay receipts-only.

