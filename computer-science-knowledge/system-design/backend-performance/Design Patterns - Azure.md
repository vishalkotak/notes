### Compute Resource Consolidation
- Having micro services deployed on different App Services for a single application is expensive
- Combining applications to be on the same computational unit will be cheaper but following considerations need to be given:
	- If two services have similar resource requirements they can be part of the same unit
	- If they have different resource requirements such as one is slow steady system that polls a queue and other is a high burst small occurrence traffic system, then we need to keep them in different computational units
### External Configuration Pattern
- Majority of application runtime environments include configuration information that's held in files deployed with the application
- However, changes to the configuration require the application be redeployed, often resulting in unacceptable downtime and other administrative overhead.
![[Pasted image 20250419173452.png]]
#### Properties
- Check for the physical capabilities of the configuration store. For example: does it provide the data format of the configuration you would like to store so that our application can parse it
- Check if it provides scoping consideration such as organization, business unit and application
- How will the configuration store respond if there are errors
- Access Controls on the configuration store
#### Examples
- Zookeeper
- Consul
### Health Endpoint Monitoring Pattern
- /healthz endpoint