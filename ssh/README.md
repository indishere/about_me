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
| `main-pc.pub` | `main-pc@hackthe.world` | main-pc's user key |
| `linux-server.pub` | `linux-server-deploy` | read-only deploy key for pulling `hackthe-world/devices` |
| `gateway.pub` | `gateway-deploy` | same idea, on the gateway |
| `winvm.pub` | `winvm` | the Windows guest's user key |

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
printf '%s %s\n' <hostname> "$(cat hosts/linux-server.pub)" >> ~/.ssh/known_hosts
```

Only the ed25519 host key is published. Every one of these hosts also has RSA
(and the Windows ones ECDSA) host keys, but modern OpenSSH negotiates ed25519,
and pinning one good key beats maintaining three.

## Where these live on the machines

| host | host keys | user keys |
|---|---|---|
| `gateway`, `linux-server` | `/etc/ssh/` | `~/.ssh/` |
| `main-pc`, `winvm` | `C:\ProgramData\ssh\` | `~/.ssh/` |

On Windows the host keys **must stay** in `C:\ProgramData\ssh\`: sshd runs as a
service and reads them from there, under ACLs that only SYSTEM and
Administrators can read. Only user material — `id_ed25519`, `config`,
`known_hosts`, `authorized_keys` — belongs in `~/.ssh` on those boxes.

Two consequences worth knowing:

- `main-pc.pub` was read from its running sshd with `ssh-keyscan`, not from
  disk, because that ACL blocks even reading the file as the logged-in user.
  `ssh-keyscan` does not carry the comment field, so that key has no trailing
  comment. It is the same key.
- `winvm` is a rebuildable VM. Reinstalling it generates **new** host keys, so
  `hosts/winvm.pub` has to be refreshed whenever that happens — and old
  `known_hosts` entries for it will correctly start complaining.

`linux-pc` and `phone` are in the device inventory but not built yet, so they
have no keys to publish.
