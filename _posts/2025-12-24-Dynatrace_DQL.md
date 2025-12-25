---
layout: post
title: "Dynatrace Query Language"
date: 2025-12-24 21:00:00 -07:00
categories: dynatrace
tags:
- dynatrace
- dql
---

Dynatrace Query Language snippets

## Describes schema for dt.entity.service

```sql
describe dt.entity.service
```

|field|data_types|
|-|-|
|agentTechnologyType|string|
|akkaActorSystem|string|
|applicationBuildVersion|array|
|applicationEnvironment|array|
|applicationName|array|
|applicationReleaseVersion|array|
|awsNameTag|string|
|belongs_to|record|
|boshName|string|
|called_by|record|
|calls|record|
|className|string|
|cloudDatabaseProvider|string|
|clustered_by|record|
|contains|record|
|contextRoot|string|
|customIconPath|string|
|databaseHostNames|array|
|databaseName|string|
|databaseVendor|string|
|dt.security_context|array|
|dt.system.environment|string|
|dt.system.table|string|
|entity.conditional_name|string|
|entity.customized_name|string|
|entity.detected_name|string|
|entity.name|string|
|entity.type|string|
|esbApplicationName|string|
|fallbackServiceType|string|
|gcpZone|string|
|groups|record|
|ibmCtgGatewayUrl|string|
|ibmCtgServerName|string|
|icon|record|
|id|string|
|indirectly_receives_from|record|
|indirectly_sends_to|record|
|instantiates|record|
|ipAddress|array|
|isExternalService|boolean|
|lifetime|timeframe|
|managementZones|array|
|matchedServiceDetectionV2Rules|array|
|oneAgentCustomHostName|string|
|path|string|
|port|long|
|publicCloudId|string|
|publicCloudRegion|string|
|publicDomainName|string|
|receives_from|record|
|remoteCrossEnvironments|array|
|remoteEndpoint|string|
|remoteServiceName|string|
|runs_on|record|
|sends_to|record|
|serviceDetectionAttributes|record|
|serviceTechnologyTypes|array|
|serviceType|string|
|softwareTechnologies|array|
|tags|array|
|unifiedServiceIndicators|array|
|webApplicationId|string|
|webServerName|string|
|webServiceName|string|
|webServiceNamespace|string|

## Get Service ID, Name, and Type

```sql
fetch dt.entity.service
| fields id, entity.name, entity.type, serviceType, serviceTechnologyTypes, softwareTechnologies, applicationName, webServerName
```

Filtered by a string value in the service name

```sql
fetch dt.entity.service
| fields id, entity.name, entity.type, serviceType, serviceTechnologyTypes, softwareTechnologies, webServerName
| filter matchesPhrase(entity.name, "arsys")
```

Filtered by service type and technology types IIS and Java. Returns computer's hostname that service runs on

```sql
fetch dt.entity.service
| filter serviceType == "WEB_REQUEST_SERVICE" and in(serviceTechnologyTypes, array("IIS","Java"))
| expand dt.entity.host=runs_on[dt.entity.host]
| fields entityName(dt.entity.host),  entity.name, webServerName, id, serviceTechnologyTypes
```

## References

[DQL language reference](https://docs.dynatrace.com/docs/discover-dynatrace/platform/grail/dynatrace-query-language/dql-reference)