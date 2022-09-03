---
layout: post
title: "Dutch Bug Bounty: A Local File Inclusion"
date: 2022-08-03 23:45:13 -0400
background: '/img/posts/02.jpg'
---

## TL;DR
Through an unnamed parameter that gets passed as a filename to a shell script on the serverside, arbitrary files could be read from the remote system. However, the filename had to start with `data/`, as only files from this folder should be included. This restriction was bypassed by using a simple Path Traversal payload, resulting in:

```bash
climexp.knmi.nl/select.cgi?data/../../../../../etc/hostname
```

---

## Prologue
The Dutch government has a public [reponsible vulnerability disclosure program](https://english.ncsc.nl/contact/reporting-a-vulnerability-cvd), where defects within the vital infrastructure of the Dutchs can be securely submitted. Additionally, they provide a excel spread sheet that lists all assets that are in scope [here](https://www.communicatierijk.nl/vakkennis/r/rijkswebsites/verplichte-richtlijnen/websiteregister-rijksoverheid).

## Finding a target
I spent a few days looking for a suitable target and manually skimmed though the assets listed in the mentioned excel spread sheet. I noticed, that a lot of the sites utilize a CMS, so I slapped together a quick scan script that would visit every website and look for a `Generator` meta tag that reveals the CMS in use (and sometimes even the exact version). There was a lot of `WordPress` and `Drupal` used, but all versions seemed up to date or at least properly hardened so that no quick wins like `Drupalgeddon` were viable here.

I continued to manually look for some fragile web apps until I finally stumbled upon [climexp.knmi.nl](http://climexp.knmi.nl). This site can be used to investigate the climate by plotting all kinds of different graphs and letting it perform calculations and stuff.

## The Vulnerability
The website allows users to create their own `form fields` that then can be shared with other users. Upon inspecting the `form field`, I quickly noticed that there is a local path within the URL that seems to contain data related to the field I just created.

![Normal behaviour](../img/posts/Pasted%20image%2020220904002852.png)

```python
print("hi")
```


## Suggested Mitigations

## Epilogue
Visit [here](https://www.government.nl/topics/cybercrime/fighting-cybercrime-in-the-netherlands/responsible-disclosure)

## Disclosure Timeline