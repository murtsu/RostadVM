# Security Policy

## Supported Versions

RostadVM is a solo project under active development.

| Version | Supported |
|---------|-----------|
| main    | Yes       |
| all others | No   |

## Reporting a Vulnerability

This is a single-maintainer project. There is no security team. There is me.

If you find a vulnerability, email me directly. Do not open a public GitHub issue for security matters. Do not post it on social media. Do not sit on it.

Contact: [murtsu@gmail.com]

I will respond within 7 days. If I have not responded in 7 days, send it again.

## Scope

RostadVM manages virtual machines via Libvirt. The attack surface that matters:

- Privilege escalation via VM restore operations
- Unauthorized access to the copy-on-write snapshot mechanism
- Libvirt socket exposure
- Unsafe handling of VM configuration files

Out of scope: anything running inside the guest VM. That is your problem.

## Disclosure Policy

I will fix it. I will credit you if you want credit. I will not negotiate timelines with strangers on the internet.

Coordinated disclosure window is 90 days from first contact. After 90 days you are free to publish.

## What I Will Not Do

I will not accept pull requests that modify security-critical paths without prior discussion.

I will not merge code I have not read.

I will not add dependencies I do not understand.

## Access

One maintainer. One set of keys. That is the policy.
