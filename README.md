This are config files for using stunnel to tunnel ssh connections
inside TLS on port 443, for defeating application-level firewalls
that detect the ssh protocol. The implementation is simple:
on the client side, `ProxyCommand` (see ssh_config(5) for details)
is used to wrap the ssh connection in TLS, and on the server side,
stunnel(1) is used to terminate TLS and forward the resulting connection
to the local sshd.

(Forked from https://github.com/slingamn/inconveniences/tree/master/system/ssh_tls_proxy
and modified to use OpenSSL instead for the go program and stunnel).

Even smarter solution, not requering stunnel but using ALPN proxy rules in nginx,
even allowing normal https on the same port!
https://superuser.com/a/1328474

Server-side configuration:

1. You need a stunnel instance that listens on port 443, then terminates TLS
and forwards the connection to your sshd (typically on port 22).
`sshd443_server.conf` is a stunnel config file that handles this.
2. Listening on port 443 requires privileges. On a systemd system, you can
install `sshd443.service` and `sshd443.socket` in `/etc/systemd/system` and
enable them with `systemctl enable`; this will allow systemd to listen on 443
and then pass the listen socket to an unprivileged stunnel process. With this
setup, you may need to disable stunnel's own systemd unit, e.g.
`systemctl disable stunnel4`.
3. For a simpler configuration, you can modify `sshd443_server.conf` to listen
on 443 directly, then run it as root.

Client-side configuration:

1. Add the following block to ~/.ssh/config:

```
Host my-host.domain
    ProxyCommand openssl s_client -connect %h:443 -ign_eof
```

If you don't have have a valid TLS certificate for the server, OpenSSL will
still connect.
This is probably OK because you can rely on the known_hosts file to
authenticate the actual sshd.

