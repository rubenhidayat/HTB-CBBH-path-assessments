# INFORMATION GATHERING - WEB EDITION SKILL ASSESSMENT
## Overview & Objective
This skill assessment covers the practical application for passive reconnaissance and active reconnaissance to solve given questions. For each question, I will breakdown the question, explain relevant terminology, detail the tools I used, and outline my methodology with step-by-step instructions.

Disclaimer: This write-up focuses on documenting the methodologies, tools, and learning process involved in completing the Hack The Box Bug Bounty Path Skill Assessments. To maintain the integrity of the challenges and promote independent learning, direct answers or flags will not be disclosed.

## Questions and Answers
### Setting Up
Before starting the assessment, make sure to add the target machine's IP into `/etc/hosts`, pairing it with the domain as the target IP is a docker container and not a public domain that can be reached using DNS lookup. 

Feel free to use text editors like `nano`, `vim`, or `sublime text` to add the IP-domain pair like the image below : ![1](https://github.com/user-attachments/assets/df7b12d3-7b66-4fd8-b4b7-8459d32b6e1e)

Or I can just add it using command line like example below :
```bash
sudo sh -c 'echo "<SERVER-IP> inlanefreight.htb" >> /etc/hosts'
```

This way we can access `inlanefreight.htb` straight from browser without any problem

---

### Question 1
What is the IANA ID of the registrar of the inlanefreight.com domain?

#### Answer :
IANA ID is a numerical code to accredited domain name registrar, it is used to identify which company manages a domain's registration.

To look up IANA ID, I can use `whois`, a command line tool to query public information about internet resources. To lookup IANA ID for `inlanefreight.com` I can use `grep` to filter the output:

```bash
whois inlanefreight.com | grep "IANA ID"
```

I should find the IANA ID within the `whois` output.

---

### Question 2
What http server software is powering the inlanefreight.htb site on the target system? Respond with the name of the software, not the version, e.g., Apache.

#### Answer
There are multiple ways to find out the server software of a website. One is to trigger error page, such as `404 Not Found`, the website will usually display the server software along with its version underneath the error message. Another way is to use `cURL`, a tool for sending arbitrary web requests through command line which will respond with web responses, and the server software is typically displayed within the response headers. The command for `cURL` command is :  
```bash
curl inlanefreight.htb:<port> -I
```
- Command Explanation : 
	- `-I` flag to show only the response header

---

### Question 3
What is the API key in the hidden admin directory that you have discovered on the target system?

#### Answer
The solution for this question is a bit tricky, since it does not mention what or where the hidden directory is, it could be hidden deep inside the file structure. So normally I would check `/robots.txt` since it could contain hidden files. Or I could `fuzz` the directory to find any potentially relevant admin directory that could store the API key. `fuzz` or fuzzing is an automated technique to discover hidden files/directories on a web server by guessing names from a wordlist. I will use `FFUF` to fuzz the web directory, the command for that is :

```bash
ffuf -u inlanefreight.htb:<port>/FUZZ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -ic
```
- Command Explanation
	- `-u` : Provide the URL and PORT
	- `-w` : Provide the path to wordlist
	- `-ic` : Removes the copyright text inside the output

After scanning, there's no directory to be found. I was thinking where's another way for a website to store a directory other than inside the main domain, and then I decided to try to use subdomain fuzzing, since it could be some hidden subdomain that i can find inside the website. Using the same tools, I fuzzed the subdomain using the following command, the first scan will display all results as valid since some VHosts are set to respond with `200` response code, so I cancel it immediately and look for the content size to use it for filtering :

```bash
ffuf -u http://inlanefreight.htb:<port> -H "HOST: FUZZ.inlanefreight.htb" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 200 -fs <number>
```
- Explanation :
	- `-t 200` : use 200 threads for faster fuzzing though this could overload the target server
	- `-fs <number>` : Filter the content size 

The subdomain will be found as shown in the redacted image below:
![3](https://github.com/user-attachments/assets/642ad805-fe87-4b54-aea2-670a2c2afb58)


Before visiting the site with the newly found subdomain, input the new domain into `/etc/hosts` so it can be accessed
```bash
sudo sh -c 'echo "<SUBDOMAIN>.<SERVER-IP> inlanefreight.htb" >> /etc/hosts'
```

Visit the site and investigate if there's anything relevant. If not, try to look for `/robots.txt` to see potential hidden web content. The hidden admin page can be found there.

Visit the hidden admin page and grab the flag.

---

### Question 4
After crawling the inlanefreight.htb domain on the target system, what is the email address you have found? Respond with the full email, e.g., mail@inlanefreight.htb.

#### Answer
The question was asking to crawl the website, crawling means using a tool to automatically explore the website by visiting links available inside the webpages. From the pages that I investigated so far, there's nothing for the crawler to visit since there's no link that can be accessed. So there must be another VHost subdomain related to the newly found subdomain that can be fuzzed.

Use the same VHost subdomain fuzzing command earlier with added subdomain :
```bash
ffuf -u http://inlanefreight.htb:<port> -H "HOST: FUZZ.<subdomain>.inlanefreight.htb" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 200 -fs <number>
```

The new subdomain will be found as shown in the redacted image below: 
![4](https://github.com/user-attachments/assets/a978be71-2d02-4a35-a718-0b28dde02cd6)

Using the new subdomain, start crawling using a tool called ReconSpider, it's a python based crawler that will generate output in a JSON file called `results.JSON`. Use command below :
```bash
python3 ReconSpider.py http://<subdomain>.<subdomain>.inlanefreight.htb:<port>
```

Use cat command to see `results.JSON` or use your preferable text editor: 
```bash
cat results.json
```

The answer can be found in the file

---

### Question 5
What is the API key the inlanefreight.htb developers will be changing too?

### Answer
The answer for this question can be found in the `results.JSON` 
