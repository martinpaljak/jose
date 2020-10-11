# JOSE
Usage of JOSE (JWK, JWS, JWT) for authentication and messaging

## JWT: X509 WebAuth
Profile of OpenID ID Token.



## Notarized JWK-s / Subkeys (JWT)

Public key pinning / `"npk"` - Notarized Public Key

Given a master key with JWK thumbprint `gNcTLfHQVkVbnoUSHezrNYdACB8G215Yhuz8v-WQ73s`
```
{
  "crv" : "Ed25519",
  "kty" : "OKP",
  "x" : "dLz8B-W06uOuzFFzywK_aWQsMK5siPi6YRU10vYzvO8"
}
```

Given a subkey with JWK thumbprint `WgL_9NpEmJtUBQ8q-Ff3txkU2vInbZxFF9rxeintiRU`
```
{
  "crv" : "Ed25519",
  "kty" : "OKP",
  "x" : "wy8hrsVOUxT6eGdQnccdVNbx7JnV7oqiSGMY2kmGgjo"
}
```
Will produce this kind of JWT for the `"npk"`
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
- "iat", "nbf" and "exp" MUST be present
- "iss" SHOULD be present and represent the "x" of EdDSA public key for self-contained NPK-s
- "kid" SHOULD be present and represent the JWT thumbprint of a pinned public key.


When used signing the following:
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
will yield an actual JWS of `eyJraWQiOiJXZ0xfOU5wRW1KdFVCUThxLUZmM3R4a1UydkluYlp4RkY5cnhlaW50aVJVIiwibnBrIjoiZXlKMGVYQWlPaUpLVjFRaUxDSnJhV1FpT2lKblRtTlVUR1pJVVZaclZtSnViMVZUU0dWNmNrNVpaRUZEUWpoSE1qRTFXV2gxZWpoMkxWZFJOek56SWl3aVlXeG5Jam9pUldSRVUwRWlmUS5leUpwYzNNaU9pSmtUSG80UWkxWE1EWjFUM1Y2UmtaNmVYZExYMkZYVVhOTlN6VnphVkJwTmxsU1ZURXdkbGw2ZGs4NElpd2libkJySWpwN0ltTnlkaUk2SWtWa01qVTFNVGtpTENKcmRIa2lPaUpQUzFBaUxDSjRJam9pZDNrNGFISnpWazlWZUZRMlpVZGtVVzVqWTJSV1RtSjROMHB1VmpkdmNXbFRSMDFaTW10dFIyZHFieUo5TENKcFlYUWlPakUyTURJME1EWXdNVEFzSW01aVppSTZNVFl3TWpRd05qQXhNQ3dpWlhod0lqb3hOakF5TkRreU5ERXdmUS4xNHN4MjhfNlNTSEd3RXBoSEIzZGRYSEhCVmVxRFRKR1hvdlFTWVhadjNzdmRudGYtYmhaTXBfaGZqa25GRWRZWVNtSm1zNzgtRzlyTURadl9Db2NBUSIsImFsZyI6IkVkRFNBIn0.eyJIZWxsbyI6IldvcmxkIiwibm92ZWx0eSI6ZmFsc2V9.9LVJvJrJNGGlozLYhts3G-AetgMeBmmTVAoKlt9-J7OEOPM8jLvL_dHBHEEOyXxdUxhxl7DAWyra4sWTehioDw`


NOTES
- "kid" MAY be present in header
- when "kid" is present in header, it MUST match the "kid" of the "npk"
