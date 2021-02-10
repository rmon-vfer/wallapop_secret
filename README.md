# wallapop_secret
Wallapop's API X-Signature Generator

To authenticate requests to Wallapop's REST API you need a client-side generated token, namely X-Signature, generated mangling and hashing the current timestamt with a secret key, HTTP method used and the target endpoint.

The basic token structure is (pseudocode):
`Base64DecodeFromHex( HMACSHA256( Base64DecodeFromUTF8('UTI5dVozSmhkSE1zSUhsdmRTZDJaU0JtYjNWdVpDQnBkQ0VnUVhKbElIbHZkU0J5WldGa2VTQjBieUJxYjJsdUlIVnpQeUJxYjJKelFIZGhiR3hoY0c5d0xtTnZiUT09'), "/api/v3/{ENDPOINT}+#+{METHOD}+#+{TIMESTAMP}+#+") )`

A simple python script to automate the signature generation:
```python
import hmac
import hashlib
import time
import base64
import codecs

def generate_xsignature(endpoint, method, timestamp):
    secret_key = b"UTI5dVozSmhkSE1zSUhsdmRTZDJaU0JtYjNWdVpDQnBkQ0VnUVhKbElIbHZkU0J5WldGa2VTQjBieUJxYjJsdUlIVnpQeUJxYjJKelFIZGhiR3hoY0c5d0xtTnZiUT09"
    total_params = b"/api/v3/"+ endpoint.encode() +b"+#+"+ method.encode() +b"+#+" + timestamp.encode() + b"+#+"
    signature= hmac.new(base64.b64decode(secret_key), total_params, hashlib.sha256).digest()
    return str(codecs.encode(signature, 'base64').decode()).strip()
```

Timestamps can get generated using `time` as follows:
```python
timestamp = str(time.time()).split(".")[0]
```

Applying Base64DecodeFromUTF8 twice on the secret key gives:
```
Congrats, you've found it! Are you ready to join us? jobs@wallapop.com
```

Everything was reverse-engineered using Firefox Devtools, Frida and Charles Proxy

**Update:** They migrated to React and reimplemented the signature generation algorithm:

New key: "Tm93IHRoYXQgeW91J3ZlIGZvdW5kIHRoaXMsIGFyZSB5b3UgcmVhZHkgdG8gam9pbiB1cz8gam9ic0B3YWxsYXBvcC5jb20=="
New algo:
```js
const SIGNATURE = 'Tm93IHRoYXQgeW91J3ZlIGZvdW5kIHRoaXMsIGFyZSB5b3UgcmVhZHkgdG8gam9pbiB1cz8gam9ic0B3YWxsYXBvcC5jb20==';

export function getSignature(url, method, timestamp) {
  try {
    if (url.includes('registration')) {
      url = url.split('?')[0];
    }
  }
  catch (e) {}
  
  var separator = '|';
  var signature = [method, url, timestamp].join(separator) + separator;

  return window.CryptoJS.enc.Base64.stringify(window.CryptoJS.HmacSHA256(signature, SIGNATURE));
}
```
The previous python code still works changing the corresponding key (although I'd rather use their implementation)

The hiring message has also been changed: "Now that you've found this, are you ready to join us? jobs@wallapop.com"
For more information see issue https://github.com/rmon-vfer/wallapop_secret/issues/1#issuecomment-694163493
