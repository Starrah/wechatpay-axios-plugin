# Wechatpay APIv3 [Axios](https://github.com/axios/axios) Plugin

[![NPM version](https://badge.fury.io/js/wechatpay-axios-plugin.svg)](http://badge.fury.io/js/wechatpay-axios-plugin)
[![npm module downloads per month](http://img.shields.io/npm/dm/wechatpay-axios-plugin.svg)](https://www.npmjs.com/package/wechatpay-axios-plugin)


## Features

- [x] The Node native code of the wechatpay APIv3's AES cryptography(`aes-256-gcm`) encrypt/decrypt
- [x] The Node native code of the wechatpay APIv3's RSA cryptography(`sha256WithRSAEncryption` with `RSA_PKCS1_OAEP_PADDING`) encrypt/decrypt/sign/verify
- [x] Most of the APIv3's GET/POST requests, except the wechatpay public certificates download
- [x] The APIv3's media file upload is out, optional dependency on [form-data](https://github.com/form-data/form-data)

## Installing

`$ npm install wechatpay-axios-plugin`

## Requirements

The `oaepHash` used in `Rsa.encrypt` and `Rsa.decrypt` were added on Node v12.9.0. So that the Node minimum version should be 12.9.0(I'm not very sure).

## Examples

### Initialization

```js
const axios = require('axios')
const wxp = require('wechatpay-axios-plugin')
const {readFileSync} = require('fs')

const merchantPrivateKey  = readFileSync('/your/home/hellowechatpay/apiclient_key.pem')
const wechatpayPublicCert = '-----BEGIN CERTIFICATE-----' + '...' + '-----END CERTIFICATE-----'

const instance = axios.create({
  baseURL: 'https://api.mch.weixin.qq.com',
})

const client = wxp(instance, {
  mchid: 'your_merchant_id',
  serial: 'serial_number_of_your_merchant_public_cert',
  privateKey: merchantPrivateKey,
  certs: {
    'serial_number': wechatpayPublicCert,
  }
})
```

**Note:** The `readFileSync` function is not mandatares, you may passed the plaintext certs such as `wechatpayPublicCert` sample.

### Promise style

#### POST `/v3/combine-transactions/jsapi` with `JSON` body payload

```js
client.post('/v3/combine-transactions/jsapi', {}).then(response => {
  console.info(response)
}, error => {
  console.error(error)
})
```

#### POST `/v3/smartguide/guides/{guide_id}/assign` mixed `RESTful` parameter with `JSON` body payload, response as `204` status code.

```js
client.post(`/v3/smartguide/guides/${guide_id}/assign`, {
  sub_mchid,
  out_trade_no,
}).catch(error => {
  console.error(error)
})
```

#### POST `/v3/marketing/favor/media/image-upload` image file uploading

```js
const FormData = require('form-data')
const {createReadStream} = require('fs')

const imageMeta = {
  filename: 'hellowechatpay.png',
  // easy calculated by the command `sha256sum hellowechatpay.png` on OSX
  sha256: '1a47b1eb40f501457eaeafb1b1417edaddfbe7a4a8f9decec2d330d1b4477fbe',
}

const imageData = new FormData()
imageData.append('meta', JSON.stringify(imageMeta), {contentType: 'application/json'})
imageData.append('file', createReadStream('./hellowechatpay.png'))

client.post('/v3/marketing/favor/media/image-upload', imageData, {
  meta: imageMeta,
  headers: imageData.getHeaders()
}).then(res => {
  console.info(res.data.media_url)
}).catch(error => {
  console.error(error)
})
```

### Async/Await style

#### GET `/v3/merchant-service/complaints` with `query` parameters

```js
(async () => {
  try {
    const res = await client.get('/v3/merchant-service/complaints', {
      params: {
        limit      : 5,
        offset     : 0,
        begin_date : '2020-03-07',
        end_date   : '2020-03-14',
      }
    })
    console.info(res.data)
  } catch (error) {
    console.error(error)
  }
})()
```

#### POST `/v3/pay/partner/transactions/native` with `JSON` body payload

```js
(async () => {
  try {
    const res = await client.post('/v3/pay/partner/transactions/native', {
      sp_appid,
      sp_mchid,
      sub_mchid,
      description,
      out_trade_no,
      time_expire: new Date( (+new Date) + 33*60*1000 ), //after 33 minutes
      attach,
      notify_url,
      amount: {
        total: 1,
      }
    })
    console.info(res.data.code_url)
  } catch (error) {
    console.error(error)
  }
})()
```

#### GET `/v3/marketing/favor/stocks/{stock_id}` mixed `query` with `RESTful` parameters

```js
(async () => {
  try {
    const res = await client.post(`/v3/marketing/favor/stocks/${stock_id}`, {
      params: {
        stock_creator_mchid,
      }
    })
    console.info(res.data)
  } catch(error) {
    console.error(error)
  }
})()
```

#### POST `/v3/marketing/partnerships/build` with special `Header` field parameter

```js
(async () => {
  try {
    const res = await client.post(`/v3/marketing/partnerships/build`, {
      partner: {
        type,
        appid
      },
      authorized_data: {
        business_type,
        stock_id
      }
    }, {
      headers: {
        [`Idempotency-Key`]: 12345
      }
    })
    console.info(res.data)
  } catch (error) {
    console.error(error)
  }
})()
```

#### POST `/v3/merchant/media/video_upload` video file uploading

```js
const FormData = require('form-data')
const {createReadStream} = require('fs')

(async () => {
  const videoMeta = {
    filename: 'hellowechatpay.mp4',
    // easy calculated by the command `sha256sum hellowechatpay.mp4` on OSX
    sha256: '1a47b1eb40f501457eaeafb1b1417edaddfbe7a4a8f9decec2d330d1b4477fbe',
  }

  const videoData = new FormData()
  videoData.append('meta', JSON.stringify(videoMeta), {contentType: 'application/json'})
  videoData.append('file', createReadStream('./hellowechatpay.mp4'))

  try {
    const res = await client.post('/v3/merchant/media/video_upload', videoData, {
      meta: videoMeta,
      headers: videoData.getHeaders()
    })
    console.info(res.data.media_id)
  } catch (error) {
    console.error(error)
  }
})()
```

You may find some advance usage examples via the [Axios](https://github.com/axios/axios) project.

## TODO

- [ ] documentation
- [ ] coding comments
- [ ] certificates download

## License

The MIT License (MIT)

Copyright (c) 2020 James ZHANG(TheNorthMemory)

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
