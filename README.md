# St.JOSE - Secure, targeted JOSE
Opinionated usage of JWK/JWS/JWT stack for representing authentication and confidentiality messaging in cloud services

## `"npk"` - Notarized Public Keys (JWT as JWS header value with a notarized JWK)
> AKA - pinned key with temporary subkeys.

Goals / setup description:
- have a cloud service that depend on Ed25519 signing keys for identity (pubkey == identity, much like bitcoin addresses)
- have client applications pin that public key in code
- don't have exposed copies of the pinned key private component in cloud services
- don't create a full X509 style PKI CA system
- keep pinned key offline and distribute temporary keypairs to cloud services during deployment

Here because https://www.reddit.com/r/crypto/comments/hoph9d/x509_in_jsonjwt/ yielded no results.

In essence something not unlike PGP subkeys or mini-CA where there is no "subject identity", just public keys.

In essence:
- "npk" field for JWT payload, which is like a certificate; contains the public key to be notarized
- "npk" field for JWS header, which acts like replacement for "kid" - includes a self-contained claim that chains to an explicitly trusted public key.
- convention for expressing with JW* technologies the claim: "this key" certifies "that key" to be valid for a while as "this key"

Similar:
- https://tools.ietf.org/id/draft-yusef-oauth-nested-jwt-00.html
- https://gist.github.com/kag0/afc424b76d9ac46ba88cbf18dd57dc18

#### Given a master key with JWK thumbprint `gNcTLfHQVkVbnoUSHezrNYdACB8G215Yhuz8v-WQ73s`
```
{
  "crv" : "Ed25519",
  "kty" : "OKP",
  "x" : "dLz8B-W06uOuzFFzywK_aWQsMK5siPi6YRU10vYzvO8"
}
```
NOTES
- public key value "x" MUST be pinned in the application

#### Given a subkey with JWK thumbprint `WgL_9NpEmJtUBQ8q-Ff3txkU2vInbZxFF9rxeintiRU`
```
{
  "crv" : "Ed25519",
  "kty" : "OKP",
  "x" : "wy8hrsVOUxT6eGdQnccdVNbx7JnV7oqiSGMY2kmGgjo"
}
```
#### Shall be combined into the following JWT containing `"npk"`
```
{
  "typ" : "JWT",
  "kid" : "gNcTLfHQVkVbnoUSHezrNYdACB8G215Yhuz8v-WQ73s",
  "alg" : "EdDSA"
}
{
  "iss" : "dLz8B-W06uOuzFFzywK_aWQsMK5siPi6YRU10vYzvO8",
  "npk" : {
    "crv" : "Ed25519",
    "kty" : "OKP",
    "x" : "wy8hrsVOUxT6eGdQnccdVNbx7JnV7oqiSGMY2kmGgjo"
  },
  "iat" : 1602406010,
  "nbf" : 1602406010,
  "exp" : 1602492410
}
```
NOTES
- "npk" MUST be present and contain the JSON used to calculate the JWK Thumbrpint of the key
  - XXX: would use "sub" but that must be a string. Encoding "npk" into base64 for inclusion in "sub" seems weird
- "iat", "nbf" and "exp" MUST be present and MUST be checked against current time when consumed
- "iss" SHOULD be present and represent the "x" of EdDSA public key for self-contained NPK-s
- "kid" SHOULD be present and represent the JWT thumbprint of a pinned public key.


#### When used to sign the following:
```
{
  "kid" : "WgL_9NpEmJtUBQ8q-Ff3txkU2vInbZxFF9rxeintiRU",
  "npk" : "eyJ0eXAiOiJKV1QiLCJraWQiOiJnTmNUTGZIUVZrVmJub1VTSGV6ck5ZZEFDQjhHMjE1WWh1ejh2LVdRNzNzIiwiYWxnIjoiRWREU0EifQ.eyJpc3MiOiJkTHo4Qi1XMDZ1T3V6RkZ6eXdLX2FXUXNNSzVzaVBpNllSVTEwdll6dk84IiwibnBrIjp7ImNydiI6IkVkMjU1MTkiLCJrdHkiOiJPS1AiLCJ4Ijoid3k4aHJzVk9VeFQ2ZUdkUW5jY2RWTmJ4N0puVjdvcWlTR01ZMmttR2dqbyJ9LCJpYXQiOjE2MDI0MDYwMTAsIm5iZiI6MTYwMjQwNjAxMCwiZXhwIjoxNjAyNDkyNDEwfQ.14sx28_6SSHGwEphHB3ddXHHBVeqDTJGXovQSYXZv3svdntf-bhZMp_hfjknFEdYYSmJms78-G9rMDZv_CocAQ",
  "alg" : "EdDSA"
}
{
  "Hello" : "World",
  "novelty" : false
}
```
#### will yield an actual JWS of npk header + payload
`eyJraWQiOiJXZ0xfOU5wRW1KdFVCUThxLUZmM3R4a1UydkluYlp4RkY5cnhlaW50aVJVIiwibnBrIjoiZXlKMGVYQWlPaUpLVjFRaUxDSnJhV1FpT2lKblRtTlVUR1pJVVZaclZtSnViMVZUU0dWNmNrNVpaRUZEUWpoSE1qRTFXV2gxZWpoMkxWZFJOek56SWl3aVlXeG5Jam9pUldSRVUwRWlmUS5leUpwYzNNaU9pSmtUSG80UWkxWE1EWjFUM1Y2UmtaNmVYZExYMkZYVVhOTlN6VnphVkJwTmxsU1ZURXdkbGw2ZGs4NElpd2libkJySWpwN0ltTnlkaUk2SWtWa01qVTFNVGtpTENKcmRIa2lPaUpQUzFBaUxDSjRJam9pZDNrNGFISnpWazlWZUZRMlpVZGtVVzVqWTJSV1RtSjROMHB1VmpkdmNXbFRSMDFaTW10dFIyZHFieUo5TENKcFlYUWlPakUyTURJME1EWXdNVEFzSW01aVppSTZNVFl3TWpRd05qQXhNQ3dpWlhod0lqb3hOakF5TkRreU5ERXdmUS4xNHN4MjhfNlNTSEd3RXBoSEIzZGRYSEhCVmVxRFRKR1hvdlFTWVhadjNzdmRudGYtYmhaTXBfaGZqa25GRWRZWVNtSm1zNzgtRzlyTURadl9Db2NBUSIsImFsZyI6IkVkRFNBIn0.eyJIZWxsbyI6IldvcmxkIiwibm92ZWx0eSI6ZmFsc2V9.9LVJvJrJNGGlozLYhts3G-AetgMeBmmTVAoKlt9-J7OEOPM8jLvL_dHBHEEOyXxdUxhxl7DAWyra4sWTehioDw`


NOTES
- "kid" MAY be present in header
- when "kid" is present in header, it MUST match the "kid" of the "npk"
- application MAY cache NPK-s, in which case it MUST check the time related fields of the JWT before each use.


### NPK verification
Given a JWS, if it has a "kid" header, check to see if it matches a pre-defined, pinned public key. 
Alternatively, if "npk" is present, check if it contains a JWT signed with a pre-defined, pinned public key. And if the JWS is signed with the public key contained in the "npk" field of the JWT.


### Links
- https://www.iana.org/assignments/jose/jose.xhtml field registry
- https://tools.ietf.org/html/rfc7515 JWS
- https://tools.ietf.org/html/rfc7638 JWK thumbprint
- https://tools.ietf.org/html/rfc7519 JWT
- https://tools.ietf.org/html/rfc7516 JWE
- https://tools.ietf.org/html/rfc8037 Ed25519
