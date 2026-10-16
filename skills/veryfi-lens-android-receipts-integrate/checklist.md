# Validation checklist

Run this before delivering host code. Fix conflicts by removing extra flags.

- [ ] Dependency is exactly `com.veryfi.lens:veryfi-lens-receipts-sdk:2.7.0.6` plus the Nexus repository
- [ ] `documentTypes` contains only RECEIPT, OTHER, and/or INVOICE
- [ ] `dataExtractionEngine` is `VeryfiCloudAPI` (or `None` when the customer processes the package)
- [ ] Base hardening flags match settings.md, including `moreMenuIsOn = false` and `ignoreRemoteSettings = true`
- [ ] `setDelegate` then `configure { ok -> }` then `showCamera()` / `showGallery()` / `showDocumentBrowser()` only when ok is true
- [ ] Delegate implements close, update, error, and success; failure status compared to `failed`
- [ ] Gallery, browser, and `uploadImages` choices match the host entry point; PDFs use the Uri overload
- [ ] `uploadImages` is not described as the Headless Receipts SDK
- [ ] Fraud and `documentFieldsToDetect` are not promised from this artifact alone
- [ ] Any-Documents templates are escalated to Veryfi support instead of invented
- [ ] App size guidance compares the installed app, not the AAR

- [ ] Every relative link in the generated host notes stays inside this skill folder
- [ ] Unconfirmed behavior the host depends on is labeled **Unconfirmed** in the delivery note
- [ ] Host `<application>` sets `android:allowBackup="false"` and `tools:replace="android:allowBackup"` so a new-project `true` does not fail the manifest merger
- [ ] No repository password, Demo `SettingsHolder` preset, or `VeryfiLensHelper` mutation is in the host code
