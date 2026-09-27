# Networking Commands

1. **`ping google.com`** – Checks connectivity to a remote server.
2. **`ifconfig`** – Displays network interfaces (deprecated, use `ip`).
3. **`ip a`** – Shows IP addresses of network interfaces.
4. **`netstat -tulnp`** – Displays open network connections.
5. **`curl https://example.com`** – Fetches a webpage's content.
6. **`wget https://example.com/file.zip`** – Downloads a file from the internet.

---
 
## Networking commands (fetching content)
 
### `curl https://example.com` — fetch a webpage's content
 
**Real output:**
 
```text
$ curl https://example.com
<!doctype html>
<html>
<head><title>Example Domain</title></head>
<body>
<h1>Example Domain</h1>
<p>This domain is for use in illustrative examples in documents.</p>
</body>
</html>
```
 
**What's going on here:** `curl` reaches out to the given address and prints back whatever it receives — by default, straight to your terminal screen rather than saving it anywhere. Here it fetched the actual HTML source code of that webpage. `curl` isn't limited to just viewing pages — it's often used to test APIs, check whether a server responds at all, or inspect the raw response (headers, status codes) behind the scenes of a web request.
 
**DevOps read**: a quick way to check "is this server actually responding, and with what?" without needing a full browser — commonly used to test that a service is up or to see exactly what an API endpoint returns.
 
**Flags you'll actually reach for most:**
 
- **`-I`** — show just the response headers (status code, content type, etc), without pulling the whole page. Great for a fast "is it up?" check.
```text
  $ curl -I http://localhost:8080/health
  HTTP/1.1 200 OK
  Content-Type: application/json
  Content-Length: 15
  Date: Sun, 27 Sep 2026 10:30:00 GMT
```
  The `200 OK` alone already tells you the service is up and healthy — no need to read a whole page just to confirm that.
 
- **`-X POST`** (or `PUT`, `DELETE`...) — send a request using a different method instead of the default (`GET`, "just give me the data"). Combined with **`-d`** to send data along with it — needed anytime you're testing an API that creates or changes something.
```text
  $ curl -X POST -d 'name=Alex' https://api.example.com/users
  {"id": 42, "name": "Alex", "created": true}
```
  Here the API created a new user and sent back confirmation as its response, which curl prints straight to your screen.
 
- **`-H 'Header: value'`** — add a custom header, most commonly `-H 'Authorization: Bearer <token>'` for hitting an API that needs authentication.
```text
  $ curl -H 'Authorization: Bearer abc123' https://api.example.com/profile
  {"user": "alex", "plan": "pro"}
```
  Without that header, this same request would typically come back with something like `{"error": "unauthorized"}` instead.
 
- **`-o filename`** — save the output to a file instead of printing it to your screen. No visible output in the terminal itself — that's the point, it goes straight to the file instead.
- **`-s`** — "silent": hide the extra progress/status noise curl normally prints, leaving just the actual response — handy inside scripts where you don't want that clutter mixed in with the real output.
### `wget https://example.com/file.zip` — download a file from the internet
 
**Real output:**
 
```text
$ wget https://example.com/file.zip
--2026-09-27 10:20:01--  https://example.com/file.zip
Resolving example.com (example.com)... 93.184.216.34
Connecting to example.com (example.com)|93.184.216.34|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5242880 (5.0M) [application/zip]
Saving to: 'file.zip'
 
file.zip            100%[===================>]   5.00M  2.14MB/s    in 2.3s
 
2026-09-27 10:20:04 (2.14 MB/s) - 'file.zip' saved [5242880/5242880]
```
 
**What's going on here:** where `curl` defaults to printing content to your screen, `wget` defaults to the opposite — saving what it fetches directly to a file on disk, which is why it's the more common choice specifically for downloading something (like a file or a whole website). The output shows it resolving the domain name to an IP address, connecting, confirming the server said "200 OK" (success), then a live progress bar as the file downloads, ending with confirmation of the saved filename and size.
 
**DevOps read**: your go-to when the goal is literally "get this file onto the machine" — for example, downloading an installer, a release archive, or a script directly onto a server. `curl` can also save to a file (with `-o`/`-O`), and `wget` can also print to your screen (with `-O -`), so the two overlap quite a bit in practice — but the defaults above reflect what each tool is typically reached for: `curl` for inspecting/testing responses, `wget` for straightforward downloading.
 
**Flags you'll actually reach for most:**
 
- **`-O filename`** — save the download under a specific name of your choosing, instead of whatever name the URL happens to end in.
```text
  $ wget -O app.zip https://example.com/download?id=8821
  Saving to: 'app.zip'
```
  Without this, wget would have tried to save it as something ugly like `download?id=8821` instead — this gives it a clean, predictable name.
 
- **`-c`** — "continue": resume a download that got interrupted partway through, instead of starting over from zero. Very useful for large files over a shaky connection.
```text
  $ wget -c https://example.com/file.zip
  Saving to: 'file.zip'
  file.zip            34%[========>            ]   1.70M  2.10MB/s
```
  Notice it picks up already at 34% — it detected a partial `file.zip` already on disk from before and continued from there instead of restarting.
 
- **`-q`** — "quiet": suppress the progress output, leaving no clutter — handy inside scripts, same idea as curl's `-s`. No visible output in the terminal at all — that's the intended effect.
- **`-r`** — "recursive": download an entire folder structure of files/pages, not just one file — used for mirroring a whole directory or simple website.
```text
  $ wget -r https://example.com/docs/
  Saving to: 'example.com/docs/index.html'
  Saving to: 'example.com/docs/guide.html'
  Saving to: 'example.com/docs/images/logo.png'
```
  One command, but it pulled down every linked file it found under that folder, not just the single page you pointed it at.
 
