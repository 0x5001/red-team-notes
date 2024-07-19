# Domain Enumeration - BloodHound
- Bloodhound is a GUI tool that provides AD entities and relationships for the data collected by its ingestors.
- It uses graph theory for providing the capability of mapping shortest path for interesting things like Domain Admins.
- There are built-in queries for frequently used actions.
- It also supports custom Cypher queries.
- Ingestors are tools that collects data.
- This collected data can be uploaded to BloodHound to analyze Active Directory environment.
- Supply data to bloodhound: `Invoke-BloodHound -CollectionMethod all` or `sharphound.exe`.
- The gathered data from ingestors can be uploaded to BloodHound application.
