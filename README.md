# Apache Web Server Log Analysis

## Introduction
In this project, we will do log analysis by working with Apache web server logs. Apache logs are a rich source of information about web traffic and can help identify potential security incidents, usage patterns, and performance issues. By the end of this project, students will be able to extract and interpret valuable information from Apache access and error logs.

## Pre-requisites
- Apache web server installed

## Lab and Tools I used
- Linux distribution with Apache installed
- Apache web server installed and running
- Access to Apache log files (located in `/var/log/apache2/`)

---

## Exercise 1: Accessing Apache Log Files

### Steps:

1. On your Linux machine, open the terminal.
2. Navigate to the Apache log directory:

```bash
cd /var/log/apache2/
```

3. List the log files:

```bash
ls -l
```

### Expected Output:
- A list of Apache log files such as `access.log` and `error.log`.

---

## Exercise 2: Understanding Access Logs

### Steps:

1. Open the access log file:

```bash
less access.log
```

2. Observe log structure:
   - IP address  
   - Timestamp  
   - Request method  
   - URL  
   - HTTP status code  
   - User agent  

3. Identify and analyze entries.

### Expected Output:
- Understanding of Apache access log format.

---

## Exercise 3: Filtering Log Entries

### Steps:

1. Filter by IP address:

```bash
grep '192.168.1.100' access.log
```

2. Filter by HTTP 404 errors:

```bash
grep ' 404 ' access.log
```

3. Combine filters:

```bash
grep '192.168.1.100' access.log | grep ' 404 '
```

### Expected Output:
- Filtered log results based on conditions.

---

## Exercise 4: Analyzing Error Logs

### Steps:

1. Open error log:

```bash
less error.log
```

2. Look for:
   - Missing files  
   - Permission errors  
   - Script failures  

3. Note timestamps and frequency.

### Expected Output:
- Understanding of server error patterns.

---

## Exercise 5: Summarizing Log Data

### Steps:

1. Requests per IP:

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr
```

2. Requests per day:

```bash
awk '{print $4}' access.log | cut -d: -f1 | sort | uniq -c
```

3. Most requested URLs:

```bash
awk '{print $7}' access.log | sort | uniq -c | sort -nr
```

### Expected Output:
- Summary of traffic by IP, date, and URL.

---

### SOC Security Analysis & Incident Detection

During log analysis several suspicious patterns were identified that may indicate potential security threats.

---

### 1. Possible Brute Force Attack Detection

Using the access logs, repeated failed login attempts were observed from the same IP address.

```bash
grep " 401 " access.log
grep " 403 " access.log
```

#### Findings:
- Multiple unauthorized access attempts (HTTP 401/403 errors)
- Same IP repeating login requests in a short time frame

#### Interpretation:
This behavior is consistent with a **brute force login attack**, where an attacker attempts multiple username/password combinations.

#### 🛡️ Recommendation:
- Block suspicious IP using firewall (iptables / ufw)
- Enable rate limiting
- Implement account lockout policies

---

### 🚨 2. Possible Directory Scanning / Reconnaissance

High number of 404 errors were detected from specific IPs.

```bash
grep " 404 " access.log
```

#### 🔍 Findings:
- Repeated requests to non existent pages
- Pattern of systematic URL guessing

#### ⚠️ Interpretation:
This indicates a possible **directory brute force / reconnaissance attack**, where an attacker scans for hidden files or admin pages.

#### 🛡️ Recommendation:
- Use Web Application Firewall (WAF)
- Hide sensitive endpoints
- Monitor repeated 404 spikes

---

### 🚨 3. Suspicious High Traffic IP

Top requesting IP addresses were analyzed:

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr
```

#### 🔍 Findings:
- One IP generated unusually high number of requests

#### ⚠️ Interpretation:
This may indicate:
- Bot activity
- Potential DDoS attempt
- Automated scanning tool

#### 🛡️ Recommendation:
- Rate limiting per IP
- Temporary IP blocking
- Monitor traffic baseline

---

### 📊 SOC Summary

From the log analysis:
- Detected brute force-like behavior (401/403 spikes)
- Detected reconnaissance activity (404 scanning)
- Identified high-frequency traffic sources
