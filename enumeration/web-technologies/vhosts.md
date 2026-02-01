# VHosts

Tools and example commands for virtual host enumeration.

## Tools

### virtual-host-discovery

https://github.com/jobertabma/virtual-host-discovery

Example:

```bash
ruby scan.rb --ip=192.168.1.101 --host=domain.tld
```

### vhosts-sieve

https://github.com/dariusztytko/vhosts-sieve

Example:

```bash
python3 vhosts-sieve.py -d domains.txt -o vhosts.txt
```

### fierce

(fierce DNS scanner)

Example:

```bash
fierce -dns example.com
```

### VHostScan

https://github.com/codingo/VHostScan

Example:

```bash
VHostScan -t example.com
```

## Techniques

Reference: https://pentestbook.six2dez.com/enumeration/webservices/vhosts#techniques

Copy of common techniques:

```bash
# ffuf
badresponse=$(curl -s -H "host: totallynotexistsforsure.bugcrowd.com" https://bugcrowd.com | wc -c)
ffuf -u https://TARGET.com -H "Host: FUZZ.TARGET.com" -w werdlists/dns-hostnames/nmap-vhosts-all.txt -fs $badresponse

# Manual with subdomains list
for sub in $(cat subdomains.txt); do
    echo "$sub $(dig +short a $sub | tail -n1)" | anew -q subdomains_ips.txt
done
```
