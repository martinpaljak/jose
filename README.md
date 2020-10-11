# JOSE
Usage of JOSE (JWK, JWS, JWT) for authentication and messaging

## JWT: X509 WebAuth
Profile of OpenID ID Token.



## Notarized JWK-s / Subkeys (JWT)

Public key pinning

`"npk"` - Notarized Public Key

When used in header: contains the JWT (XX.XX.XX string) with `"npk"` in payload

When used in payload: contains the minimal representation of the public key as JSON structure

JSON Web Key (JWK) Thumbprint https://tools.ietf.org/html/rfc7638

Self-describing NPK should have the public key x 

## 
