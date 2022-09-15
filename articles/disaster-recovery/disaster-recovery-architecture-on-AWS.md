# Disaster Recovery Architecture on AWS

The downtime of software systems could have a significant impact on business, customer satisfaction, reputation, or income of the company. Thus maintaining the availability and durability must be the most crucial part of a software system. Disaster recovery (DR) helps engineers prepare for disaster events. This post summaries the architecture for disaster recovery on AWS.

## DR objectives

There are two key objectives:

Recovery time objective (RTO): The maximum time range between service collapse and service restoration. It represents how quickly the service could be restarted.
Recovery point objective (RPO): The maximum time range between data being last backed up and the disaster happening. It represents how much loss of data is acceptable.


![dr-objectives](https://raw.githubusercontent.com/ZintrulCre/warehouse/master/resources/disaster-recovery/dr-objectives.png)


You can observe from the figure above that the lower RTO and RPO are, the less recovery time and less loss of data could be. But in the meanwhile, lower RTO and RPO also take more resources, e.g, redundancy, money and operational complexity. Therefore you must decide on the appropriate RTO and RPO values that suites the best for your services.

## DR strategies

AWS offers four strategies for DR, shown below.


![dr-strategies](https://raw.githubusercontent.com/ZintrulCre/warehouse/master/resources/disaster-recovery/dr-strategies.png)

There is no absolute best strategy, you may need to analyze the business needs, the expenditure, the benefits and the risks of applying each strategy to your business.