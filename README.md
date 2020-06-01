# serverless rust [![Build Status](https://github.com/softprops/serverless-rust/workflows/Main/badge.svg)](https://github.com/softprops/serverless-rust/actions) [![npm](https://img.shields.io/npm/v/serverless-rust.svg)](https://www.npmjs.com/package/serverless-rust)


> A ⚡ [Serverless framework](https://www.serverless.com/framework/docs/) ⚡ plugin for [Rustlang](https://www.rust-lang.org/) applications 🦀

## 📦 Install

Install the plugin with npm

```sh
$ npm i -D serverless-rust
```

💡 This serverless plugin assumes you are building Rustlang lambdas targeting the AWS Lambda "provided" runtime. The [AWS Lambda Rust Runtime](https://github.com/awslabs/aws-lambda-rust-runtime) makes this easy.

Add the following to your serverless project's `serverless.yml` file

```yaml
service: demo
provider:
  name: aws
  runtime: rust
plugins:
  # this adds informs serverless to use
  # the serverless-rust plugin
  - serverless-rust
# creates one artifact for each function
package:
  individually: true
functions:
  test:
    # handler value syntax is `{cargo-package-name}.{bin-name}`
    # or `{cargo-package-name}` for short when you are building a
    # default bin for a given package.
    handler: your-cargo-package-name
    events:
      - http:
          path: /test
          method: GET
```

> 💡 The Rust Lambda runtime requires a binary named `bootstrap`. This plugin renames the binary cargo builds to `bootstrap` for you before packaging. You do **not** need to do this manually in your Cargo.toml configuration file.

## 🖍️ customize

You can optionally adjust the default settings of the dockerized build env using
a custom section of your serverless.yaml configuration

```yaml
custom:
  # this section allows for customization of the default
  # serverless-rust plugin settings
  rust:
    # flags passed to cargo
    cargoFlags: '--features enable-awesome'
    # custom docker tag
    dockerTag: 'some-custom-tag'
    #  custom docker image
    dockerImage: 'dockerUser/dockerRepo'
```

### 🥼 (experimental) local builds

While it's useful to have a build environment match your deployment
environment, dockerized builds do come with some notable tradeoffs.

The external dependency on docker itself often causes friction as an added dependency to your build. Depending on the docker image limited which versions of rust you could deploy with. The docker image tracked stable rust. Some users might wish to try unstable versions early. Local builds enable that.

If you wish to build lambda's locally, this plugin also supports an experimental `dockerless` mode. 

```diff
custom:
  # this section allows for customization of the default
  # serverless-rust plugin settings
  rust:
    # flags passed to cargo
    cargoFlags: '--features enable-awesome'
    # experimental! when set to true, artifacts are built locally outside of docker
+   dockerless: true
```

This will build and link your lambda as a static binary that can be deployed in to the lambda execution environment as a static binary using [MUSL](https://doc.rust-lang.org/edition-guide/rust-2018/platform-and-target-support/musl-support-for-fully-static-binaries.html).

In order to use this mode its expected that you install the `x86_64-unknown-linux-musl` target on all platforms locally with

```sh
$ rustup target add x86_64-unknown-linux-musl
```

On linux platforms, you will need to install musl-tools

```sh
$ sudo apt-get update && sudo apt-get install -y musl-tools
```

On Mac OSX, you will need to install a MUSL cross compilation toolchain

```sh
$ brew install filosottile/musl-cross/musl-cross
```

Using MUSL comes with some other notable tradeoffs. One of which is complications that arise when depending on dynamically linked dependencies.

* With OpenSSL bindings which you can safely replace is with [rustls](https://github.com/ctz/rustls) or [vendor it](https://docs.rs/openssl/0.10.29/openssl/#vendored)
* Other known issues have been reported with [backtrace](https://github.com/KodrAus/rust-cross-compile#limitations). 

If you find other MUSL specific issues, please reported them by [opening an issue](https://github.com/softprops/serverless-rust/issues/new?template=bug_report.md).

### 🎨 Per function customization

If your serverless project contains multiple functions, you may sometimes
need to customize the options above at the function level. You can do this
by defining a `rust` key with the same options inline in your function
specification.

```yaml
functions:
  test:
    rust:
      # function specific flags passed to cargo
      cargoFlags: '--features enable-awesome'
    # handler value syntax is `{cargo-package-name}.{bin-name}`
    # or `{cargo-package-name}` for short when you are building a
    # default bin for a given package.
    handler: your-cargo-package-name
    events:
      - http:
          path: /test
          method: GET
```

## 🤸 usage

Every [serverless workflow command](https://serverless.com/framework/docs/providers/aws/guide/workflow/) should work out of the box.

### invoke your lambdas locally

```sh
$ npx serverless invoke local -f hello -d '{"hello":"world"}'
```

### deploy your lambdas to the cloud

```sh
$ npx serverless deploy
```

### invoke your lambdas in the cloud directly

```sh
$ npx serverless invoke -f hello -d '{"hello":"world"}'
```

### view your lambdas logs

```sh
$ npx serverless logs -f hello
```


## 🏗️ serverless templates

### ^0.2.*

* a minimal echo application - https://github.com/softprops/serverless-aws-rust
* a minimal http application - https://github.com/softprops/serverless-aws-rust-http
* a minimal multi-function application - https://github.com/softprops/serverless-aws-rust-multi
* a minimal apigateway websocket application - https://github.com/softprops/serverless-aws-rust-websockets
* a minimal kinesis application - https://github.com/softprops/serverless-aws-rust-kinesis

### 0.1.*

Older versions targeted the python 3.6 AWS Lambda runtime and [rust crowbar](https://github.com/ilianaw/rust-crowbar) and [lando](https://github.com/softprops/lando) applications

* lando api gateway application - https://github.com/softprops/serverless-lando
* multi function lando api gateway application - https://github.com/softprops/serverless-multi-lando
* crowbar cloudwatch scheduled lambda application - https://github.com/softprops/serverless-crowbar

Doug Tangren (softprops) 2018-2019
