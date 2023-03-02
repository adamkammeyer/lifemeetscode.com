---
title: "Fedora 26 Upgrade"
date: 2017-07-24T15:59:54-06:00
draft: false
type: "post"
tags:
  - linux
  - fedora
aliases:
  - /blog/fedora-26-upgrade
---

Fedora 26 was released on July 11, 2017. Being a Fedora user, I wanted to upgrade immediately. :) Since I didn't have any issues during the 24 to 25 upgrade, I had confidence this would be as sommoth. I had just reinstalled Fedora 25 a week before 26 was launched so my system was relatively clean. I followed the [blog post on upgrading through the Software Center](https://fedoramagazine.org/upgrading-fedora-25-fedora-26/) and wanted to share my experience.

I pulled up the Software Center, but did not see the upgrade notification. Once I refreshed the Software Center's data it showed up. The download was slow, but that's to be expected on release day. ;)

## The good

Overall, my Initial impressions were good. I was still able to get into every native Linux app I use.

## The "bad"

I use quotes for bad because the issues I ran into were minor in my opinion.

### Comic Collector

I've used [Comic Collector from Collectorz.com](https://www.collectorz.com/comic) for years to manage my comic book collection. Unfortunately there's no native Linux version. I've been able to keep it working under Wine. The trick is that 32-bit emulation works best since the program relies on Internet Explorer being available to properly render the preview layout and 32-bit Internet Explorer 8 is pretty much the only version you can get running under Wine.

I ended up reinstalling Comic Collector under 64-bit Wine. At least I can still manage my collection until I figure out a better way. A virtual machine may be an option, but I've never liked having to load all of Windows just to launch a single progeram.

### Disabled repos

#### PHP

After upgrading, I did a `dnf update` to verify everything was up-to-date and noticed that the `dropbox` and `remi-php71` repos were disabled. First I tackled the `remi-php71` repo by looking at the available disabled repos.

```
sudo dnf repolist disabled | grep remi

remi                                     Remi's RPM repository - Fedora 26 - x86
remi-debuginfo                           Remi's RPM repository for Fedora 26 - x
remi-php70-debuginfo                     Remi's PHP 7.0 RPM repository for Fedor
remi-php70-test-debuginfo                Remi's PHP 7.0 test RPM repository for 
remi-php71                               Remi's RPM repository - PHP 7.1 - Fedor
remi-php71-debuginfo                     Remi's PHP 7.1 RPM repository for Fedor
remi-php71-test                          Remi's RPM repository - Testing - PHP 7
remi-php71-test-debuginfo                Remi's PHP 7.1 test RPM repository for 
remi-php72                               Remi's RPM repository - PHP 7.2 - Fedor
remi-php72-debuginfo                     Remi's PHP 7.2 RPM repository for Fedor
remi-php72-test                          Remi's RPM repository - Testing - PHP 7
remi-php72-test-debuginfo                Remi's PHP 7.2 test RPM repository for 
remi-test                                Remi's RPM repository - Testing - Fedor
remi-test-debuginfo                      Remi's test RPM repository for Fedora 2
```

It showed I was indeed using the Fedora 26 repo (Pretty sure that updated itself, unless I was accidentally using the Fedora 26 repo on Fedora 25). I tried just cleaning out the cache with `dnf clean all`, but that did nothing (other than clean out my cache).

The answer eventually came from the [Remi Repo Config wizard](https://rpms.remirepo.net/wizard/). I just had to re-enable the main repo:

```
dnf config-manager --set-enabled remi
```

Once enabled, there were (luckily) some updates pending for PHP 7.1 so I could verify it was pulling updates correctly. However, that led to another problem. `dnf` was telling me that the public GPG key was not installed.

```
Public key for libzip-1.2.0-1.fc26.remi.x86_64.rpm is not installedFailing package is: libzip-1.2.0-1.fc26.remi.x86_64
 GPG Keys are configured as: file:///etc/pki/rpm-gpg/RPM-GPG-KEY-remi
```

I had to look at the installed GPG keys:

```
sudo rpm -q gpg-pubkey --qf "%{summary} ->%{version}-%{release}\n"
```

The output showed that I had only one of Remi's GPG keys installed. Per [step 4 of the repo config](https://blog.remirepo.net/pages/Config-en), I verified that the signature of the key I had installed matched the one of the fingerprints in the instrutions. I downloaded the 2017 GPG Key and imported it with:

```
rpm --import RPM-GPG-KEY-remi2017
```

After that, I was able to successfully dnf update.

#### Dropbox

Having gone through all that with the Remi repo, I new the first step for enabling the Dropbox repo was

```
sudo dnf config-manager --set-enabled Dropbox
```

I did a quick `dnf update `to verify and I no longer received the warning about the Dropbox repo being disabled. However, after a restart, I continued to receive notice about the Dropbox repo being disabled. I did a little more digging by examing the `/etc/yum.repos.d/dropbox.repo` file. After seeing the URL fo the repo, I determined that there was no Fedora 26-specific endpoint for Dropbox. I updated `dropbox.repo` to point specifically to the Fedora 25 endpoint for now to ensure I get any updates Dropbox pushes out. My `/etc/yum.repos.d/dropbox.repo` file now looks like

```
[Dropbox]
name=Dropbox Repository
#baseurl=http://linux.dropbox.com/fedora/$releasever/
baseurl=http://linux.dropbox.com/fedora/25/
gpgkey=https://linux.dropbox.com/fedora/rpm-public-key.asc
enabled = 1
```
