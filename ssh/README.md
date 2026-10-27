# SSH 

- The `hosts` are the SSH keys of the SSH servers, therefore allowing you to assure the server/device is from my fleet.
- The `identities` are the SSH Keys of me and my devices, which allow whitelisting me and my devices to access my fleet and other external servers using SSH.

## Executioner

`identities/executioner.pub` is the public half derived from Executioner's active
SSH connection credential on 2026-10-27. The same encrypted private key is used
for SSH commands and SFTP on command-center, gateway and director.

Fingerprint: `SHA256:ElxtaH7xswr0Qi2AQYsP+64ciYC0Wl51BhbvRr5mH5k`.

This currently matches `identities/command-center.pub`: it is a shared identity,
not a separately generated key. Rotating or revoking that identity affects both
uses. No private key is published here. A dedicated Executioner identity would
require a separate key and explicit authorization on the fleet before switching.

The former `identities/inds-space-context.pub` was removed in commit `d824dde`
on 2026-10-15 and held a different key; it is not the currently accepted identity.
Server host keys under `hosts/` authenticate devices and are separate from these
login identities.
