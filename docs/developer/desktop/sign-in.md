---
hide_title: true
title: Sign-in with Idena protocol
sidebar_label: Sign-in with Idena
---

# Sign-in with Idena protocol

The sign-in with Idena function can be used on your website to check whether a user has multiple accounts. The Idena app pops up automatically when the user clicks `dna://signin/...` link (see more about [Idena app URL scheme](./dna-url)). The following dialog appears in Idena app:

![image](https://user-images.githubusercontent.com/47352542/105166500-5444f180-5b39-11eb-9292-c4a65c9610be.png)

When the user clicks the `Confirm` button specific endpoints (`nonce_endpoint` and `authentication_endpoint`) will be called on your server and the `callback_url` will be opened in the user's browser:

![image](https://user-images.githubusercontent.com/47352542/105168390-e51ccc80-5b3b-11eb-903c-b78399ae3052.png)

As a result your user's Idena address will be known to your server. You can check the validation `state` of the user's address using the [API](http://api.idena.io/api/swagger/index.html#/Identity/Identity) or the Idena node RPC to grant privileges to the user on your website.

## `signin` app URL example

URL example for signing in with the Idena public address:

```
dna://signin/v1?token=session_token&
            callback_url=https%3A%2F%2Fmywebsite.com&
            nonce_endpoint=https%3A%2F%2Fmywebsite.com%2Fauth%2Fv1%2Fstart-session&
            authentication_endpoint=https%3A%2F%2Fmywebsite.com%2Fauth%2Fv1%2Fauthenticate
            favicon_url=https%3A%2F%2Fmywebsite.com%2Ffavicon.ico
```

1. `token`: GUID string (can be generated in the client's browser).

2. `nonce_endpoint`: specifies url for the POST method to get a random nonce from the website server.

Successful response with a random nonce has to be provided. Nonce must have `signin-` prefix.
See example `POST /start-session` method below.

3. `authentication_endpoint`: specifies url for the POST method for the authentication.

Successful response with `authenticated` flag has to be provided.
See example `POST /authenticate` method below.

4. `callback_url`: specifies url that will be opened in the client's browser automatically after successful authentication.

5. `favicon_url`: specifies custom url for the icon displayed for user in the Idena app (optional parameter).

### `POST /start-session` method

Request body example:

```
 {
  "token": "428489af-3ca1-4861-b1c7-5f634f6466e2",
  "address": "0xFf893698faC953dBbCdC3276e8aD13ed3267fB06"
}
```

Successful response example:

```
{
  "success": true,
  "data": {
    "nonce": "signin-0652c409-17ef-4ad6-b580-3faaefcc204d"
  }
}
```

Nonce provided in the response data must have `signin-` prefix.

Fail response example:

```
{
  "success": false,
  "error": "This is a error message"
}
```

### `POST /authenticate` method

Request body example:

```
{
  "token": "428489af-3ca1-4861-b1c7-5f634f6466e2",
  "signature": "0xe0434ea8ff5123a570b6b7e5f1b837af4524372d4552021bfcede66219abe00c
                376a8c8417299be23938b9644ba922ffd36bbbdd1cdf15719da9b2af9affdec601"
}

```

Successful response must be returned if user's address is equal to address derived from the signature (e.g. [function signatureAddress](https://github.com/idena-network/idena-auth/blob/30c3f8f5406592bb297bd78a4e931ed2228d017b/core/auth.go#L109) to get address from the nonce signature). [Ethereum utils](https://github.com/ethereumjs/ethereumjs-util) can be used for signature verification as following:

```
 import {
   bufferToHex,
   ecrecover,
   fromRpcSig,
   keccak256,
   pubToAddress,
   rlp,
 } from 'ethereumjs-util'
   ...
   const nonce = 'signin-0652c409-17ef-4ad6-b580-3faaefcc204d'
   const signature = '0xe0434ea8ff...'

   const nonceHash = keccak256(keccak256(Buffer.from(nonce, 'utf-8'))
   // nonceHash = [66 67 192 202 176 90 98 74 227 105 104 190 23 230 249 49
   //              18 228 90 68 91 187 217 203 235 31 142 216 170 106 51 103]
   // or 0x4243c0cab05a624ae36968be17e6f93112e45a445bbbd9cbeb1f8ed8aa6a3367

   const {v, r, s} = fromRpcSig(signature)
   // v = 28
   // r = [224 67 78 168 255 81 35 165 112 182 183 229 241 184 55 175
   //      69 36 55 45 69 82 2 27 252 237 230 98 25 171 224 12]
   //   or 0xe0434ea8ff5123a570b6b7e5f1b837af4524372d4552021bfcede66219abe00c
   // s = [55 106 140 132 23 41 155 226 57 56 185 100 75 169 34 255 211
   //      107 187 221 28 223 21 113 157 169 178 175 154 255 222 198]
   //   or 0x376a8c8417299be23938b9644ba922ffd36bbbdd1cdf15719da9b2af9affdec6

   const pubKey = ecrecover(nonceHash, v, r, s)
   // [104 195 68 18 145 186 221 100 156 89 197 34 219 60 124 28 74 241 86 219 81
       0 252 0 196 246 79 96 197 29 60 3 28 252 139 19 129 58 97 237 192 165 118 174
       182 186 69 1 38 212 194 86 203 103 164 137 3 190 135 111 164 219 210 101]
   // or 0x68c3441291badd649c59c522db3c7c1c4af156db5100fc00c4f64f60c51d3c031cfc8b13813a6
   //    1edc0a576aeb6ba450126d4c256cb67a48903be876fa4dbd265

   const addrBuf = pubToAddress(pubKey)
   // addrBuf = [83 234 239 254 48 93 154 211 143 109 177 4 205 227 212 130 42 96 191 77]
   //        or 0x53eaeffe305d9ad38f6db104cde3d4822a60bf4d

   const addr = bufferToHex(addrBuf)
   return addr
```

Successful response example:

```
{
  "success": true,
  "data": {
    "authenticated": true
  }
}
```

Failed authentication response example:

```
{
  "success": true,
  "data": {
    "authenticated": false
  }
}
```

Fail response example:

```
{
  "success": false,
  "error": "This is a error message"
}
```

## Additional methods

These methods are not used for the Idena authentication protocol.

### `GET /get-account` method

Request example:

```
/get-account?token=428489af-3ca1-4861-b1c7-5f634f6466e2
```

Successful response example:

```
{
  "success": true,
  "data": {
    "address": "0xFf893698faC953dBbCdC3276e8aD13ed3267fB06"
  }
}
```

Fail response example:

```
{
  "success": false,
  "error": "This is a error message"
}
```

### `POST /logout` method

Request body example:

```
{
  "token": "428489af-3ca1-4861-b1c7-5f634f6466e2",
}
```

Successful response example:

```
{
  "success": true,
  "data": {
    "loggedout": true
  }
}
```

Failed log out response example:

```
{
  "success": true,
  "data": {
    "loggedout": false
  }
}
```

Fail response example:

```
{
  "success": false,
  "error": "This is a error message"
}
```