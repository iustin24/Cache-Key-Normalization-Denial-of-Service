```ceylon
GET /static/main.js HTTP/1.1
Host: redacted.com
Authorization: AWS4-HMAC-SHA256 Credential=AKIAIOSFODNN7EXAMPLE/20130524/us-east-1/s3/aws4_request, SignedHeaders=host;range;x-amz-date, Signature=fe5f80f77d5fa3beca038a248ff027d0445342fe2855ddc963176630326f1024
x-amz-content-sha256: STREAMING-AWS4-HMAC-SHA256-PAYLOAD
```
```ceylon
HTTP/1.1 403 Forbidden
x-amz-request-id: 2S66W5WD44PXMHB5
x-amz-id-2: 2xrm46zxtG0n9ZSQhLFxaIPjxq7l9FdEWrWK4/nAVABowlSaGOIXCtcDkBeN+Tn+APojKMVkhhw=
CF-Cache-Status: HIT
Age: 2
```
