---
aliases:
    - /en/overview/tasks/traffic-management/region/
description: Dynamically configure same-region/zone priority in Dubbo-Admin
linkTitle: Same Region Priority
title: Same Region/Zone Priority
type: docs
weight: 4
---

To ensure overall high availability of services, we often adopt a strategy of deploying services across multiple availability zones (data centers). Through this redundant/disaster recovery deployment model, when one region fails, we can still ensure the overall availability of services.

When applications are deployed across multiple different data centers/regions, cross-region calls between applications will occur, and cross-region calls will increase response time and affect user experience. Same-region/zone priority means that when an application calls a service, it prioritizes calling service providers in the same data center/region, avoiding network latency caused by cross-region calls, thereby reducing call response time.

## Before You Begin

* [Deploy Shop Mall Project](../#deploying-the-mall-system)
* Deploy and open [Dubbo Admin](../../../reference-manual/architecture/)

## Task Details

Both Detail application and Comment application have dual-region deployment, where Detail v1 and Comment v1 are deployed in Beijing region, and Detail v2 and Comment v2 are deployed in Hangzhou region. To ensure the response speed of service calls, we need to add same-region priority calling rules to ensure that Detail v1 in Beijing region always calls Comment v1 by default, and Detail v2 in Hangzhou region always calls Comment v2.

![region1](/imgs/v3/tasks/region/region1.png)

When services in the same region fail or become unavailable, cross-region calls are allowed.

![region2](/imgs/v3/tasks/region/region2.png)

### Configure `Detail` to Access Same-Region Deployed `Comment` Service

After normally logging into the mall system, the homepage displays product detail information by default. After refreshing the page multiple times, you will find that the product details (description) and comments (comment) options will show multiple different version combinations. Combined with the deployment structure of Detail and Comment above, this indicates that service calls are not following the same-region priority principle.

![region3](/imgs/v3/tasks/region/region3.png)

Therefore, we need to add same-region priority rules to ensure:
* Detail services in `hangzhou` region call Comment services in the same region, i.e., description v1 and comment v1 are always displayed in combination
* Detail services in `beijing` region call Comment services in the same region, i.e., description v2 and comment v2 are always displayed in combination

#### Operation Steps
1. Log in to Dubbo Admin console
2. In the left navigation bar, select [Service Governance] > [Conditional Routing].
3. Click the "Create" button, fill in the service to enable same-region priority such as `org.apache.dubbo.samples.CommentService` and `region identifier` such as `region`.

![Admin Same Region Priority Setting Screenshot](/imgs/v3/tasks/region/region_admin.png)

After same-region priority is enabled, when you try to refresh the product detail page again, you can see that description and comment always maintain v1 or v2 synchronization.

![region4](/imgs/v3/tasks/region/region4.png)

If you take offline all Comment v2 versions deployed in the `hangzhou` region, Detail v2 will automatically cross-region access Comment v1 in the `beijing` region.

#### Rule Details

**Rule key**: `org.apache.dubbo.samples.CommentService`

**Rule body**
```yaml
configVersion: v3.0
enabled: true
force: false
key: org.apache.dubbo.samples.CommentService
conditions:
  - '=> region = $region'
```

Here we use conditional routing, where `region` is the region identifier in our example, which automatically recognizes the region value where the calling party is located. When a request reaches Detail deployed in the `hangzhou` region, the request sent from Detail automatically filters Comment addresses with the `region=hangzhou` identifier in the URL. If a valid address subset is found, the request is sent; if no addresses match the conditions, it is randomly sent to any available regional address.

```yaml
conditions:
  - '=> region = $region'
```

`force: false` is also key, which allows cross-region service calls when there are no valid addresses in the same region.

## Cleanup
To avoid affecting other task effects, delete or disable the same-region traffic rules just configured through Admin.

## Other Considerations

Our example above does not include the complexity of multi-region registry centers. If each region deploys an independent registry center, then address synchronization between multi-regions becomes a problem that needs to be considered. For this scenario, Dubbo also provides same-region priority support through multi-registry & multi-subscription mechanisms. For details, please refer to the [Multi-Registry & Multi-Subscription](/en/overview/mannual/java-sdk/advanced-features-and-usage/service/multi-registry/) related documentation.