# sudoforge/murmur [![badges-travis-ci]][travis-ci]

![badges-murmur] ![badges-docker-pulls] ![badges-docker-stars]

**Mumble** is a VOIP application which allows users to talk to each other via
the same server. It uses a client-server architecture, and encrypts all
communication to ensure user privacy. **Murmur** is the server that Mumble
clients to connect to. [Learn More][vendor-mumble].

`sudoforge/murmur` enables you to easily run multiple (lightweight) murmur
instances on the same host.

## Getting started

This guide assumes that you already have [Docker][docker-install-docs]
installed.

### Pull the official image

An image is available from the [Docker Hub][docker-hub-repo-url] registry, built
automatically from this repository. It's easy to get started:

```text
docker pull sudoforge/murmurd[:tag]
```

You don't _need_ to specify a tag, but it's a good idea to so that you don't
pull `latest` and risk getting different versions on different hosts. Versions
are kept in line with the [releases from mumble-voip/mumble][vendor-releases].

The examples throughout this document assume we are not using a tag for the sake
of brevity. If you pull the image with a tag other than `latest`, you will need
to use that tag number when running the image via `docker run`.

### Create a container

Now that you have the image pulled, it's time to get a container up and running.

```text
docker run -d \
    -p 64738:64738/tcp \
    -p 64738:64738/udp \
    --name murmur-001 \
    sudoforge/murmurd[:tag]
```

You should now be able to open up the Mumble client, and connect to the server
running at `127.0.0.1:64738`.

### Configuration options

The following variables can be passed into the container (when you execute
`docker run`) to change various configuration options.

For example:

```text
docker run -d \
    -p 64738:64738/tcp \
    -p 64738:64738/udp
    -e MUMBLE_SERVERPASSWORD='superSecretPasswordHere' \
    --name murmur-001 \
    sudoforge/murmurd[:tag]
```

Here is a list of all options supported through environment variables:

