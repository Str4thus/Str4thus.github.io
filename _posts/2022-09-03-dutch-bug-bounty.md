---
layout: post
title: "Dutch Bug Bounty: A Local File Inclusion"
date: 2022-09-02 23:45:13 -0400
background: '/img/posts/02.jpg'
---

Through an unnamed parameter that gets passed as a filename to a shell script on the serverside, arbitrary files could be read from the remote system. However, the filename had to start with `data/`, as only files from this folder should be included. This restriction was bypassed by using a simple Path Traversal payload, resulting in:

```sh
climexp.knmi.nl/select.cgi?data/../../../../../etc/hostname
```

---

## Prologue
The Dutch government has a public [reponsible vulnerability disclosure program](https://english.ncsc.nl/contact/reporting-a-vulnerability-cvd), where defects within the vital infrastructure of the Dutchs can be securely submitted. Additionally, they provide a excel spread sheet that lists all assets that are in scope [here](https://www.communicatierijk.nl/vakkennis/r/rijkswebsites/verplichte-richtlijnen/websiteregister-rijksoverheid).

As a reward for submitting a valid report, the NCSC offers a shirt which says `I hacked the Dutch government and all I got was this lousy t-shirt`, which basically got me all in.

## Finding a target
I spent a few days looking for a suitable target and manually skimmed though the assets listed in the mentioned excel spread sheet. I noticed, that a lot of the sites utilize a CMS, so I slapped together a quick scan script that would visit every website and look for a `Generator` meta tag that reveals the CMS in use (and sometimes even the exact version). There was a lot of `WordPress` and `Drupal` used, but all versions seemed up to date or at least properly hardened so that no quick wins like `Drupalgeddon` were viable here.

I continued to manually look for some fragile web apps until I finally stumbled upon [climexp.knmi.nl](http://climexp.knmi.nl). This site can be used to investigate the climate by plotting all kinds of different graphs and letting it perform calculations and stuff.

## The Vulnerability
The website allows users to create their own `form fields` that then can be shared with other users. Upon inspecting the `form field`, I quickly noticed that there is a local path within the URL that seems to contain data related to the field I just created.

![Normal Behaviour](/img/posts/path_url.png)
*Normal Behaviour*

The next thing I tried was to simply change the parameter to a file most definitely exists on any linux box: `/etc/hostname`. It is important to not try files like `/etc/passwd` or even `/etc/shadow` here as they would **give up sensitive information about the system and thus be out of scope**.

![Trying a simple file inclusion](/img/posts/Pasted%20image%2020220904011745.png)
*Trying to include /etc/hostname*

Now the site yields an error, stating that it `cannot handle /etc/hostname (yet)`. So, what about a file that is almost certain to not exist on the remote target?

![Non-existing file](/img/posts/Pasted%20image%2020220904012019.png)
*Trying to include a non-existing file*

This time, the site gives up more information. It seems like, that the parameter gets passed into a shell script called `describefield.sh`. Here, I tried to inject a command into the filename, but as only valid filenames seem to get passed along onto the command line, I had no success with that. I kept on playing with the parameter and figured, that the filename has to start with `data/`, as otherwise the `cannot handle` error appeared.

But luckily, to escape the `data/` directory restriction a simple `Path Traversal` payload was enough! The final payload was:

```py
http://climexp.knmi.nl/select.cgi?data/../../../../../../../../../etc/hostname
```

![Final Payload](/img/posts/dutchbb_lfi.png)

>From here on, an attacker might have gained `remote code execution` on the remote system by exploiting `Log Poisioning` techniques. 

## Suggested Mitigations
Within my report, I suggested the following two mitigations:

1. Proper sanitazation of the parameter (filtering out `../` to prevent path traversal in the upward direction)
2. Passing the parameter through a `basename` function, so that `data/../../../../../../../../../etc/hostname` would get turned into `hostname`
 
## Disclosure Timeline
- 26.08.2022 - Reported the vulnerability to NCSC
- 26.08.2022 - NCSC confirmed the vulnerability and informed the responsible organisation
- 30.08.2022 - NCSC stated, the vulnerability has been fixed and asked for confirmation (was fixed)
- 03.09.2022 - I received the Dutch bug bounty shirt
