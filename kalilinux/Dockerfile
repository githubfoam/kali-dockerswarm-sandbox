FROM kalilinux/kali

ENV BASE_TOOLZ="aircrack-ng crackmapexec crunch curl dirb dirbuster dnsenum dnsrecon dnsutils dos2unix enum4linux exploitdb ftp git gobuster hashcat hping3 hydra impacket-scripts john joomscan masscan metasploit-framework mimikatz nasm ncat netcat-traditional nikto nmap patator php powersploit proxychains python-impacket python-pip python2 python3 recon-ng responder samba samdump2 smbclient smbmap snmp socat sqlmap sslscan sslstrip theharvester vim wafw00f weevely wfuzz whois wordlists wpscan ruby-dev gcc libffi-dev make"

RUN set -euxo  && \
    apt-get update && apt-get install -y --no-install-recommends ${BASE_TOOLZ} && \
    groupadd -r -g 998 builder && \
    useradd -r -u 998 -g 998 -M -c "docker builder account" -d /tmp builder && \
    gem install winrm winrm-fs colorize stringio evil-winrm && \
    echo "alias l='ls -al'" >> /root/.bashrc && \
    echo "alias nse='ls /usr/share/nmap/scripts | grep '" >> /root/.bashrc && \
    echo "alias scan-range='nmap -T5 -n -sn'" >> /root/.bashrc && \
    echo "alias http-server='python3 -m http.server 8080'" >> /root/.bashrc && \
    echo "alias php-server='php -S 127.0.0.1:8080 -t .'" >> /root/.bashrc && \
    echo "alias ftp-server='python -m pyftpdlib -u \"admin\" -P \"S3cur3d_Ftp_3rv3r\" -p 2121'" >> /root/.bashrc
# login user
USER builder
# Set working directory to user builder
WORKDIR /tmp
# Open shell
CMD ["/bin/bash"]
