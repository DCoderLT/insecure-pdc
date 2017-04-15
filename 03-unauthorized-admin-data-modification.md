# Delete library resources uploaded by administrators

This extensions lets backend users upload and create certain resources that can be used by frontend users: clipart images, fonts, shapes.
With no good reason, the frontend controllers contain functionality allowing frontend users to delete those resources. 

## Delete images

### Request:
```
POST https://testing.local.tld/pdp/index/deleteImageById/ HTTP/1.1
Content-Type: application/x-www-form-urlencoded

img_list=_1,_2
```

### Response
Empty.

This deletes all mentioned image records from the database. 
The image files are not deleted.


## Delete fonts

### Request:
```
POST https://testing.local.tld/pdp/index/deleteFontById/ HTTP/1.1
Content-Type: application/x-www-form-urlencoded

font_list=_1,_2
```

### Response
Empty.

This deletes all mentioned font records from the database. 
The font files are also deleted. 
Uploading fonts is possible for both administrators and frontend users, 
but font file names are stripped of dots, so malformed file names are difficult to create.

## Delete admin templates

### Request:
```
POST https://testing.local.tld/pdp/index/removeSampleData/ HTTP/1.1
Content-Length: 145

---------------------------acebdf13572468
Content-Disposition: form-data; name="product-id"

1235
---------------------------acebdf13572468--
```

### Response
Varies depending on whether there is (was) an admin template associated with the given product id.

The first admin template associated with that product ID is deleted from the DB. 

## Delete shapes

### Request:
```
POST https://testing.local.tld/pdp/shape/deleteImageById/ HTTP/1.1
Content-Type: application/x-www-form-urlencoded

img_list=_1,_2
```

### Response
Empty.

This deletes all mentioned shape records from the database. 
