# yii2-fileupload

[![Packagist Version](https://img.shields.io/packagist/v/studio255/yii2-fileupload?style=flat-square)](https://packagist.org/packages/studio255/yii2-fileupload)
[![PHP](https://img.shields.io/badge/PHP-%3E%3D%208.1-777BB4?style=flat-square&logo=php&logoColor=white)](https://php.net)
[![Yii2](https://img.shields.io/badge/Yii2-~2.0.43-ED1E24?style=flat-square)](https://www.yiiframework.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-22c55e?style=flat-square)](LICENSE)

[FilePond](https://pqina.nl/filepond/)-based file upload widget for Yii2 — chunked uploads, ActiveRecord binding, and image variants out of the box.

---

## Features

- **FilePond widget** — drop-in Yii2 widget with auto-initialization for plain `<input>` fields
- **Chunked uploads** — enabled by default, bypasses PHP `post_max_size` limits
- **Multiple files** — configurable file count and MIME type filters
- **ActiveRecord binding** — auto-assigns filenames to model attributes after upload
- **Image variants** — optional server-side resizing (GD-based, keeps PNG transparency)
- **CSRF support** — sends `X-CSRF-Token` header automatically

---

## Installation

```bash
composer require studio255/yii2-fileupload
```

---

## Quick Start

### 1. Widget in a view

```php
use studio255\fileupload\FileUpload;
use yii\helpers\Url;

echo FileUpload::widget([
    'id'            => 'my-upload',
    'url'           => Url::to(['/upload/handle']),
    'acceptedFiles' => 'image/*',   // e.g. 'image/*,application/pdf'
    'maxFiles'      => 5,
    'multiple'      => true,
]);
```

### 2. Controller action

```php
use studio255\fileupload\UploadHandler;

public function actionHandle()
{
    return UploadHandler::processUpload([
        'targetPath' => \Yii::getAlias('@webroot/uploads'),
        // optional model binding:
        // 'modelClass' => app\models\MyModel::class,
    ]);
}
```

---

## Widget Options

| Option | Type | Default | Description |
|---|---|---|---|
| `id` | string | `'filepond'` | HTML `id` of the input |
| `url` | string | — | Upload URL |
| `multiple` | bool | `true` | Allow multiple files |
| `maxFiles` | int\|null | `5` | Maximum number of files |
| `acceptedFiles` | string\|null | `null` | Accepted MIME types / extensions |
| `model` | array | `[]` | Extra URL parameters passed to the server |
| `options` | array | `[]` | Additional FilePond options (see below) |

**Common `options` keys:**

| Key | Default | Description |
|---|---|---|
| `chunkUploads` | `true` | Enable chunked uploads |
| `chunkSize` | `1048576` | Chunk size in bytes (1 MB) |
| `extraData` | `[]` | Additional form fields sent with each upload |

---

## UploadHandler Options

```php
UploadHandler::processUpload([
    // Paths
    'targetPath' => \Yii::getAlias('@webroot/uploads'),  // default
    'tmpPath'    => \Yii::getAlias('@runtime/fileupload'), // chunk temp dir

    // Model binding (optional)
    'modelClass' => app\models\MyModel::class,
    // model_id and model_attribute are read from GET params by the widget

    // Image variants (optional)
    // 'createVariants' => true,  // generates a_ and w_ prefix files
    // 'a_' => [null, 200],       // thumbnail: height 200 px, keep aspect ratio
    // 'w_' => [800, null],       // preview: width 800 px, keep aspect ratio
]);
```

### Where files are saved

- Default: `@webroot/uploads/<filename>`
- With `model_id` param: `@webroot/uploads/{model_id}/<filename>`  
  The subfolder is cleared before saving (pass `emptyIdFolder=0` to disable).

---

## Multiple Instances

Use `options['instances']` to mount several FilePond ponds from a single widget call:

```php
echo FileUpload::widget([
    'options' => [
        'instances' => [
            [
                'id'            => 'pond-cover',
                'url'           => Url::to(['/upload/handle']),
                'multiple'      => false,
                'acceptedFiles' => 'image/*',
            ],
            [
                'id'       => 'pond-gallery',
                'url'      => Url::to(['/upload/handle']),
                'multiple' => true,
                'maxFiles' => 10,
            ],
        ],
    ],
]);
```

---

## Plain `<input>` (no widget)

The JS helper auto-initializes every `<input class="filepond">` on DOM ready via `data-*` attributes:

```html
<input type="file"
       class="filepond"
       data-url="/upload/handle"
       data-multiple="true"
       data-accepted="image/*"
       data-maxfiles="5"
/>
```

| Attribute | Description |
|---|---|
| `data-url` | Upload endpoint |
| `data-multiple` | `"true"` / `"false"` |
| `data-accepted` | Accepted MIME types |
| `data-maxfiles` | Max number of files |
| `data-model-id` | Sent to server as subfolder ID |

---

## Chunked Upload Protocol

The handler implements the FilePond TUS-like protocol automatically:

| Request | Purpose |
|---|---|
| `POST` (no files, `Upload-Length` header) | Init — returns a temp file ID |
| `HEAD ?patch=<id>` | Resume — returns current `Upload-Offset` |
| `PATCH ?patch=<id>` | Append chunk — finalizes on last chunk |

The CSRF token is sent as `X-CSRF-Token` from `<meta name="csrf-token">`. No extra configuration needed for standard Yii2 setups.

---

## Notes

- Filenames are sanitized server-side: only `A–Z a–z 0–9 - _` plus a lowercase extension are kept. German umlauts are transliterated (`ä→ae`, etc.).
- Image variants require the GD extension. Input formats: JPG, PNG, GIF. PNG output preserves transparency; all others are saved as JPEG at quality 90.
- Assets (FilePond CSS/JS) are registered automatically by the widget.

---

## License

MIT