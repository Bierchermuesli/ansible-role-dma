# DragonFly Mail Transport Agent (DMA)

[![CI](https://github.com/Bierchermuesli/ansible-role-dma/actions/workflows/ci.yml/badge.svg)](https://github.com/Bierchermuesli/ansible-role-dma/actions/workflows/ci.yml)

Installs and configures [DragonFly Mail Transport Agent (DMA)](https://github.com/corecode/dma)
to relay mail via a smarthost. Optionally sets a catch-all alias in `/etc/aliases`.

See the Debian [DefaultMTA](https://wiki.debian.org/Debate/DefaultMTA) overview for context.

## Requirements

None. On Debian/Ubuntu, installing `dma` automatically replaces any existing MTA (e.g. exim4) via the `mail-transport-agent` virtual package conflict resolution.

## Role Variables

### Mandatory

| Variable | Description |
|---|---|
| `dma_smarthost` | Smarthost to relay mail to, e.g. `smtp.gmail.com` |

### Optional — strings (omit to exclude from config)

| Variable | DMA directive | Description | Default |
|---|---|---|---|
| `dma_port` | `PORT` | Port to connect to on the smarthost | — |
| `dma_user` | — | Username for smarthost authentication | — |
| `dma_password` | — | Password for smarthost authentication | — |
| `dma_masquerade` | `MASQUERADE` | Override envelope-from address | — |
| `dma_alias_mail` | — | Forward all local mail to this address (creates `/etc/aliases`) | — |
| `dma_mailname` | `MAILNAME` | Hostname DMA identifies itself as | `ansible_nodename` |
| `dma_fingerprint` | `FINGERPRINT` | SHA256 fingerprint to pin the smarthost certificate | — |
| `dma_certfile` | `CERTFILE` | Path to SSL certificate file | — |
| `dma_spooldir` | `SPOOLDIR` | Override spool directory | — |

### Optional — booleans

| Variable | DMA directive | Description | Default |
|---|---|---|---|
| `dma_securetransfer` | `SECURETRANSFER` | Enable TLS/SSL | `false` |
| `dma_starttls` | `STARTTLS` | Enable STARTTLS (requires `dma_securetransfer`) | `false` |
| `dma_opportunistictls` | `OPPORTUNISTIC_TLS` | Try STARTTLS but don't fail if unavailable | `false` |
| `dma_defer` | `DEFER` | Queue mail locally, deliver only on manual flush | `false` |
| `dma_fullbounce` | `FULLBOUNCE` | Include full original message in bounces | `false` |
| `dma_nullclient` | `NULLCLIENT` | Relay all mail to smarthost, no local delivery | `false` |
| `dma_insecure` | `INSECURE` | Skip certificate verification | `false` |

## Dependencies

None.

## Example Playbook

```yaml
- hosts: servers
  roles:
    - role: dma
      vars:
        dma_smarthost: smtp.example.com
        dma_port: 587
        dma_user: user@example.com
        dma_password: secret
        dma_securetransfer: true
        dma_starttls: true
        dma_masquerade: user@example.com
        dma_alias_mail: admin@example.com
```

## Reference

- [dma(8) man page](https://man.freebsd.org/cgi/man.cgi?query=dma)

## Operations

```bash
# List the mail queue
dma -bp

# Flush the queue (attempt immediate delivery)
dma -q1

# Send a test mail
echo "test body" | mail -s "test subject" user@example.com

# Check mail logs
journalctl -t dma
tail -f /var/log/mail.log

# Inspect the queue spool directly
ls /var/spool/dma/
```

## License

MIT

## Credits

Originally authored by [Niklaas Baudet von Gersdorff](https://www.niklaas.eu/).
See his [post on switching from nullmailer to DMA](https://www.niklaas.eu/posts/2018-04-08-from-nullmailer-to-dma/) for background.
