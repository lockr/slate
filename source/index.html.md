---
title: Lockr Docs

language_tabs: # must be one of https://git.io/vQNgJ
  - php
  # - javascript

toc_footers:
  - <a href='https://lockr.io/user/register'>Create a Lockr account</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  # - errors

search: true
---

# Introduction

## Welcome!
Welcome to our documents and support! Even though Lockr is an easy-to-use plugin for WordPress or Drupal, occasionally users run into some challenges and we want to help you through it. Below are some resources to help guide you through the installation, registration, and use of Lockr.

When necessary, you can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right.

These docs are open to public editing, you can find them at [our github page](https://github.com/lockr/slate). Feel free to submit pull requests and help us make the docs better with your experiences.

## Additional Support

If you'd like support outside these docs, we're here to help! You can find us on our [slack channel](https://slack.lockr.io)

<a style="margin: 0 auto; display: block; width: 280px;" href="https://slack.lockr.io"><img src="/images/lockrslack.png"></a>

The easiest way to setup Lockr is with one of our [hosting partners](#lockr-partners)

# Lockr Partners

Lockr partners are hosting providers and website platforms that have integrations with Lockr to make registration and use simple and secure. Our Partners include:

* **[Pantheon](#pantheon)**

<aside class="success">
While Lockr works best when integrated with one of our hosting partners, you can use Lockr anywhere you want.
</aside>

# Lockr General Concepts

Lockr is a secrets management service, which integrates into any application to provide a secure way to store values such as API tokens, passwords, and encryption keys. Currently Lockr provides integrations with [Drupal](#drupal) and [WordPress](#wordpress) which allow for registration, and storage without any extra code necessary.

Below are some general concepts to understand when using Lockr.

## How Lockr protects secrets

Lockr takes protecting secrets very seriously and appreciates the trust our customers put in us to keep them safe. To do this, we make sure each layer of our system is built to meet or exceed industry standards for security and availability.

### Layers of Encryption

Lockr deploys multiple layers of encryption in order to protect secrets. The first and foremost layer of encryption happens before the value even leaves your site. The Lockr modules and plugins encrypt the value to be stored using AES 128 bit encryption with a SHA256 HMAC. If you know what this means, you know that's secure. If you don't to put it in [perspective](https://security.stackexchange.com/questions/6141/amount-of-simple-operations-that-is-safely-out-of-reach-for-all-humanity/6149#6149), it will take 10 years, using all the energy output for the entire earth to compute a single value (assuming technology continues to double till the year 2040).

This process is called key wrapping, as it takes one value, and encrypts (wraps) it using a second key. Lockr takes the second key, usually termed the KEK (key encryption key) and stores it in the website in place of the original value. This KEK does not have any function outside securing the value stored in Lockr so it is safe to store on the site. Anyone gaining access to this key still does not have the original value, and since they are not related or a derivation of the value, it's useless to an attacker.

Lockr then takes the encrypted value to be stored, and sends it over a secure, encrypted connection to the Lockr servers. Once in Lockr, it is routed to and stored encrypted at rest inside a an HSM (Hardware Security Module), provided by [Townsend Security](https://www.townsendsecurity.com/products/centralized-encryption-key-management).

This is what's termed "end to end encryption" as the value that is sent to Lockr is encrypted and only able to be decrypted where it started. Lockr cannot at anytime view, modify, or lose your value.

### Certificates for Authentication

What good is an API token to secure an API token? We asked ourselves the same question when coming up with Lockr, which is why we rely on x509 certificates for authentication. These certificates aren't unlike the ones used to provide SSL on your website, and allow us to cryptographically prove the source of the value. It also allows us to quickly invalidate and re-issue new certificates should one get lost or compromised during an attack.

These certificates are at the heart of all that Lockr does. They provide us a secure way to validate the request, encrypt the transmission of the data to Lockr, and identify the site and environment the request is coming from. Because of this, it's important to keep these certificates in a safe location. If you're on a [partnering host](#lockr-partners) you're taken care of, it's all handled under the covers of the host. If you're not, be sure to [put them into a safe location](#custom-certificate-location).

## Separate Environments

One of the benefits of Lockr, besides storing your values in a secure encrypted HSM, is that values for development and production environments are isolated from each other. This is done for a number of reasons:

1. If you are encrypting values with a key stored in Lockr, those values will not be able to be decrypted in development environments. This is important in regulated environments where you need to be able to control who has access to encrypted data.
2. Admit it, we've all done it. You've accidentally sent an email from a development environment to a production service. Lockr helps keep your development environment isolated by keeping your production tokens and keys away from developers. (If you want horror stories of why this is important just ask us, we've seen some good ones)

This can cause some [confusion](#values-in-different-environments) for first time users, but once synchronized it becomes a background process that keeps data where it should be and developers safe from accidentally triggering production events.

# Download and Installation

## Downloads

### Drupal Module

Drupal is where Lockr got its start, as a simple and secure way to store encryption keys and secrets via the [key modue](https://www.drupal.org/project/key). You can find Lockr for Drupal and download it from our [module page](https://drupal.org/project/lockr).

Below we'll go through how to download, install, and register Lockr on both Drupal 7 and Drupal 8. Once you have the module installed, the UI for registration and key management between the two versions are very similar. Be sure to download the Key module along with Lockr as it's a dependency for it to function.

### WordPress Plugin

You can download Lockr for WordPress from our [plugin page](https://www.wordpress.org/plugins/lockr).

### PHP Library

We have our php library, which is used by both the Drupal and WordPress plugins, available on packagist [here](https://www.packagist.org/packages/lockr/lockr-client)

## Drupal 7 Installation

When installing a module on Drupal 7, you first need to get the module files into the right spot. Once you have the files where you need them, you likely will need to commit them to your version control (likely git) or SFTP them into the server. We **highly** recommend using git as a version control system as it is the most secure way to manage site files. If you do need to upload them directly to the server, be sure to use SFTP and not FTP.

### Download the files

Upload the newest version of the [Lockr](https://www.drupal.org/project/lockr) and [Key](https://www.drupal.org/project/key) modules to the below file path. Once placed in that folder path, login and activate the plugin. Then jump to set up and registration.
`/sites/all/modules/contrib`

### Command Line (Drush)

Using Drush to install Lockr is the easiest way, if you're comfortable in the command line. Once [installed](http://docs.drush.org/en/master/install/) you can simply run
`drush dl lockr key` to download the modules.

### Drupal Interface

This is the least recommended method as it requires your website to have write access to your filesystem which can lead to insecure setups. Most hosts and site platforms will not allow this. If you can't use the above methods, this does provide a way to install the modules while still in the site admin.

First go to the admin page for modules located at `/admin/modules` on your website. There you'll find a link at the top to "+ Install new module" which should lead you to a page that looks like:

![Upload Module in Drupal](/images/Drupal-screenshot-02.png "Install a module through the UI")

Once on this page you can upload the module you downloaded, or you can enter in the URL from the Drupal project page of the version you want to install. Once complete, repeat the steps with the Key module.

### Enable the Module

Once you've got the modules into the site directory, you can then enable them by going to `/admin/modules` and checking the box next to Key and Lockr, then clicking the **Save Configuration** button at the bottom of the screen. The [Ctools](https://www.drupal.org/project/ctools) is required as well but you likely already have this enabled as it's standard requirement to a lot of Drupal modules.

![Enable Lockr module in Drupal 7](/images/Drupal-screenshot-03.png "Install Lockr in Drupal 7")

## Drupal 8 Installation

In Drupal 8, installation is done through [composer](https://www.drupal.org/docs/develop/using-composer/using-composer-to-manage-drupal-site-dependencies) as this is the best practice recommended. To download the module as well as the php library from your command line, inside the site folder run:
`composer require drupal/lockr`

This will download not only Lockr but all the requirements and update the composer.json with these requirements.

Once you've got the module into the site, you can then enable them by going to `/admin/modules` and checking the box next to Key and Lockr, then clicking the **Install** button at the bottom of the screen.

![Enable Lockr module in Drupal 7](/images/Drupal-screenshot-03.png "Install Lockr in Drupal 7")

## WordPress Installation

When installing a plugin on WordPress, like any CMS, there are multiple ways to accomplish it. In essence you will need to [download the plugin](https://www.wordpress.org/plugins/lockr) and make sure it is in the right directory `/wp-contents/plugins`. Once you have the files where you need them, you likely will need to commit them to your version control (likely git) or SFTP them into the server. We **highly** recommend using git as a version control system as it is the most secure way to manage site files. If you do need to upload them directly to the server, be sure to use SFTP and not FTP.

### File Upload

[Download](https://www.wordpress.org/plugins/lockr) the Lockr plugin and upload to the `/wp-contents/plugins` directory of your website. As mentioned above, this can be done either by downloading and checking it into your version control system (git) or via SFTP directly to the site/server. Once placed in that folder path, login to the admin dashboard and activate the plugin by clicking the activate link next to the plugin. (`/wp-admin/plugins.php`). Once activated, jump to [setup and registration](#setup-and-registration).

### WP-CLI

WP-CLI is a command line interface to do many necessary and advanced functions for your WordPress site. More information on it can be found [here](https://make.wordpress.org/cli/) and [here](https://wp-cli.org).

With WP-CLI you simply need to run the following command:
`wp plugin install lockr - activate`

### Terminus

If you are on [Pantheon](#Pantheon) there is a [terminus plugin](https://github.com/lockr/lockr-terminus) available to automate the install process. More information on this can be found on [Pantheon's docs](http://pantheon.io/docs/terminus/) as well as [the Lockr Pantheon docs](https://pantheon.io/docs/guides/lockr/).

### WordPress Dashboard

If your server allows for it, you are able to install Lockr directly from your WP-Admin dashboard (`/wp-admin` on your website). Access the WordPress Plugins in the left admin panel. This will bring you to the list of currently installed plugins used with your site. Now locate the “Add new” button near the top of this new screen. Now search for Lockr in the search plugins data field. Find the Lockr plugin, install and activate it.

![Enable Lockr module in WordPress](/images/wordpress-install.png "Install Lockr in WordPress")

Once you’ve installed and activated the Lockr plugin you’ll notice your WordPress dashboard reloads and there is now a Lockr settings tab located in the left column near the bottom. You’ll be brought to a screen like this below and will need to register your site before using it.

![Setup Lockr module in WordPress](/images/WordPress-03.png "Setup Lockr in WordPress")

# Setup and Registration

Once you have the Lockr plugin or module installed and enabled, you will first need to register the site with the Lockr service before you are able to store any secrets. This process is simple and quick, no matter what environment you're in. During the setup process, we'll only need minimal information about your site to create the server credentials needed to authenticate.

If you're on a [partner](#lockr-partners) the server credentials are already in place, which means we simply need an email to associate with the site. To make setup as simple as possible, if you haven't already registered an account on the [lockr website](https://www.lockr.io/user/register) that's ok, if we detect a new email in the input, we'll create one for you and email information on how to login. If you already created an account and want to use the same email for multiple sites, we'll prompt you for a password to login ito the system and register the site to your account.

If you are not on a partnering server, that's ok too! The registration process will have just one extra step at the beginning to create server credentials for your site. These are how all communication with Lockr is authenticated, and vital to the security of your secrets. Be sure to keep them in a safe location.

## Drupal Registration

Whether you're using Drupal 7 or Drupal 8 the registration process is the same. Follow the steps below to register your site with Lockr to begin storing keys.

### Registration on partnering host

1. Since you've installed the Lockr module on a supporting hosting provider, you’ll notice the certificate is valid, shown by the green row across the top. If that is not the case, you may need a [custom certificate for your installation](#registering-a-custom-certificate-in-drupal). With this, the only input to fill out is your email address.
   ![Register Lockr module in Drupal](/images/Drupal-screenshot-04.png "Register Lockr in Drupal")
2. If you already have an account you’ll be prompted to enter your Lockr password. If not, enter your email address in the previous step and an account will be created for you.
   ![Register Lockr with an existing account in Drupal](/images/Drupal-screenshot-06.png "Register Lockr with an existing account in Drupal")
3. Once you’ve logged in...that’s it! Your site has successfully been added to Lockr. You're now ready to start [saving values in Lockr](#storing-values-in-lockr).
   ![Successful registration of Lockr in Drupal](/images/Drupal-screenshot-07.png "Successful registration of Lockr in Drupal")

### Registering a custom certificate in Drupal

If you are not on a partnering host, you can still use Lockr with one additional step. Since Lockr uses certificates (like SSL certificates) to secure the communications and authenticate to the service, we'll need to create some for you. You'll know you need custom certificates if you see the line in the status table for "valid certificate" in red like this:
![Registration of Lockr in Drupal with Custom Certificate](/images/Drupal-screenshot-09.png "Registration of Lockr in Drupal with Custom Certificate")

<aside class="warning">
Lockr requires the private directory be available and setup in Drupal to create custom certificates. This is a safer place to put the certificates however we **highly** recommend moving the certificates to a directory outside the webroot in a read-only directory to prevent anyone from accessing, reading, or altering the files. Follow the <a href="#advanced-settings">advanced settings</a> for how to do this.
</aside>

1. Near the bottom of the screen, fill out the form, enter Country, State, Locality (or City) and organization name. Lastly, click the "Create Certificate." This will request a certificate from the Lockr servers and place it in the site's private directory. **As mentioned above, if possible, move the certificates to a safe directory outside the webroot**
2. Once the custom certificate is created, the status table should show a valid certificate found. If this is the case, all you need to do now is enter the email address you'd like the site to be registered to in the Lockr system.
3. If you already have an account you’ll be prompted to enter your Lockr password. If not, enter your email address in the previous step and an account will be created for you.

## WordPress registration

Registering your WordPress site can be done by following a few simple steps outlined below. You'll know the site is not yet registered when any of the Lockr admin screens only show prompts to register the site, like this:
![Registration needed in Lockr plugin in WordPress](/images/WordPress-03.png "Registration needed in Lockr plugin in WordPress")

### Registration on a partnering host

1. Since you've installed the Lockr plugin on a supporting hosting provider, you'll notice the certificate is valid in the status table (highlighted in green). If that is not the case you'll need for follow the direction on how to [register a custom certificate in WordPress](#registering-a-custom-certificate-in-wordpress). With this, the only input to fill out is your email address.
   ![Valid certificate for Lockr plugin in WordPress](/images/WordPress-04.png "Valid certificate for Lockr plugin in WordPress")
2. If you already have an account, you'll get a second set of inputs to enter your email address and password for Lockr. If you don't already have an account, in the previous step we created an account for you and will email you the information on how to log into the website.
3. Once you’ve logged in...that’s it! Your site has successfully been added to Lockr. You're now ready to start [saving values in Lockr](#storing-values-in-lockr).
   ![Successful registration of Lockr in WordPress](/images/Drupal-screenshot-05.png "Successful registration of Lockr in WordPress")

### Registering a custom certificate in WordPress

If you are not on a partnering host, you can still use Lockr with one additional step. Since Lockr uses certificates (like SSL certificates) to secure the communications and authenticate to the service, we'll need to create some for you. You'll know you need custom certificates if you see the line in the status table for "valid certificate" in red like this:

![Custom certificate needed in Lockr plugin in WordPress](/images/WordPress-08.png "Custom certificate needed in Lockr plugin in WordPress")

<aside class="warning">
Lockr writes to the WP Content directory when it initially creates the certificates. This is a the only reliable place to put the certificates from a plugin perspetive however we **highly** recommend moving the certificates to a directory outside the webroot in a read-only directory to prevent anyone from accessing, reading, or altering the files. Follow the <a href="#advanced-settings">advanced settings</a> for how to do this.
</aside>

1. The first screen you’ll see is similar to the above but Lockr will automatically detect that you are not on a support hosting provider and will prompt you to enter your country, state or province, city or locality, and company name. Once the information is complete click on "Generate Cert". This will request a certificate from the Lockr servers and place it in the site's private directory. **As mentioned above, if possible, move the certificates to a safe directory outside the webroot**
2. Once the custom certificate is created, the status table should show a valid certificate found. If this is the case, all you need to do now is enter the email address you'd like the site to be registered to in the Lockr system.
3. If you already have an account you’ll be prompted to enter your Lockr password. If not, enter your email address in the previous step and an account will be created for you.

## Advanced Settings

Lockr takes care of most of the setup for you, however there are a few advanced settings that you'll be able to change that will let you modify where Lockr finds the certificate, and what region you want your secrets stored in.

### Custom Certificate Location

If you've followed our recommendation to move your custom certificates outside the webroot into a more protected folder, kudos! You're doing it right. With that, Lockr will need to know where to look for the certs. Simply use this input in the advanced settings to point to the directory the certificate file is located. This directory will need to be readable to the webserver but doesn't need any global permissions. This input can be relative to the site root.

<aside class="success">
For an even more advanced setup, when setting up Lockr in production keep the certs in the same folder location (and name) to allow Lockr to seamlessly migrate from production environments to development environments, switching out the secrets with the ones stored in the development environment.
</aside>

### Lockr Region Selection

By default Lockr selects the US region to store secrets in. You are able to choose another available region to store your secrets in, if you are located outside the US. Because the Lockr regions are isolated, this means values set in one region will not be available in any other region. Be sure to do this as part of your setup process or you will have to re-enter all keys back into Lockr.

## Moving to Production

If you are using a custom Lockr certificate, you will need to move to production. In order to do this, you'll need to have 2 things:

1. A credit card on file with the Lockr website
2. A valid development certificate

When you move to production, Lockr will send a one-time request to the service to generate a production certificate, signing that request with the development certificate for that site. If successful, a production certificate will be returned and put in place of the development certificate. **Protect this certificate as it is what gives any request to our service the ability to return production values**

# Troubleshooting

If you are running into issues with Lockr, that's not fun for anyone. We want to make sure you get the support you need. We're always available at our [slack channel](https://slack.lockr.io) if you want to chat with us or anyone else in the Lockr community.

In supporting customers, here are some common issues we find people running into and how to fix them. Many will rely on information that can be found on the Lockr configuration page from within the site. There we provide a status table to know the live status of your site and Lockr.

## No Valid Certificate

The #1 thing we find people running into is Lockr either not having a valid certificate, or not knowing where to look for one. Without a valid certificate, Lockr plugins cannot do anything and all unsigned requests are immediately rejected by our services.

If you are on a supported hosting provider, and are seeing an invalid certificate on your site then we are having issues with the hosting provider and it needs to be elevated to Lockr and/or your hosting provider asap.

More often, we see this issue when someone is on a custom certificate. It is likely due to the certificate being in a location that Lockr doesn't know about. The [advanced settings](#advanced-settings) allow you to specify this location for the certificates. Once you know the location of the directory, enter it here and you should now be seeing green on the status table for a valid certificate.

## Key value not found

Key values are not returned from Lockr because of one of 2 issues: The certificate was not valid or the value requested doesn't exist in the environment it's being requested. You can reference a valid certificate not found above, but if you're seeing a valid certificate and no key values it's likely because Lockr separates development and production environments. Without setting a value in each environment, Lockr may not find the value you're looking for.

## Values in different Environments

Lockr [separates production and development environments](#separate-environments). This can lead to issues where a key is being requested in an environment it doesn't exist. If you first setup Lockr in a development environment (which we recommend) the values you set will be in our development vaults. Because of this, when you move to production the values will not be present. You will need to re-enter the production value into the admin interface. Be sure to name the key the same you did in testing, so when you pull the production database back to development environments Lockr will seamlessly transition to using the development values instead of production.

<aside>
Currently there are no ways to automatically migrate values from Production down to development, and visa versa. Values will need to be set independently in each environment with the same name to synchronize on migration.
</aside>

# FAQ

  - [What types of secrets does Lockr manage?](#what-types-of-secrets-does-lockr-manage)
  - [Can Lockr be used by a non-CMS application?](#can-lockr-be-used-by-a-non-cms-application)
  - [How is Lockr different from other key management systems?](#how-is-lockr-different-from-other-key-management-systems)
  - [Is Lockr safe?](#is-lockr-safe)
  - [Will developers be able to access the secrets?](#will-developers-be-able-to-access-the-secrets)
  - [Who can access the secrets?](#who-can-access-the-secrets)
  - [Where are the secrets stored?](#where-are-the-secrets-stored)
  - [How can I tell what my key usage is?](#how-can-i-tell-what-my-key-usage-is)
  - [What happens if my site / application is hacked?](#what-happens-if-my-site-application-is-hacked)
  - [What happens if Lockr gets hacked?](#what-happens-if-lockr-gets-hacked)
  - [Can Lockr access my keys?](#can-lockr-access-my-keys)
  - [What if I want to leave Lockr?](#what-if-i-want-to-leave-lockr)

## What types of secrets does Lockr manage?

* API keys for e-commerce services such as: Paypal, Stripe, Square, Authorize.net, FedEx, UPS, etc.
* API keys for email and marketing service providers: MailChimp, Sendgrid, Campaigner, HubSpot, Gmail, Office365 and other SMTP mail servers
* Access tokens for cloud services like Amazon Web Services (AWS), Azure, Google Cloud, SalesForce, Zoho and more.
* Keys used for encrypting data on your website (Don't have an encryption key? Lockr can [create one for you](#create-an-encryption-key))
* LDAP and SSO authentication credentials

## Can Lockr be used by a non-CMS application?

Yes! Non-CMS applications can easily integrate with Lockr using our API. The API supports creating authentication certificates and saving/retrieving key values. We currently offer a [php library on Packagist](https://packagist.org/packages/lockr/lockr-client?). If you have another language you’d like to see a library built in, contact support and we’d be happy to help.

## How is Lockr different from other key management systems?

Lockr makes key management simple, secure, and carefree by taking all the hard work and monitoring on leaving you to focus on your site / application. Lockr sets itself apart by encrypting the keys prior to leaving the site or application. This process, called key wrapping, encrypts the key while it is still in your website / application. Key wrapping prevents keys stored in Lockr from being viewed or compromised outside of your website / application (Lockr cannot see your key values - EVER) In addition to these steps, the wrapped keys are always transferred via secure HTTPS connections and stored in Townsend Security’s FIPS 140-2 compliant key manager, making them secured to the highest of industry standards.

Lockr also introduces the concept of managing keys on a “per environment" basis, which helps eliminate the potential of keys being shared from production to development environments. No longer will you have to worry about sending a test notification from development to production users, or having production data decrypted in development environments.

## Is Lockr safe?

Yes! Lockr can be used to secure any API key, application secret, encryption keys and other types of credentials. Once enabled in the CMS, keys entered into Lockr are wrapped (encrypted) and then sent over an encrypted connection to the Lockr system. Many times the credentials used to access Lockr are provided by the site host or application platform to prevent hijacking and tampering. This prevents unauthorized access and also enables the separation of development and production environments. Using certificates, either by the hosting provider or ones issued by Lockr, allows for secure authentication without passwords or tokens. These certifications expire, can be revoked and are automatically renewed making managing access simple and secure. Using key wrapping (see above), keys are rendered useless from being used outside the website or application environment. Everyone, including Lockr, is unable to get the values of your key.

## Will developers be able to access the secrets?

Only if you want them to. Lockr has isolated development and production environments and working with your hosting provider, or with custom certificates from Lockr, developers can be restricted to only access keys in the development environment.

If you're encrypting sensitive information in your production environment, that data should not be decrypted anywhere but in production. With Lockr, data encrypted in production with a production key is not retrievable outside that environment. This is because Lockr is aware of what environment a request is coming from and uses entirely different key storage environments for development and production keys. When cloned to development, the keys that development environment has access to cannot decrypt the data.

## Who can access the secrets?

The only access to the encrypted keys stored in Lockr is given to a request signed by the certificate used to save that key. Using a proprietary method, Lockr routes and stores keys using a “fingerprint” based on the access certificate which prevents anyone with a valid certificate getting a key belonging to another account.

Since keys are encrypted with AES-256 before they are ever transmitted to Lockr there is no way for Lockr to access key values. We’d have to bend the rules of physics and math in order to do read a key value.

## Where are the secrets stored?

Once transmitted to Lockr, encrypted key values are routed to a highly redundant cluster of cloud Hardware Security Modules (HSMs) for storage based on the environment and region desired for that key. Lockr runs isolated US and EU clouds to ensure storage meets regulatory standards and are positioned as close to your site / app for performance. Lockr is backed by Townsend Security Alliance Key Managers, which are FIPS 104-2 validated and NIST compliant, ensuring the highest level of industry standards.

## How can I tell what my key usage is?

Once transmitted to Lockr, encrypted key values are routed to a highly redundant cluster of cloud Hardware Security Modules (HSMs) for storage based on the environment and region desired for that key. Lockr runs isolated US and EU clouds to ensure storage meets regulatory standards and are positioned as close to your site / app for performance. Lockr is backed by Townsend Security Alliance Key Managers, which are FIPS 104-2 validated and NIST compliant, ensuring the highest level of industry standards.

## What happens if my site / application is hacked?

If your site is hacked, usually the first thing a hacker does is make a copy of the environment they’re in. This means they grab a copy of your database if it is a SQL exploit or a copy of the server if it’s a server breach. Each of these environments do not alone have the ability to expose a key stored in Lockr. The database stores the secondary key, used in the key wrapping process, but by itself this key is useless. Likewise the server environment holds the connection certificate, but without the ability to unwrap (decrypt) the value given back from Lockr the value is rendered useless. Additionally, certificates can be revoked and re-issued. By revoking a certificate, a hacker will no longer be able to access the encrypted values stored in Lockr.

By isolating the various pieces necessary to get the true key value, it greatly reduces the chances a hacker will be able to access your keys.

## What happens if Lockr gets hacked?

In the unfortunate situation where Lockr itself is hacked, we’ve taken the necessary steps to isolate systems to keep your keys safe. By going through the key wrapping within your site or application, by the time Lockr has the key it is not longer in a usable form. This means that even if a hacker gained access to Lockr’s HSMs they would be unable to get access to the true keys stored within. Additionally, by using our proprietary fingerprinting methodology even the name of a key is not decipherable.

Access to the Lockr database is worthless as all values within it are enrypted with keys we don’t have access to, or are obfuscated using data we don’t store.

## Can Lockr access my keys?

Since keys are encrypted with AES-256 before they are ever transmitted to Lockr there is no way for Lockr to access key values. We’d have to bend the rules of physics and math in order to do read a key value.

## What if I want to leave Lockr?

We hate to see you go! However, if you need to leave Lockr, we can either migrate you to another Townsend Alliance Key Manager or provide you with a way to export your keys and unwrap them in your application. Contact support for help in this process.
