---
Creation date (YYYY-MM-DD) and time: <% tp.file.creation_date() %>
LHOST: 
LPORT: 
RHOST: 
RPORT: 
DOMAIN: 
user: 
password: 
---
### RECON
1. Add the RHOST and DOMAIN to your `/etc/hosts` file (if you do not care where it lands):
```zsh
sudo sed -i "$(awk '!/^#/ {print NR; exit}' /etc/hosts)s/^/<% tp.frontmatter["RHOST"] %>   <% tp.frontmatter["DOMAIN"] %>\n/" /etc/hosts && cat /etc/hosts && ping -c 5 <% tp.frontmatter["RHOST"] %>
```
2. 1st lane:
```zsh
sudo sed -i '1i <% tp.frontmatter["RHOST"] %>    <% tp.frontmatter["DOMAIN"] %>' /etc/hosts && cat /etc/hosts && ping -c 5 <% tp.frontmatter["RHOST"] %>
```
3. Now, run:
```zsh
sudo nmap <% tp.frontmatter["RHOST"] %> -Pn --disable-arp-ping -sV -p- --stats-every=10s -v -sS -n -oA nmap_<% tp.frontmatter["RHOST"] %>
```
4. UDP NMAP scan (ONLY IF STUCK) chose your ports with `-F` (default), `-p x` or `-p-:
```zsh
sudo nmap <% tp.frontmatter["RHOST"] %> -Pn -sU -F --stats-every=10s -v -n -oA nmap_<% tp.frontmatter["RHOST"] %>
```
5. NMAP scan with decoys (USE SPARINGLY):
```zsh
sudo nmap <% tp.frontmatter["RHOST"] %> -Pn --disable-arp-ping -sV -p- --stats-every=10s -v -sS -n -D RND:100 -oA nmap_<% tp.frontmatter["RHOST"] %>
```
6. Service enumeration:
```zsh
sudo nc -nv <% tp.frontmatter["RHOST"] %> <% tp.frontmatter["RPORT"] %>
```
7. Check SMB shares:
```zsh
smbclient -N -L //<% tp.frontmatter["RHOST"] %>
```
8. If you found a share, then plug it in and check what is going on:
```zsh
smbclient //<% tp.frontmatter["RHOST"] %>/<share>
```
9. Found Network File System?
```zsh
showmount -e <% tp.frontmatter["RHOST"] %>
```
10. Now to mount the NFS:
```zsh
mkdir 'NFS <% tp.frontmatter["RHOST"] %>' && sudo mount -t nfs <% tp.frontmatter["RHOST"] %>:/ ./NFS_<% tp.frontmatter["RHOST"] %>/ -o nolock && cd NFS_<% tp.frontmatter["RHOST"] %> && tree .
```
11. Footprint SNMP if present:
```zsh
snmpwalk -v2c -c public <% tp.frontmatter["RHOST"] %>
```
12. 161:
```zsh
onesixtyone -c /usr/share/wordlists/seclists/Discovery/SNMP/snmp.txt <% tp.frontmatter["RHOST"] %>
```
13. Subdomains BF:
```zsh
gobuster dns -q -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt --domain <% tp.frontmatter["DOMAIN"] %> -o gobuster_dns_<% tp.frontmatter["RHOST"] %>
```
14. Can you perform the zone transfer?
```zsh
dig axfr <% tp.frontmatter["DOMAIN"] %> @<% tp.frontmatter["RHOST"] %>
```
15. No DNS hit, then is it possible that the virtual host is running here?
```zsh
gobuster vhost -q -u <% tp.frontmatter["DOMAIN"] %>:<% tp.frontmatter["RPORT"] %> --append-domain -o gobuster_vhost_<% tp.frontmatter["RHOST"] %> -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
```
16. Check for directories:
```zsh
gobuster dir -q -u <% tp.frontmatter["DOMAIN"] %>:<% tp.frontmatter["RPORT"] %> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster_dir_<% tp.frontmatter["RHOST"] %>
```
17. Let's check banners:
```zsh
curl -I http://<% tp.frontmatter["DOMAIN"] %>
```
18. Use `wafw00f`:
```zsh
wafw00f http://<% tp.frontmatter["DOMAIN"] %>:<% tp.frontmatter["RPORT"] %>
```
19. Can `nikto` help you?
```zsh
nikto -h http://<% tp.frontmatter["DOMAIN"] %>:<% tp.frontmatter["RPORT"] %>
```
20. Let's crawl:
```zsh
python3 ReconSpider.py http://<% tp.frontmatter["DOMAIN"] %>:<% tp.frontmatter["RPORT"] %> > reconspider_<% tp.frontmatter["RHOST"] %>
```
21. Also, try `finalrecon`:
```zsh
finalrecon --full --url http://<% tp.frontmatter["DOMAIN"] %>:<% tp.frontmatter["RPORT"] %>
```
22. Try `feroxbuster` for recursive BF:
```zsh
feroxbuster -u http://<% tp.frontmatter["DOMAIN"] %>:<% tp.frontmatter["RPORT"] %> -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 100  -o feroxbuster_<% tp.frontmatter["RHOST"] %>
```
23. ⚠ IF wordlists did not work out, use `cewl` to build a new wordlist, and go back to step # 12:
```zsh
cewl -d 3 -w cewl_<% tp.frontmatter["DOMAIN"] %>.txt http://<% tp.frontmatter["DOMAIN"] %>:<% tp.frontmatter["RPORT"] %>
```
