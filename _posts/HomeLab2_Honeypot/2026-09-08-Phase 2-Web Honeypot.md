---
categories: [Honey Pot]
title: "Phase 2. Setting up a Web Honeypot"

---


Catching WordPress Attackers with a Web Honeypot


## Goal

After using Cowrie to collect SSH attacks, I wanted to catch web traffic too, so I built a fake website. The idea is simple. I put up a real server and watch what random bots on the internet actually look for and try.

## Why a Web Honeypot

Port 80 (the web) gets scanned just as much as port 22 (SSH), maybe even more. WordPress is a huge target because it powers so much of the internet. If a bot pokes at a random server, there is a good chance it is running WordPress, so automated attacks aim at it first.

There have been some big WordPress problems recently too. In July 2026, a bug called "wp2shell" (CVE-2026-63030, CVE-2026-60137) was found in WordPress core itself. It let attackers run code on the server without even logging in. Since WordPress core runs on more than 500 million sites, mass scanning started as soon as the exploit went public. In early September, a SQL injection bug (CVE-2026-19949) was found in a backup plugin called All-in-One WP Migration and Backup, installed on 5 million sites, that let attackers mess with the database without logging in.

Whenever news like this breaks, bots immediately start scanning the whole internet for servers that have the bug. But even without breaking news, there are attack patterns that show up all the time:

- `/wp-login.php`, `/wp-admin`: brute forcing the login page with random usernames and passwords
- `/xmlrpc.php`: using WordPress's XML-RPC feature to get around login limits, or to attack other servers through it
- `/.env`, `/.git/config`: trying to steal database passwords or API keys from config files that got exposed by accident
- `/admin`, `/phpmyadmin`: checking if there is an admin panel at all, even on sites that are not WordPress

This list shows that attackers, or really their bots, are not aiming precisely at one site. They just walk through these paths automatically on every server they find. So instead of building a perfect WordPress clone, I decided a generic bait that catches all of these paths at once would be closer to what actually happens on the internet.

## Summary

I built a web app in Flask and put it on the VPS on port 80. It sends back fake pages for the common vulnerable paths listed above and logs every request. Just like Cowrie, it runs under its own separate user account and is registered as a systemd service, so it comes back up automatically if the server reboots.

## What I Built It With

- **Flask** (Python): the web server that serves the fake pages
- **gunicorn**: the WSGI server that runs the Flask app in production
- **systemd**: registers the service and restarts it automatically
- **iptables (NAT REDIRECT)**: quietly forwards requests coming in on port 80 to the internal port 8080, where gunicorn is actually listening

## How It Works

Here is the basic flow:

1. When someone connects to the server on port 80, iptables redirects that traffic to the internal port where gunicorn is running.
2. The Flask app looks at the request path and sends back a matching fake page. `/wp-login.php` shows something that looks like a WordPress login screen, `/.env` shows a fake but believable environment file, and anything not on the list just gets a 404.
3. If someone types a username and password into the login form and submits it, that value gets written to the log exactly as entered, and then they see a fake "wrong password" message. It is neither a full success nor a hard failure, so they keep trying.
4. Every request gets logged as JSON, including the visitor's IP, the time, the path they hit, their browser info, and anything they submitted. This is the same format as Cowrie's logs, so later I can send both to Splunk and analyze them together.


## Results

![the fake WordPress login page](/assets/images/Honeypot/webpage.png)

This is what shows up in a real browser at `http://<server IP>/wp-login.php`. From the outside, it looks like an ordinary WordPress admin login screen.

![a login attempt in the logs](/assets/images/Honeypot/weblog.png)

This is what gets logged after I typed a test username and password into that page and submitted it. You can see the username and password I entered, along with the visitor's IP, browser info, and the exact time. If a real attacker tried this, the same level of detail would show up.

## What's Next

Now that both Cowrie and the web honeypot are running and logging, the next step is pulling both sets of logs into Splunk to build a dashboard that shows incoming attacks in real time.
