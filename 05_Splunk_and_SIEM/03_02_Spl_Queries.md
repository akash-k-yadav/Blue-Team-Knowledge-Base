
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

