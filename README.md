# Sigstore the Hard Way

Welcome to sigstore the hard way.

The driver for this project is to get potential users, developers or collaborators familiar with the inner workings of
sigstore's infrastructure.

To best achieve a good familiarity with sigstore, we will walk through the whole process manually. This will help
provide a view of how each component project in sigstore glues together, while deliberately avoiding automation.  This
means no Dockerfiles or deployment framework playbooks. Everything is set up manually.

With 'Sigstore the Hard Way' we will install, configure and run the following components to provide a 'keyless' signing
infrastructure.

1. [Fulcio](https://github.com/sigstore/fulcio) WebPKI
2. [Rekor](https://github.com/sigstore/rekor), signature transparency log and timestamping authority
3. [Certificate Transparency Log](https://github.com/google/certificate-transparency-go/tree/master/trillian)
4. [Dex](https://github.com/dexidp/dex), OpenID Connect provider
5. [Cosign](https://github.com/sigstore/cosign), container (and more) signing and verifying tool

## Requirements

This tutorial leverages the [GCP](https://cloud.google.com/) for the provisioning of the compute
infrastructure required to bootstrap the sigstore infra from the ground up.
Free credits are available on [Sign up](https://cloud.google.com/free/). For when it comes to saving costs, the recommendation
is to shutdown any instances when you're not using them and once you have completed the tutorial, delete
all the instances, networks etc.

You can of course use local machines if you have them, or any other provider such as AWS, Azure (pull requests welcomed!)

The only other requirement is a domain name, where you have the ability to create some subdomains. We need a domain
for an OpenID Connect session (providers don't always like redirect_urls to IP addresses). It's up to you who you use, any provider will do. If you already have a domain, it makes sense to use that. We won't be messing with the root domain if you're already running something there, just creating subdomains (e.g. rekor.example.com, fulcio.example.com)



## Certificate Authority

For the Certificate Authority we will have three options to choose from:

* File CA
* [SoftHSM](http://www.softhsm.org/)
* Google's Certificate Transparency Service

The above are listed in order of setup ease. If you just want to kick the tyres and don't need a secure CA, you can use the File CA.

Google's is a paid service, but easy to set up. SoftHSM is completely free, but requires a little more setup (but nothing
too challenging)

Last of all we will sign a container image using cosign.

## Caveats

> :warning: sigstore is not ready for production! Use is at your own risk! There will be bugs (tell us about them!)

### Copyright

If you have not guessed by name, this is based off, and comes with credit to [Kelsey Hightower's Kubernetes the Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img ealt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.

## Chapters

* [Prerequisites](docs/01-prerequisites.md)
* [Provisioning Compute Resources](docs/02-compute-resources.md)
* [Domain Configuration](docs/03-domain-configuration.md)
* [Install Rekor](docs/04-rekor.md)
* [Install Dex](docs/05-dex.md)
* [Install Fulcio](docs/06-fulcio.md)
* [Install CTL](docs/07-certifcate-transparency.md)
* [Configure Container Registry](docs/08-configure-registry.md)
* [Install cosign](docs/09-cosign.md)
* [Sign a container image](docs/10-sign-container.md)

## Having issues, not working?

Raise an issue (best option, as others can learn) or message me on the [sigstore slack](https://join.slack.com/t/sigstore/shared_invite/zt-mhs55zh0-XmY3bcfWn4XEyMqUUutbUQ), I'm always happy to help.

## Thanks!..
@nsmith5
@avoidik
@AyushAmbastha
@axelsimon
@rebelopsio
@mc-slava
