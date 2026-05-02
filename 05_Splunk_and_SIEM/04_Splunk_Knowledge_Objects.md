# Knowledge Object

## What are knowledge objects
- knowledge objects are user defined entities that enrich, interpret, and transform data in Splunk  
- in simple terms, they are a way to teach Splunk what the data means  
- they help convert raw logs into meaningful and usable information  

- they answer questions like `status=404` means an error  

---

## Purpose of Knowledge Objects

- **Problem:-** in large environments, Splunk receives a huge volume of logs from different sources like firewalls, Windows events, DNS, web servers, etc  
  if you want to search something like failed login attempts per user, you have to write the same query again and again  
  different users may also write different queries for the same thing  

- **Solution:-** knowledge objects allow us to save logic once and reuse it whenever needed without rewriting queries  


### Key purposes

- no repetition :- write the SPL query once, save it as a knowledge object, and reuse it anytime  

- consistency across team :- without knowledge objects, different users may write different queries and get different results, but with knowledge objects everyone gets consistent results  

- enrich data :- raw logs may contain only basic data like `ip=192.168.56.44`, but using lookup knowledge objects we can add more context like country, city, or department  

- faster search :- pre defined knowledge objects reduce the need to write complex queries, making searches faster and more efficient  

- non technical user support :- users who are not familiar with SPL can still use saved searches, dashboards, and alerts without writing queries  

- reuse and standardization :- helps standardize common queries and detection logic across the organization  

---

## Common Knowledge Objects

### Field Extraction 
- extracts useful fields from raw logs  


### Lookup
- adds additional context using external data (like CSV files or tables)  


### Tags
- labels used to group related events for easier searching  


### Saved Searches
- a search query that is saved and reused whenever needed  


### Alert
- alert is a saved search with conditions and time settings  
- it generates a notification when specific conditions are met  

- example:
  - multiple failed logins in short time  
  - login from unusual time or location  

- **Note:-** alerts are not the same as detection rules  
  alerts notify based on defined conditions, while detection rules are part of a broader detection strategy and may involve multiple data sources and correlation  


### Event Types
- groups events based on a defined search pattern  
- helps in quickly identifying specific types of events  

---

## Key Points

- knowledge objects help convert raw data into meaningful information  
- they reduce repetition and improve efficiency  
- they ensure consistency across users and teams  
- they help enrich logs with additional context  
- they make searching faster and easier  
- they allow non technical users to interact with data  
- they are important for building detections and analysis workflows  