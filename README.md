# AWS S3 Reverse Proxy

The `aws-s3-proxy` will reverse-proxy all incoming S3 API calls to the public
AWS S3 backend by rewriting the Host header and re-signing the original request.

AWS uses its [Signature
v4]((https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html)) to
protect API access. All requests sent to AWS S3 have to have a valid signature
which includes your AWS security credentials (commonly known as
`AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`). This signature is encoded in
the HTTP Authorization header and also contains a SHA256 of various other HTTP
headers, including the `Host` header, which directly corresponds to the
`endpoint_url`. Since we want to reverse-proxy requests, we need to rewrite this
header, and therefore need to re-sign the request.

In order to re-sign any Signature v4 request, we need the full set of AWS
security credentials, the `AWS_ACCESS_KEY_ID` is already part of the
`Authorization`, but the `AWS_SECRET_ACCESS_KEY` needs to be provided as
configuration option to the reverse proxy.

## Features

  * can reverse-proxy to a configurable AWS Region
  * limits access based on source IP and subnet of the client
  * limits access based on endpoint URL
  * full instrumentation with Prometheus metrics
  * HTTP and HTTPS support for clients
  * uses secure HTTPS for upstream connections by default
  * run as single binary or Docker container
  * configuration via CLI, or using the same options in a config file

## Getting Started

The proxy uses the official AWS SDK for Go for most of the heavy lifting. More
information can be found in the [developer
guide](https://docs.aws.amazon.com/sdk-for-go/v1/developer-guide/configuring-sdk.html)

The available options and help information can be displayed with:
```
docker run --rm aws-s3-reverse-proxy --help
```

# Build
All build dependencies and steps are contained in the `Dockerfile`:
```
docker build -t aws-s3-reverse-proxy .
```

# Run

## Server Examples
```
$ docker run --rm -ti \
  -p 8099 \
  aws-s3-reverse-proxy
  --allowed-source-addr=192.168.1.0/24
  --allowed-endpoint=my.host.example.com:8099
  --aws-credentials=AWS_ACCESS_KEY_ID,AWS_SECRET_ACCESS_KEY
```

Or you can use a config file:
```
# config.cfg file for aws-3-reverse-proxy
--allowed-source-addr=192.168.1.0/24
--allowed-endpoint=my.host.example.com:8099
--aws-credentials=AWS_ACCESS_KEY_ID,AWS_SECRET_ACCESS_KEY
```
And then run it like:
```
$ docker run --rm -it -v $(pwd)/config.cfg:/config.cfg -p 8099 aws-s3-reverse-proxy @config.cfg -v
```

Or just run the binary the old-fashioned way:
```
./aws-s3-reverse-proxy --help
```

## Client Examples

Client with the [official awscli](https://aws.amazon.com/cli/):
```
$ aws s3 --endpoint-url http://my.host.example.com:8099 ls s3://my-bucket/
```

# Contributing

`aws-s3-reverse-proxy` welcomes contributions from anyone! Unlike many other
projects we are happy to accept cosmetic contributions and small contributions,
in addition to large feature requests and changes.

# License

`aws-s3-reverse-proxy` is made available under the MIT License. For more
details, see the `LICENSE` file in the repository.

# Authors

``aws-s3-reverse-proxy`` was created by Thomas Kriechbaumer, and is maintained
by the community.