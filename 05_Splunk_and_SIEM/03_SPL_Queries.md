# SPL Queries 

## What are SPL queries 
- Search Processing Language (SPL) queries are structured commands used in Splunk to search, analyze, and visualize machine generated data  

- they act as the main interface to interact with Splunk, allowing users to extract insights from logs and metrics by filtering, transforming, and aggregating data  


### Pipeline Structure of SPL Queries 
- SPL works similar to Linux pipelines, where commands are separated by "|" pipes  
- the output of one command becomes the input for the next  
- SPL flows from left to right  

```
command1 | command2 | command3
```

- *Explanation:-* command1 runs first, its output is passed to command2, then command2 output goes to command3, and the final result is returned  



### Components of SPL Query 
- an SPL query can include:
  - key value pairs  
  - literal strings  
  - commands  
  - logical operators  
  - comparison operators  
  - clauses  
  - pipes  

---

## Search Command

- syntax:

```
index=main sourcetype="stream:dns" host="xyz"
```

- this is the basic search format and most queries start like this  
- index is the storage location of logs (like a bucket)  
- sourcetype tells Splunk the format of the data and how to parse it  
- host is optional and helps filter logs from a specific machine  

- search acts like a filter, removing unwanted data and keeping only relevant results  

- **Note:-** index and sourcetype help narrow down the search. since SIEM tools handle large volumes of data, using them reduces unnecessary processing and improves performance  


### Search by Keyword 

```
index=windows "failed password"
```

- searches for the phrase "failed password" anywhere in raw logs  

```
index=windows "192.168.1.50"
```

- searches for logs containing the IP 192.168.1.50  

- **Note:-** keyword search can return noisy results, so it is better to use field based search when possible  


### Search by Field Value

```
index=windows EventCode=4625
```

- this is the most common method  
- uses key value pairs  
- returns logs where EventCode equals 4625  

### Search with Logic

```
index=windows EventCode=4625 AND user="admin"
```

- uses logical operators  
- this query returns events where:
  - EventCode is 4625  
  - user is admin  

- other operators include OR and NOT  


### Wildcards

```
index=windows user=admin*
index=windows src_ip=192.168.1.*
```

- admin* matches values like administrator, admin123, admin_backup  
- 192.168.1.* matches all IPs in that subnet  


### The Full Picture
```
index=windows sourcetype=WinEventLog:Security EventCode=4625 NOT user=guest
```

- reading in plain English:

- `index=windows` → search in Windows logs  
- `sourcetype=WinEventLog:Security` → only security logs  
- `EventCode=4625` → failed login events  
- `NOT user=guest` → exclude guest account  

- this query filters logs step by step to give focused results  

