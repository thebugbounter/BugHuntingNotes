___
# Bug Hunting Recon Process


### **Step 1: Subdomain Enumeration**

1. **Subfinder:**
    
    - Find subdomains using Subfinder.
    
    ```bash
    subfinder -d abc.com -all -recursive > subdomain.txt
    ```
    
2. **Assetfinder:**
    
    - Find subdomains using Assetfinder.
    
    ```bash
    assetfinder abc.com > assetfinder.txt
    ```
    
3. **Combine & Deduplicate Subdomains:**
    
    - Combine and deduplicate the subdomains found by both tools.
    
    ```bash
    sort -u subdomain.txt assetfinder.txt > total_subdomains.txt
    ```
    
4. **Check for Subdomain Takeover:**
    
    - Use Subzy to check for subdomain takeovers.
    
    ```bash
    sudo subzy run -targets total_subdomains.txt
    ```
    

---

### **Step 2: Host Discovery**

1. **Check Live Hosts:**
    
    - Use httpx to find live hosts from the list of subdomains.
    
    ```bash
    httpx -l total_subdomains.txt -o livehosts.txt
    ```
    
2. **Check for Open Ports:**
    
    - Use `httpx-toolkit` to check for open ports (80, 443, 8080, 8000, 8888).
    
    ```bash
    cat subdomain.txt | httpx-toolkit -ports 80,443,8080,8000,8888 -threads 200 > subdomains_alive.txt
    ```
    
3. **Filter Based on Response Codes (200, 403, 400, 500):**
    
    - Use httpx to filter out responses based on certain status codes.
    
    ```bash
    cat subdomain.txt | httpx-toolkit -ports 80,443,8080,8000,8888 -mc 200,403,400,500 -o live.txt
    ```
    
4. **Check Status Codes of Live Hosts:**
    
    - Use httpx to check the status codes of live hosts.
    
    ```bash
    cat live.txt | httpx -status-code
    ```
    

---

### **Step 3: URL Discovery & Enumeration**

1. **Gather URLs using Katana:**
    
    - Use Katana to extract URLs from various sources (Wayback Machine, CommonCrawl, AlienVault).
    
    ```bash
    katana -u subdomains_alive.txt -d 5 -ps -pss waybackarchive,commoncrawl,alienvault -kf -jc -fx -ef woff,css,png,svg,jpg,woff2,jpeg,gif,svg -o allurls.txt
    ```
    
2. **Find Sensitive Files:**
    
    - Search for potentially sensitive files like `.txt`, `.log`, `.config`, `.db`, etc.
    
    ```bash
    cat allurls.txt | grep -E "\.txt|\.log|\.cache|\.secret|\.db|\.backup|\.yml|\.json|\.gz|\.rar|\.zip|\.config"
    ```
    

---

### **Step 4: JavaScript File Enumeration**

1. **Extract JavaScript Files:**
    
    - Extract URLs ending with `.js`.
    
    ```bash
    cat allurls.txt | grep -E "\.js$" > js.txt
    ```
    
2. **Scan JavaScript Files for Exposures:**
    
    - Use Nuclei to scan the JavaScript files for exposures.
    
    ```bash
    cat js.txt | nuclei -t /path/to/nuclei-templates/http/exposures/
    ```
    

---

### **Step 5: Directory Bruteforce**

1. **Fuzz Directories and Files:**
    
    - Use Dirsearch to fuzz directories and files on the target website.
    
    ```bash
    dirsearch -u https://abc.com -e conf,config,bak,backup,swp,old,db,sql,asp,aspx,asp-,py,py-,rb,php,php-,bkp,cache,cgi,conf,csv,html,inc,jar,js,json,jsp,lock,log,rar,sql,tar,tar.gz,txt,wadl,zip,.log,.xml
    ```
    

---

### **Step 6: Vulnerability Scanning**

2. **Check for CORS Misconfiguration:**
    
    - Use Corsy to check for CORS misconfigurations.
    
    ```bash
    python Corsy -i subdomains_alive.txt -t 100
    ```
    
3. **LFI (Local File Inclusion) Scanning:**
    
    - Use ffuf and payloads to check for LFI vulnerabilities.
    
    ```bash
    cat lfi_candidates.txt | xargs -I {} sh -c 'ffuf -u "{}?file=FUZZ" -w /path/to/LFI_payloads.txt -v -mr "root:x:0:0:" -o lfi_results_$(echo {} | sed "s/[^a-zA-Z0-9]/_/g").txt'
    ```
    
4. **SQL Injection (SQLi) Scanning:**
    
    - Use gau, urldedupe, and gf to find SQLi vulnerabilities.
    
    ```bash
    echo abc.com | gau | urldedupe -qs | gf sqli
    ```
    
    - Use sqlmap to scan the identified parameters for SQLi.
    
    ```bash
    sqlmap -m parameters.txt --batch --level=5 --risk=3 --dbs
    ```
    
5. **Open Redirect Scanning:**
    
    - Use gf and openredirex to find open redirects.
    
    ```bash
    cat allurls.txt | gf redirect | openredirex -p <Payloads>
    ```
    
    - Gather endpoints and find redirects.
    
    ```bash
    cat endpoints.txt | gau | urldedupe -qs | gf redirect > redirect.txt
    ```
    

---

### **Step 7: Port Scanning**

6. **Scan Ports with Nmap:**
    
    - Use Nmap to scan open ports on the subdomains.
    
    ```bash
    nmap -iL subdomains.txt -T4 -oN nmap_scan.txt
    ```
    

---

### **Step 8: XSS Vulnerability Scanning**

7. **XSS Scanning using Nuclei:**
    
    - Use Nuclei to scan for XSS vulnerabilities.
    
    ```bash
    nuclei -l livehosts.txt -tags xss
    ```
    

---

