# letsencrypt-rs

[![Build Status](https://secure.travis-ci.org/onur/letsencrypt-rs.svg?branch=master)](https://travis-ci.org/onur/letsencrypt-rs)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/onur/letsencrypt-rs/master/LICENSE)
[![Crates.io](https://img.shields.io/crates/v/letsencrypt-rs.svg)](https://crates.io/crates/letsencrypt-rs)

Easy to use Let's Encrypt client and acme client library to issue and renew
TLS certs.

## Installation

You can install letsencrypt-rs with:
`cargo install --git https://github.com/onur/letsencrypt-rs`

`letsencrypt-rs` is currently using a specific version of openssl crate,
and it will soon be available in crates.io when openssl gets updated.


## Usage

letsencrypt-rs is using openssl library to generate all required keys
and certificate signing request. You don't need to run any openssl command.
But also, you can use your already generated keys and CSR if you want.
You don't need any root access while running letsencrypt-rs.

letsencrypt-rs is using simple HTTP validation to pass Let's Encrypt DNS
validation challenge. You need a working HTTP server to host challenge file.


### Easiest way to sign a certificate

```sh
letsencrypt-rs -s -D example.org -P /var/www -K domain.key -o domain.crt
```

This command will generate a user key, domain key and X509 certificate signing
request. It will register a new user account and identify domain ownership
by putting required challenge token into `/var/www/.well-known/acme-challenge/`.
And if everything goes well, it will save domain private key into `domain.key`
and signed certificate into `domain.crt`.

You can also use `--email` option to provide a contact addres on registration.



### Using your own keys and CSR

You can use your own RSA keys for user registration and domain. For example:

```sh
letsencrypt-rs \
  --sign \
  --user-key user.key \
  --domain-key domain.key \
  --domain-csr domain.csr \
  --domain example.org \
  -P /var/www \
  -o domain.crt
```

This will not generate any key and it will use provided keys to sign
certificate.


### Revoking a signed certificate

letsencrypt-rs can also revoke a signed certificate. You need to use your
user key and a signed certificate to revoke.

```sh
letsencrypt-rs \
  --revoke \
  --user-key user.key \
  --domain-crt signed.crt
```


## Options

You can get list of all available options with `letsencrypt-rs --help`:

```
letsencrypt-rs 0.1.0
Easy to use Let's Encrypt client to issue and renew TLS certs

USAGE:
    letsencrypt-rs [FLAGS] [OPTIONS]

FLAGS:
    -c, --chain      Chains the signed certificate with Let's Encrypt Authority X3 (IdenTrust
                     cross-signed) intermediate certificate.
    -r, --revoke     Revokes a signed certificate. This command requires --user-key and
                     --domain-csr options.
    -s, --sign       Signs a certificate. This is default behavior and this command requires
                     --domain and --public-dir options.
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -B, --bit-length <BIT_LENGHT>               Bit length for CSR. Default is 2048.
    -D, --domain <DOMAIN>                       Name of domain for identification.
    -T, --domain-crt <DOMAIN_CRT>               Path of signed domain certificate. This is required
                                                for revoke certificate.
    -C, --domain-csr <DOMAIN_CSR>               Path of domain certificate signing request.
        --domain-key <DOMAIN_KEY_PATH>          Domain private key path to use it in CSR
                                                generation.
    -E, --email <EMAIL>                         Contact email address (optional).
    -P, --public-dir <PUBLIC_DIR>               Directory to save ACME simple http challenge.
    -S, --save-csr <SAVE_DOMAIN_CSR>            Path to save domain certificate signing request.
    -K, --save-domain-key <SAVE_DOMAIN_KEY>     Path to save domain private key.
    -o, --save-crt <SAVE_SIGNED_CERTIFICATE>    Path to save signed certificate. Default is STDOUT.
        --save-user-key <SAVE_USER_KEY>         Path to save private user key.
    -U, --user-key <USER_KEY_PATH>              User private key path to use it in account
                                                registration.
```


## `acme-client` crate

`letsencrypt-rs` is powered by acme-client library. You can read documentation
in [docs.rs](https://docs.rs/acme-client). Example usage of `AcmeClient`:

```rust
AcmeClient::new()
    .and_then(|ac| ac.set_domain("example.org"))
    .and_then(|ac| ac.register_account(Some("contact@example.org")))
    .and_then(|ac| ac.identify_domain())
    .and_then(|ac| ac.save_http_challenge_into("/var/www"))
    .and_then(|ac| ac.simple_http_validation())
    .and_then(|ac| ac.sign_certificate())
    .and_then(|ac| ac.save_domain_private_key("domain.key"))
    .and_then(|ac| ac.save_signed_certificate("domain.crt"));
```
