# SSH public keys

Everything here is **public key material only**. No private key belongs in this
repository, ever — not encrypted, not "temporarily", not in a branch. A private
key lives in exactly one place: `~/.ssh/` on the machine it belongs to (and
`C:\ProgramData\ssh\` for a Windows host key, see below).

The two directories answer two different questions, and mixing them up is how
people end up trusting the wrong thing:

```
identities/   "this public key proves Mit, or device X, is CONNECTING"
hosts/        "this public key proves the server I reached really IS server X"
```

## identities/

One file per device, named `<device>.ed25519.pub`. These are the keys that go
into an `authorized_keys` file, or into GitHub, to let that device in.

| file | key comment | what it is |
|---|---|---|
| `main-pc.ed25519.pub` | `main-pc@hackthe.world` | main-pc's user key |
| `linux-server.ed25519.pub` | `linux-server-deploy` | read-only deploy key for pulling `hackthe-world/devices` |
| `gateway.ed25519.pub` | `gateway-deploy` | same idea, on the gateway |
| `winvm.ed25519.pub` | `winvm` | the Windows guest's user key |

`linux-pc` and `phone` are in the device inventory but not built yet, so they
have no keys to publish.

## hosts/

One directory per host, each with the host's ed25519 public key and the SHA256
fingerprints of **all** its host keys:

```
hosts/<name>/ed25519.pub        the key itself, ready for known_hosts
hosts/<name>/fingerprints.txt   `ssh-keygen -lf` output, one line per key type
```

Use these to answer "is this the real box?" when ssh shows you an unknown-host
prompt. Compare what ssh offers against `fingerprints.txt`:

```sh
ssh-keyscan -t ed25519 <host> | ssh-keygen -lf -
```

If it does not match a line in that file, **do not type yes.**

To trust a host up front instead of being asked, append its key to known_hosts
with the hostname you actually use:

```sh
printf '%s %s\n' <hostname> "$(cat hosts/linux-server/ed25519.pub)" >> ~/.ssh/known_hosts
```

### A note on where host keys live

On Linux hosts (`gateway`, `linux-server`) host keys are in `/etc/ssh/`.

On the Windows hosts (`main-pc`, `winvm`) they are in `C:\ProgramData\ssh\` and
**must stay there** — sshd runs as a service and reads them from that path, with
ACLs that only SYSTEM and Administrators can read. Only *user* SSH material
(`id_ed25519`, `config`, `known_hosts`, `authorized_keys`) lives in `~/.ssh` on
those machines.

`main-pc`'s fingerprints were read from its running sshd with `ssh-keyscan`
rather than from disk, which is why its keys show `no comment` — keyscan does
not carry the comment field. The keys themselves are identical.
