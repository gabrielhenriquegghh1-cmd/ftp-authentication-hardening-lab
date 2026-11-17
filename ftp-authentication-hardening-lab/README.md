# FTP Authentication and Hardening Lab

[Leia em português](README.pt-BR.md) · [Publishing guide](PUBLISHING.md#english)

This DIO cybersecurity project examines how a legacy FTP service responds to valid and rejected logins and what evidence the server records. The practical work used manual authentication. Medusa, brute-force attempts, and password spraying are discussed as concepts in the course.

**Result.** The Metasploitable 2 guest responded to all three ping requests. The FTP client observed a successful login and a rejected login, and the server log showed authentication failures. TCP port 21 remained open on the last recorded check, so the effect of stopping FTP is still unverified.

## Lab setup

| Component | Observed configuration |
| --- | --- |
| Virtualization | Oracle VirtualBox |
| Server | Metasploitable 2 on `192.168.56.101/24` |
| Network | Server Adapter 1 configured as Host-only on `vboxnet0` |
| Client | Linux terminal using the VirtualBox host-side interface `vboxnet0` at `192.168.56.1/24` |
| FTP service | Server banner reported `vsFTPd 2.3.4` on TCP 21 |
| Evidence date | 17 November 2026, as shown in the server’s FTP log |

The client-side screenshots show the host-side VirtualBox interface. The evidence therefore documents host-to-guest testing over the Host-only network.

## Method and evidence

From the Linux client, the connectivity check used the guest IPv4 address shown by `ifconfig`:

```bash
LAB_IP=192.168.56.101
ping -c 3 "$LAB_IP"
nc -vz -w 3 "$LAB_IP" 21
```

The FTP client was opened interactively using `ftp "$LAB_IP"`. Two observed sessions had different usernames. The accepted session used the dedicated `ftp-lab` account. That difference limits the comparison because account identity also changed between the two outcomes.

| Check | Observation | Evidence |
| --- | --- | --- |
| Virtual network | Adapter 1 attached to the Host-only network `vboxnet0` | [VirtualBox settings](images/01-host-only-network.png) |
| Server addressing | `eth0` reported `192.168.56.101/24` | [Server interface](images/02-server-ipv4.png) |
| Baseline | 3 of 3 ping replies, 0% loss, and TCP 21 open | [Client connectivity check](images/03-ftp-connectivity.png) |
| Manual authentication | Rejected session returned `530 Login incorrect`. The dedicated account returned `230 Login successful`. | [Client replies](images/04-ftp-login-outcomes.png) |
| Server logging | The FTP log excerpt showed two `FAIL LOGIN` entries from `192.168.56.1` at 15:11:47 and 15:13:07, server time. | [FTP failure events](images/05-ftp-failure-logs.png) |
| Repeat check | 3 of 3 ping replies, 0% loss, and TCP 21 still open | [Repeat connectivity check](images/06-ftp-connectivity-recheck.png) |

The reviewed excerpt of `/var/log/auth.log` did not show FTP events for these sessions. The service-specific FTP log showed the failures, but the available excerpt did not show a matching success for `ftp-lab`. The successful login is supported by the client reply. The client screenshots lack timestamps, which limits event-by-event correlation with the server log.

Screenshots were cropped and redacted for publication. They retain the lab addresses and test outcomes while omitting the personal username, hostname, and unrelated interface details.

## Concepts from the course

Medusa is a tool that automates authentication attempts. Brute-force guessing can produce a series of failures on one account. Password spraying tries a small number of candidate passwords across many accounts and can produce failures spread across those accounts. Compare accounts, sources, and timing during triage. This project used manual login checks and did not attribute the log events to a tool or attack technique.

## Hardening and validation

The proposed change is to stop FTP if the service is unnecessary. On the server, identify the process listening on TCP 21 and stop it through the applicable service manager. Record the listener state after the change, then run the same ping and TCP checks from the same client. A responsive guest with no local listener and a refused TCP connection is consistent with FTP being stopped. Persistence after reboot requires a separate check.

For environments that still need file transfer, use SFTP where appropriate, restrict permitted network sources, and monitor authentication failures. Repeated failures on one account or failures distributed across several accounts can guide triage. Those signals alone cannot identify the tool or technique used.

## References

- [Oracle VirtualBox User Guide · Host-only networking](https://docs.oracle.com/en/virtualization/virtualbox/7.2/user/networkingdetails.html#host-only-networking)
- [RFC 959 · File Transfer Protocol and reply codes](https://www.rfc-editor.org/rfc/rfc959.html)
- [OWASP Authentication Cheat Sheet · Login throttling and monitoring](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