| Environment Variable | Default Value |
| -------------------- | ------------- |
| [`MUMBLE_ICE`][mdoc-ice] | `tcp -h 127.0.0.1 -p 6502` |
| [`MUMBLE_ICESECRETREAD`][mdoc-group-icesecret] | `---` |
| [`MUMBLE_ICESECRETWRITE`][mdoc-group-icesecret] | `---` |
| [`MUMBLE_AUTOBANATTEMPTS`][mdoc-group-autoban] | `10`    |
| [`MUMBLE_AUTOBANTIMEFRAME`][mdoc-group-autoban] | `120` |
| [`MUMBLE_AUTOBANTIME`][mdoc-group-autoban] | `300` |
| [`MUMBLE_SERVERPASSWORD`][mdoc-serverpassword] | `---` |
| [`MUMBLE_OBFUSCATE`][mdoc-obfuscate] | `false` |
| [`MUMBLE_SENDVERSION`][mdoc-sendversion] | `false`|
| [`MUMBLE_KDFITERATIONS`][mdoc-kdfIterations] | `-1`|
| [`MUMBLE_LEGACYPASSWORDHASH`][mdoc-legacyPasswordHash] | `false` |
| [`MUMBLE_ALLOWPING`][mdoc-allowping] | `true`|
| [`MUMBLE_BANDWIDTH`][mdoc-bandwidth] | `7200`|
| [`MUMBLE_TIMEOUT`][mdoc-timeout] | `30`|
| [`MUMBLE_USERS`][mdoc-users] | `100` |
| [`MUMBLE_USERSPERCHANNEL`][mdoc-usersperchannel] | `0` |
| [`MUMBLE_USERNAME`][mdoc-group-channelusername] | `[-=\\w\\[\\]\\{\\}\\(\\)\\@\\|\\.]+` |
| [`MUMBLE_CHANNELNAME`][mdoc-group-channelusername] | `[ \\-=\\w\\#\\[\\]\\{\\}\\(\\)\\@\\|]+` |
| [`MUMBLE_DEFAULTCHANNEL`][mdoc-defaultchannel] | `---` |
| [`MUMBLE_REMEMBERCHANNEL`][mdoc-rememberchannel] | `true`|
| [`MUMBLE_TEXTMESSAGELENGTH`][mdoc-textmessagelength] | `5000`|
| [`MUMBLE_IMAGEMESSAGELENGTH`][mdoc-imagemessagelength] |`131072` |
| [`MUMBLE_ALLOWHTML`][mdoc-allowhtml] | `true`|
| [`MUMBLE_OPUSTHRESHOLD`][mdoc-opusthreshold] | `100` |
| [`MUMBLE_REGISTERHOSTNAME`][mdoc-registerHostname] | `---` |
| [`MUMBLE_REGISTERPASSWORD`][mdoc-registerPassword] | `---` |
| [`MUMBLE_REGISTERURL`][mdoc-registerUrl] | `---` |
| [`MUMBLE_REGISTERNAME`][mdoc-registerName] | `---`|
| [`MUMBLE_SUGGESTVERSION`][mdoc-suggestVersion] | `false` |
| [`MUMBLE_SUGGESTPOSITIONAL`][mdoc-suggestPositional] | `---` |
| [`MUMBLE_SUGGESTPUSHTOTALK`][mdoc-suggestPushToTalk] | `---` |
| [`MUMBLE_ENABLESSL`](#ssl-certificates-murmurinissl) | `0` |
| [`MUMBLE_SSLPASSPHRASE`](#ssl-certificates-murmurinissl) | `---` |
| [`MUMBLE_SSLCIPHERS`](#ssl-certificates-murmurinissl) | `---` |
| `SUPERUSER_PASSWORD` | If not defined, a password will be auto-generated. |

### Custom welcome text ([Murmur.ini::welcometext][mdoc-welcometext])

To customize the welcome text, add the contents to `welcometext` and mount that
into the container at `/data/welcometext`. Double quote characters (`"`) are
escaped automatically, but you may want to confirm that your message was parsed
correctly.

### SSL Certificates ([Murmur.ini::SSL][mdoc-sslcertkey])

Murmur will generate its own SSL certificates when the daemon is started. If you
wish to provide your own certificates and ciphers instead, you can do so by
following the instructions below.

If `MUMBLE_ENABLESSL` is set to `1`, custom SSL is enabled, as long as you have
mounted a certificate and key at the following locations:

- SSL certificate should be mounted at `/data/cert.pem`

  - If your certificate is signed by an authority that uses a sub-signed or
    "intermediate" certificate, you should either bundle that with your
    certificate, or mount it in separately at `/data/intermediate.pem` - this
    will be automatically detected.

- SSL key should be mounted at `/data/key.pem`

  - If the key has a passphrase, you should define the environment variable
    `MUMBLE_SSLPASSPHRASE` with the passphrase. This variable does not have any
    effect if you have not mounted a key *and* enabled SSL.

- Set your preferred cipher suite using `MUMBLE_SSLCIPHERS`

  - This option chooses the cipher suites to make available for use in SSL/TLS.
    See the [official documentation][mdoc-sslCiphers] for more information.

### Logging in as SuperUser

If the environment variable `SUPERUSER_PASSWORD` is not defined when creating
the container, a password will be automatically generated. To view the password
for any container at any time, look at the container's logs. As an example, to
view the SuperUser password is for an instance running in a container named
`murmur-001`:

```text
$ docker logs murmur-001 2>&1 | grep SUPERUSER_PASSWORD
> SUPERUSER_PASSWORD: <value>
```

## Tagging policy

This project does not overwrite tags; they can (and should) be treated as
immutable references, although you should still use image digests when deploying
or extending this image.

The `latest` tag follows the `master` branch of this repository and has
equivalent layers to the latest numbered tag.

### Numbered tags

For a full list of tags, please see the [tags page][tags] on Docker Hub.

Numbered tags follow the pattern:

```
<MUMBLE_VERSION>-<RELEASE>
  │                └─ the release number specific to this repository
  │
  └──── the version of mumble for this release
```



---

![badges-analytics]

[badges-murmur]: https://img.shields.io/badge/murmur-1.3.0-green.svg?maxAge=2592000 "Murmur v1.3.0"
[badges-docker-pulls]: https://img.shields.io/docker/pulls/sudoforge/murmurd.svg "Docker Pulls"
[badges-docker-stars]: https://img.shields.io/docker/stars/sudoforge/murmurd.svg "Docker Stars"
[badges-travis-ci]: https://travis-ci.org/sudoforge/docker-images.svg?branch=master "Build Status"
[travis-ci]: https://travis-ci.org/sudoforge/docker-images
[badges-analytics]: https://ga-beacon.appspot.com/UA-98603156-1/github-landing-page?flat "Analytics"
[repo-url]: https://www.github.com/sudoforge/docker-images
[releases]: https://www.github.com/sudoforge/docker-images/releases
[vendor-releases]: https://www.github.com/mumble-voip/mumble/releases
[vendor-mumble]: http://wiki.mumble.info/wiki/Main_Page "Learn About Mumble"
[docker-install-docs]: https://docs.docker.com/engine/installation/ "Docker Installation Docs"
[docker-hub-repo-url]: https://hub.docker.com/r/sudoforge/murmur/ "View on DockerHub"
[mdoc-ice]: https://wiki.mumble.info/wiki/Murmur.ini#ice
[mdoc-group-icesecret]: https://wiki.mumble.info/wiki/Murmur.ini#icesecretread_and_icesecretwrite
[mdoc-group-autoban]: https://wiki.mumble.info/wiki/Murmur.ini#autobanAttempts.2C_autobanTimeframe_and_autobanTime
[mdoc-serverpassword]: https://wiki.mumble.info/wiki/Murmur.ini#serverpassword
[mdoc-obfuscate]: https://wiki.mumble.info/wiki/Murmur.ini#obfuscate
[mdoc-sendversion]: https://wiki.mumble.info/wiki/Murmur.ini#sendversion
[mdoc-legacyPasswordHash]: https://wiki.mumble.info/wiki/Murmur.ini#legacyPasswordHash
[mdoc-kdfiterations]: https://wiki.mumble.info/wiki/Murmur.ini#kdfIterations
[mdoc-allowping]: https://wiki.mumble.info/wiki/Murmur.ini#allowping
[mdoc-welcometext]: https://wiki.mumble.info/wiki/Murmur.ini#welcometext
[mdoc-bandwidth]: https://wiki.mumble.info/wiki/Murmur.ini#bandwidth
[mdoc-timeout]: https://wiki.mumble.info/wiki/Murmur.ini#timeout
[mdoc-users]: https://wiki.mumble.info/wiki/Murmur.ini#users
[mdoc-usersperchannel]: https://wiki.mumble.info/wiki/Murmur.ini#usersperchannel
[mdoc-group-channelusername]: https://wiki.mumble.info/wiki/Murmur.ini#channelname_and_username
[mdoc-defaultchannel]: https://wiki.mumble.info/wiki/Murmur.ini#defaultchannel
[mdoc-rememberchannel]: https://wiki.mumble.info/wiki/Murmur.ini#rememberchannel
[mdoc-textmessagelength]: https://wiki.mumble.info/wiki/Murmur.ini#textmessagelength
[mdoc-imagemessagelength]: https://wiki.mumble.info/wiki/Murmur.ini#imagemessagelength
[mdoc-allowhtml]: https://wiki.mumble.info/wiki/Murmur.ini#allowhtml
[mdoc-opusthreshold]: https://wiki.mumble.info/wiki/Murmur.ini#opusthreshold
[mdoc-registerHostname]: https://wiki.mumble.info/wiki/Murmur.ini#registerHostname
[mdoc-registerPassword]: https://wiki.mumble.info/wiki/Murmur.ini#registerPassword
[mdoc-registerUrl]: https://wiki.mumble.info/wiki/Murmur.ini#registerUrl
[mdoc-registerName]: https://wiki.mumble.info/wiki/Murmur.ini#registerName
[mdoc-suggestVersion]: https://wiki.mumble.info/wiki/Murmur.ini#suggestVersion
[mdoc-suggestPositional]: https://wiki.mumble.info/wiki/Murmur.ini#suggestPositional
[mdoc-suggestPushToTalk]: https://wiki.mumble.info/wiki/Murmur.ini#suggestPushToTalk
[mdoc-sslcertkey]: https://wiki.mumble.info/wiki/Murmur.ini#sslCert_and_sslKey
[mdoc-sslCiphers]: https://wiki.mumble.info/wiki/Murmur.ini#sslCiphers
[tags]: https://hub.docker.com/r/sudoforge/murmurd/tags "sudoforge/murmurd tags"
