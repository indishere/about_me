# SSH public keys

**Public key material only.** No private key belongs in this repository — not
encrypted, not "temporarily", not on a branch. A private key lives in exactly
one place: `~/.ssh/` on the machine it belongs to (plus `C:\ProgramData\ssh\` for
a Windows *host* key, see below).

The two directories answer two different questions, and conflating them is how
you end up trusting the wrong thing:

```
identities/   "this public key proves device X is CONNECTING"
hosts/        "this public key proves the server I reached really IS X"
```

One `.pub` per name, ed25519 throughout.

## identities/

The keys that go into an `authorized_keys` file, or into GitHub, to let that
device in.

| file | key comment | what it is |
|---|---|---|
| `main-pc.pub` | `main-pc@hackthe.world` | main-pc's login key |
| `server.pub` | `server@hackthe.world` | server's login key (user `hacktheworld`) |
| `gateway.pub` | `gateway@hackthe.world` | gateway's login key (user `root`) |
| `win-vm.pub` | `win-vm@hackthe.world` | the Windows guest's login key |
| `phone.pub` | `phone@hackthe.world` | IND's phone |
| `phone-biometric.pub` | `phone-biometric@hackthe.world` | the same phone, biometric-gated (ECDSA) |
| `server-deploy.pub` | `linux-server-deploy` | read-only deploy key for pulling `hackthe-world/devices` |
| `gateway-deploy.pub` | `gateway-deploy` | same idea, on the gateway |

**Login keys and deploy keys are separate on purpose.** The `*-deploy.pub` pair
exist to pull one private repo read-only and nothing else. Reusing a repo-scoped
credential as a general login credential turns a narrow key into a broad one for
no gain, so `server` and `gateway` each got a second, dedicated login key rather
than promoting the deploy key they already had.

Every key in the first six rows is authorised on **every** host in the fleet —
main-pc, server, win-vm and gateway — so any machine reaches any other. The
grants live in `nixos/*/configuration.nix` (`meshKeys`) for the Linux boxes,
`~/.ssh/authorized_keys` on main-pc, and
`C:\ProgramData\ssh\administrators_authorized_keys` on win-vm.

### `router.pub` is not one of IND's devices

It is the gateway's own identity for the second hop of
`ssh <device>@hackthe.world`. The private half lives at
`gateway:/etc/ssh/router_ed25519`, mode `0640 root:ssh-router`, readable by the
four router accounts — so it is the one key here whose private half is readable
by something other than a single administrator.

That is exactly why it is separate from `gateway.pub` rather than reusing it.
The two grants differ deliberately:

| | authorised on |
|---|---|
| `gateway.pub` | root on main-pc, server, win-vm |
| `router.pub` | **`hacktheworld` on server (never root)**, plus both Windows boxes |

It is also **not** authorised for root on the gateway itself. The gateway is
where it is stored, so a loop back to root there would erase the whole point of
it being the weaker key. Revoking it stops routing and leaves admin access
alone, which is the property worth having.

If the gateway is ever reinstalled, `router.nix` generates a *new* key and every
route fails at once with `Permission denied (publickey)`. Replace this file with
the new public half and re-grant it on the three targets.

## hosts/

The keys that answer "is this the real box?" when ssh shows an unknown-host
prompt. Check what the server offers against the file here:

```sh
ssh-keyscan -t ed25519 <host> | ssh-keygen -lf -   # what the server claims
ssh-keygen -lf hosts/<host>.pub                    # what it should be
```

If those two fingerprints differ, **do not type yes.**

There are no fingerprint files in this repo on purpose. A fingerprint is just a
hash of the public key, so committing both would be paperwork for the paperwork —
derive it with `ssh-keygen -lf` whenever you want it.

To trust a host up front instead of being prompted, append it to known_hosts
under the name you actually connect to:

```sh
printf '%s %s\n' <hostname> "$(cat hosts/server.pub)" >> ~/.ssh/known_hosts
```

Only the ed25519 host key is published. Every one of these hosts also has RSA
(and the Windows ones ECDSA) host keys, but modern OpenSSH negotiates ed25519,
and pinning one good key beats maintaining three.

## Where these live on the machines

| host | host keys | user keys |
|---|---|---|
| `gateway`, `server` | `/etc/ssh/` | `~/.ssh/` |
| `main-pc`, `win-vm` | `C:\ProgramData\ssh\` | `~/.ssh/` |

On Windows the host keys **must stay** in `C:\ProgramData\ssh\`: sshd runs as a
service and reads them from there, under ACLs that only SYSTEM and
Administrators can read. Only user material — `id_ed25519`, `config`,
`known_hosts`, `authorized_keys` — belongs in `~/.ssh` on those boxes.

Two consequences worth knowing:

- `main-pc.pub` was read from its running sshd with `ssh-keyscan`, not from
  disk, because that ACL blocks even reading the file as the logged-in user.
  `ssh-keyscan` does not carry the comment field, so that key has no trailing
  comment. It is the same key.
- `win-vm` is a rebuildable VM. Reinstalling it generates **new** host keys, so
  `hosts/win-vm.pub` has to be refreshed whenever that happens — and old
  `known_hosts` entries for it will correctly start complaining.

`moms-laptop` and `phone` are in the device inventory but not built yet, so they
have no keys to publish.

## Filenames are the device name; embedded comments may lag

The filename is authoritative — it is the device's current name. The trailing
comment *inside* a `.pub` is whatever `ssh-keygen` stamped when the key was
generated, so after a device rename it still reads the old one:
`hosts/server.pub` says `root@linux-server`, and `hosts/win-vm.pub` says
`system@winvm`. The *identity* keys no longer lag: win-vm's was regenerated on
2026-08-18 and carries the current name.

That is cosmetic and deliberately left alone. The comment is not part of the
key, nothing verifies against it, and rewriting a live host key file to correct
a string is a good way to leave a headless box with an sshd that will not start.
It corrects itself whenever a key is legitimately regenerated.
