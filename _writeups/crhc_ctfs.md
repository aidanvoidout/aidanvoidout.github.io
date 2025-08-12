---
layout: default
title: "CRHC CTF osint challenge"
---
## > CRHC CTF OSINT Challenge
for context i came across this challenge in the scriptCTF discord and saw mandela catalogue and instantly got hooked

![test](/images/crhc_osint_images/chal.webp)

from the image, this 'person' has a social media at username r341_hum4n.

we can look up this username on [Instant Username Lookup](https://instantusername.com/?q=r341_hum4n), and an account can be found on instagram at [@r341_hum4n](https://www.instagram.com/r341_hum4n/)

![instagram](/images/crhc_osint_images/instagram.png)

scanning the qr code yields a link to a [notion blog](https://second-eoraptor-5ec.notion.site/Al3x-s-bl0g-real-human-2301e8cf443b80e5ac61cfe552b53221)

hello!

![hello](/images/crhc_osint_images/hello.png)

look at the first blog, there's a zipfile that requires a password. it says it requires the help of a good friend.

now one good friend of mine is john (the ripper), and we can attempt to crack the zip using a wordlist. but what wordlist?

immediately after we find a wordlist by looking at the second blog post

![blog2](/images/crhc_osint_images/entry2.png)

![wordlist](/images/crhc_osint_images/wordlist.png)

we feed this wordlist to john and crack the password `good friend`

![john](/images/crhc_osint_images/john.png)

unzip

![unzip](/images/crhc_osint_images/unzip.png)

view the file

![ah](/images/crhc_osint_images/open.png)

ah

![spooky](/images/crhc_osint_images/Headshot.jpg)

run some simple stego to find hidden data

![final](/images/crhc_osint_images/result%20and%20falg.png)

flag: `CRHC{y0u_4r3_r3411y_900d_47_051n7!!!}`
