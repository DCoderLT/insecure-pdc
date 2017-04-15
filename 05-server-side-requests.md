# Server side request forgery

## Method 1: Instagram integration

The extension supports fetching content from Instagram.
Instagram's API paginates long lists of data by sending a `next_url` parameter informing the client where to find the next page of results.
The extension contains an endpoint to fetch content for this usecase, but it does not validate the url before fetching it in any way.

In effect, passing any URL to this endpoint causes an HTTP GET request to be sent to that URL, and the response to be displayed directly in the browser.

### Request:
```
GET https://testing.local.tld/js/pdp/instagram/next.php?url=http://evilsite.com HTTP/1.1

```
_note that the .php file is stored under /js/, not in /app/_

### Response body:

The result of the request.

The browser continues to display the original Magento store URL in the address bar,
but the displayed content depends completely on the remote site.  
