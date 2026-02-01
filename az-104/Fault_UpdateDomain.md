“Fault Domains protect against physical hardware failures like rack, power or network outages, 
while Update Domains protect against planned Azure maintenance and patching. 
Azure ensures that VMs in an Availability Set are distributed across both fault domains and 
update domains so that a single failure or maintenance event never takes down all instances of an application.”

| Deployment                | Azure SLA  |
| ------------------------- | ---------- |
| Single VM                 | **99.9%**  |
| Availability Set (2+ VMs) | **99.95%** |
| Availability Zones        | **99.99%** |
