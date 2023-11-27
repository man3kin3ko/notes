During an external penetration test or a bug bounty campaign it is important to complete a search-based reconnaissance first.

## Common methodology

1. Use Wayback Machine with [waybackurls](https://github.com/tomnomnom/waybackurls) and [ParamSpider](https://github.com/devanshbatham/ParamSpider) to find previously discovered endpoints and parameters.
2. Use Google dorks to find application's subdomains, like `*.example.com` and `*.*.example.com`.
3. Use Google dorks to find interesting parameters, based on the type of vulnerability you want to find. See the next paragraph.
4. At this point, you can try to fuzz URL paths using dictionaries. Don't forget to brute those directories which are listed in `robots.txt` first. If you see an IIS server, use [ShortName-Scanner](https://github.com/irsdl/IIS-ShortName-Scanner).
5. Search for possible leakages in file share services: `site:pastebin.com | site:s3.amazonaws.com | site:drive.google.com | site:onedrive.live | site:dl.dropbox.com | site:digitaloceanspaces.com | site:trello.com` and `site:docs.google.com inurl:"/d/"`. Also check VK files. It also can be helpful in a fishing campaign.
6. If you have found internal domain name, search for it in the GitHub to find possible credential leakage. Perform a subdomain bruteforce in order to find more internal domains and IP addresses.
7. At the point when you have decided that you found enough subdomains, worth trying to find dangling CNAME records in order to try any [takeovers](https://github.com/EdOverflow/can-i-take-over-xyz).  
8. If a quick view on the subdomains list shows you that too many hostnames point to a single IP address and it is not a wildcard domain, use [IP history service](https://viewdns.info/iphistory/) against this address. I bet that you just have found a reverse proxy host.

## Search for the specific vulnerability

> Notice: Google can ignore several operators if he thinks that you need too much, so use 2-5 at once
### Information Disclosure

```
ext:txt | ext:sql | ext:cnf | ext:config | ext:log | ext:ini | ext:env | ext:sh | ext:bak | ext:backup | ext:swp | ext:old | ext:htpasswd | ext:htaccess
```

```
intext:"Index of" & "Parent directory"
```
### SSRF or Open Redirect

```
inurl:"=http" | inurl:"%3dhttp" | inurl:"%3d%2f"
```

```
inurl:url= | inurl:return= | inurl:next= | inurl:redirect= | inurl:redir= | inurl:ret= | inurl:state= | inurl:dest= | inurl:callback= | inurl:open= | inurl:show= | inurl:view=
```
### LFI

```
inurl:"include="" | inurl:"dir=" | inurl:"detail=" | inurl:"file=" | inurl:"folder=" | inurl:"inc=" | inurl:"page=" | inurl:"doc=" | inurl:"document=" | inurl:"folder=" | inurl:"path=" | inurl:"style="
```

## References

- https://github.com/TakSec/google-dorks-bug-bounty
- https://cheatsheet.haax.fr/open-source-intelligence-osint/dorks/google_dorks/
- https://github.com/1ndianl33t/Gf-Patterns