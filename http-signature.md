## Headers

The client must send Digest, Date and Authorization headers

The digest header would be generated as:

    Digest: SHA-256=Base64(SHA256(Body))

The digest date would be generated as:

    Date: Thu Sep 24 2020 10:22:21 GMT+0300

The authorization header and signature would be generated as:

    Authorization: Signature keyId="<partner>",algorithm="rsa-sha256",headers="date digest",signature="Base64(RSA-SHA256(signing string))"

### Signing String Composition

In order to generate the string that is signed with a key, the client MUST take
the values of each HTTP header specified by `headers` in the order they appear.

1. If the header name is not `request-line` then append the lowercased header
   name followed with an ASCII colon `:` and an ASCII space ` `.
2. If the header name is `request-line` then append the HTTP request line,
   otherwise append the header value.
3. If value is not the last value then append an ASCII newline `\n`. The string
   MUST NOT include a trailing ASCII newline.

### Clock skew parameter is 120 seconds

## Headers example
    Digest: 'SHA-256=EhVaKekk5Lh3hikgTrNsAnS7qHw67cNyYTCdYUVzqAw='
    Date: 'Thu Sep 24 2020 10:22:21 GMT+0300'
    Authorization: 'Signature keyId="<partner>",algorithm="rsa-sha256",headers="date digest",signature="FUyrr0r54tRblP2P3U/HwH9HGaH8ZSEAevjV9lcYfvbCYeVOOAGLnmGm3mcvfb8zNWbtoNXuC9ZGS7SBY+PAyqyNu7J8t6ToGFqhRXyEG348sEOp2R5+JVzGnaPtes90vysIs2aMIkyZiZUF6PYV3QJLRQh+VJ6F8b9RnbvmELE="'

# Private key for encoding signature, Dummy data below.

    -----BEGIN RSA PRIVATE KEY-----
    XXXXXXX==
    -----END RSA PRIVATE KEY-----

# Example script(Node.js)
```javascript

import request from 'request'
import { createHash, createSign } from 'crypto'
import moment from 'moment'
import fs from 'fs'

const PRIVATE_KEY = fs.readFileSync('path_to_private_key');

(async () => {

  const options = getOptions();

  const response: any = await new Promise((res, rej) => {
    request(options, (err, response, body) => err ? rej(err) : res({ response, body }))
  })
})()

function buildDigest(body: any) {
  const rawBody = typeof body === 'string' ? body : JSON.stringify(body)
  return 'SHA-256=' + createHash('sha256').update(rawBody).digest('base64')
}

function getOptions() {
  const body = [110];
  const digest = buildDigest(body);
  const date = moment().toString(); //Fri Sep 04 2020 10:57:27 GMT+0300 
  let stringToSign = `date: ${date}\ndigest: ${digest}`;
  let signature = createSign('rsa-sha256')
    .update(stringToSign)
    .sign(PRIVATE_KEY, 'base64');

  const options = {
    method: 'POST',
    uri: 'https://<verticula-host>/api/insurance/received',
    json: body,
    headers: {
      Digest: digest,
      Date: date,
      Authorization: `Signature keyId="<partner>",algorithm="rsa-sha256",headers="date digest",signature="${signature}"`
    }
  }
  return options
}
```
