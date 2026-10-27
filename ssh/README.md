# SSH 

- The `hosts` are the SSH keys of the SSH servers, therefore allowing you to assure the server/device is from my fleet.
- The `identities` are the SSH Keys of me and my devices, which allow whitelisting me and my devices to access my fleet and other external servers using SSH.
## Executioner

`identities/executioner.pub` is the dedicated Executioner public key supplied
by the owner on 2026-10-27. Fingerprint:
`SHA256:BE+4SHTXH09SJUTD773JZaMGcqYYYj40VMoKD/CDIx4`.

Rotation is pending the matching private key being saved through the encrypted
SSH connection form and verified against command-center, gateway and director.
One saved identity authenticates SSH and SFTP on all three hosts. No private
key is published here. The previously active shared identity remains recorded
in `identities/command-center.pub`; do not revoke it as part of this rotation
because it is also used by command-center.

The different former `identities/inds-space-context.pub` was removed in commit
`d824dde` on 2026-10-15. Server host keys in `hosts/` authenticate the devices
and are separate from login identities.
