
# Stats and Sort Commands

## What is stats command in SPL?
- stats command in Splunk is used to perform statistical and transformation operations on events  
- this command takes the output from the previous pipeline command and performs operations on those events  

- basic syntax
```
stats function(field) by grouping_field
```

### Purpose 
- to count number of events by particular fields  
- to summarize events by finding values like sum, average, minimum, maximum, etc  
- stats is one of the most important SPL commands because it helps transform raw data into meaningful results  

---

## Important Stats Command Functions 

### count
- this function is used to count the total number of events grouped by one or more fields  

- syntax:
```
stats count by field_1, field_2
```
- **example**
```
stats count by Account_Name
```

- this command counts events for each account name, for example:
  - admin → 340  
  - bobsmith → 880  



### dc() (distinct count)
- this function counts unique values of a field  
- if a field contains repeated values, `dc()` only counts the distinct ones  

- for example, if a field contains:
```
ip1, ip1, ip2, ip3, ip3, ip4
```

- the result will be:
```
4
```

- syntax:
```
stats dc(field_name) by field_name
```
- **example**
```
stats dc(src_ip)
```

- this command returns the total number of unique source IP addresses  



### values()
- this function returns the actual unique values from a field  
- unlike `dc()` which returns only the count, `values()` lists the values themselves  

- syntax:
```
stats values(field_name) by field_name
```

- **example**
```
stats values(dest_ip)
```

- this command returns the list of unique destination IP addresses instead of just the count  

### sum()
- this function adds all numeric values of a field together  
- mostly used for calculating total traffic, bytes transferred, or total requests  

- syntax:
```
stats sum(field_name) by field_name
```

- **example**

```
stats sum(bytes) by src_ip
```


- this command returns total bytes sent by each source IP  



### min()
- this function returns the minimum or smallest value from a field  

- syntax:
```
stats min(field_name) by field_name
```

- **example**
```
stats min(response_time) by host
```
- this command returns the lowest response time for each host  

### max()
- this function returns the maximum or largest value from a field  

- syntax:
```
stats max(field_name) by field_name
```
- **example**

```
stats max(bytes) by src_ip
```

- this command returns the highest number of bytes sent by each source IP  

### avg()
- this function calculates the average value of a numeric field  

- syntax:
```
stats avg(field_name) by field_name
```

- **example**
```
stats avg(bytes) by src_ip
```

- this command returns the average number of bytes sent by each source IP  
---
## Sort Command

### What is sort command?
- sort command is used to arrange search results in a specific order
- by default, Splunk sorts results in descending order
- it is commonly used after commands like `stats`, `table`, or `chart` to make results easier to analyze

- syntax:

```text
... | sort field_name
```

### Ascending Sort

- ascending sort arranges values from smallest to largest

- syntax:

```text
... | sort field_name
```

- **example**

```text
index=windows EventCode=4625
| stats count by user
| sort count
```

- this command sorts users by failed login count from lowest to highest

### Descending Sort

- descending sort arranges values from largest to smallest

- syntax:

```text
... | sort - field_name
```

- **example**

```text
index=windows EventCode=4625
| stats count by user
| sort - count
```

- this command sorts users by failed login count from highest to lowest

### Sort Multiple Fields

- sort can also use multiple fields

- syntax:

```text
... | sort - field_1 field_2
```

- **example**

```text
| stats count by src_ip user
| sort - count user
```

- first sorts by count in descending order
- if two records have the same count, they are sorted by user name

### Why Use Sort?

- helps identify top users, IPs, hosts, or processes quickly
- useful after `stats`, `chart`, or `table` commands
- makes large result sets easier to read and investigate