# Veryfi Lens Android — Receipts — callbacks

Source of truth: Veryfi Lens Android SDK **2.7.0.6**, commit `7f33726c`. Names below were read from that tree. Items not fully confirmed are marked **Unconfirmed**. Do not copy repository passwords, keystores, or Demo-only types into the host app.


Delegate methods, all required except `onHelpButtonClicked` (default empty): `veryfiLensClose`, `veryfiLensUpdate`, `veryfiLensError`, `veryfiLensSuccess`. Argument type is `org.json.JSONObject`. Delivery is the main thread.

Common keys (`PackageUploadEvent` / server notify, snake_case): `package_id`, `upload_id`, `receipt_id`, `document_type`, `data`, `source`, `status`, `connectivity`, `progress`, `exception`, `error`, `msg`, `img_original_data`, `img_original_path`, `img_original_paths`, `img_thumbnail_path`, `img_thumbnail_paths`, `img_thumbnail_data`, `img_stitched_pdf_path`, `img_check_front_tiff_path`, `img_check_back_tiff_path`, `file_hashes`, `img_paths`, `images`, `framework-version`, `framework-build`, `session_scan_count`, `scan_session_counter`, `queue_count`, `scan_to_pdf_pages`, `device_id`, `type`, `empty_fields`, `local_package_path`.

| Status | Callback | Meaning |
|--------|----------|---------|
| `start` | update | Package created |
| `inprogress` | update | Uploading. Progress also arrives as `msg = "progress"` |
| `failed` | update or error | Android spelling. iOS uses `fail` |
| `error` | error | Error (`error` / `msg`) |
| `clear_package` | update | Local package cleaned |
| `close` | close | Lens UI closed |
| `done` | success | Finished |

File validation strings include `invalid_file`, `empty_file`, `image_too_small`, `invalid_pdf_file`, `image_is_screenshot`, `blurred_image`, `storage_permission_missing`.

| Engine | Success body |
|--------|----------------|
| `VeryfiCloudAPI` | Veryfi API result in `data` |
| `VeryfiInApp` for a receipt | Plain success, no extraction |
| `None` and `uploadWithoutProcessingIsOn = false` | `data.local_package_path` when local files are retained |
| `None` and `uploadWithoutProcessingIsOn = true` | `bucket` and `package_path` |

In-app and `None` still post a doc-ready metadata notification (document information, no image) to Veryfi.

`veryfiLensSuccess` `data` for the default recipe is the Veryfi API document. Copy files you still need inside the callback while `autoDeleteLocalResourcesAfterProcessing` is true (the default). `returnStitchedPDF = true` can add `img_stitched_pdf_path`. Each package has its own event sequence.

