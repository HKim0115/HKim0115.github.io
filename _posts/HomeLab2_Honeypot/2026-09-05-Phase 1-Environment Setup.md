---
categories: [Honey Pot]
title: "Honeypot Phase 1. Environment Setup"
---

This phase kept the real server and the fake one (honeypot) completely separate — admin access stays locked down, while attackers get funneled into the honeypot without realizing it. Next up: adding a web honeypot and wiring both honeypots' logs into Splunk for real-time analysis.


- **Server**: AWS Lightsail (Ubuntu 24.04)
  Cheap and simple way to spin up a public-facing VPS

- **Honeypot**: Cowrie
  Open-source SSH honeypot with solid customization and logging support

- **Admin port**: 2200, key-only auth
  Freed up port 22 for the honeypot

- **Public port**: 22 → redirected internally to 2222 (Cowrie)
  So attackers land on the fake server, not the real one

- **SSH hardening**: new key pair, password login disabled, root login disabled
  Minimize the attack surface on the admin account itself

![Cowrie local test log](/assets/images/Honeypot/textlog.png)
*Local test confirming every command typed gets logged*



To summarize: the real admin SSH was moved to port 2200 and locked down with key-only auth, while all traffic hitting port 22 gets redirected to Cowrie (2222) so attack attempts are captured safely without ever touching the real server.

