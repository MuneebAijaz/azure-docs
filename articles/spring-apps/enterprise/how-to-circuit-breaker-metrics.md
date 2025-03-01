---
title: Collect Spring Cloud Resilience4J Circuit Breaker Metrics with Micrometer
description: How to collect Spring Cloud Resilience4J Circuit Breaker Metrics with Micrometer in Azure Spring Apps.
author: KarlErickson
ms.author: karler
ms.service: spring-apps
ms.topic: how-to
ms.date: 12/15/2020
ms.custom: devx-track-java, devx-track-extended-java, devx-track-azurecli
zone_pivot_groups: spring-apps-tier-selection
---

# Collect Spring Cloud Resilience4J Circuit Breaker Metrics with Micrometer (Preview)

> [!NOTE]
> Azure Spring Apps is the new name for the Azure Spring Cloud service. Although the service has a new name, you'll see the old name in some places for a while as we work to update assets such as screenshots, videos, and diagrams.

**This article applies to:** ✔️ Basic/Standard ✔️ Enterprise

This article shows you how to collect Spring Cloud Resilience4j Circuit Breaker Metrics with Application Insights Java in-process agent. With this feature, you can monitor the metrics of Resilience4j circuit breaker from Application Insights with Micrometer.

The demo [spring-cloud-circuit-breaker-demo](https://github.com/spring-cloud-samples/spring-cloud-circuitbreaker-demo) shows how the monitoring works.

## Prerequisites

* Install Git, Maven, and Java, if not already installed on the development computer.

## Build and deploy apps

Use the following steps to build and deploy the sample applications.

1. Use the following command to clone and build the demo repository:

   ```bash
   git clone https://github.com/spring-cloud-samples/spring-cloud-circuitbreaker-demo.git
   cd spring-cloud-circuitbreaker-demo && mvn clean package -DskipTests
   ```

::: zone pivot="sc-standard"

1. Use the following command to create an Azure Spring Apps service instance:


   ```azurecli
   az spring create \
       --resource-group ${resource-group-name} \
       --name ${Azure-Spring-Apps-instance-name}
   ```

1. Use the following commands to create the applications with endpoints:


   ```azurecli
   az spring app create \
       --resource-group ${resource-group-name} \
       --service ${Azure-Spring-Apps-instance-name} \
       --name resilience4j \
       --assign-endpoint
   az spring app create \
       --resource-group ${resource-group-name} \
       --service ${Azure-Spring-Apps-instance-name} \
       --name reactive-resilience4j \
       --assign-endpoint
   ```

1. Use the following commands to deploy the applications:


   ```azurecli
   az spring app deploy \
       --resource-group ${resource-group-name} \
       --service ${Azure-Spring-Apps-instance-name} \
       --name resilience4j \
       --env resilience4j.circuitbreaker.instances.backendA.registerHealthIndicator=true \
       --artifact-path ./spring-cloud-circuitbreaker-demo-resilience4j/target/spring-cloud-circuitbreaker-demo-resilience4j-0.0.1-SNAPSHOT.jar
   az spring app deploy \
       --resource-group ${resource-group-name} \
       --service ${Azure-Spring-Apps-instance-name} \
       --name reactive-resilience4j \
       --env resilience4j.circuitbreaker.instances.backendA.registerHealthIndicator=true \
       --artifact-path ./spring-cloud-circuitbreaker-demo-reactive-resilience4j/target/spring-cloud-circuitbreaker-demo-reactive-resilience4j-0.0.1-SNAPSHOT.jar
   ```

::: zone-end

::: zone pivot="sc-enterprise"

1. Use the following command to create an Azure Spring Apps service instance:


   > [!NOTE]
   > If your subscription has never been used to create an Enterprise plan instance of Azure Spring Apps, you must run the following command:

   >
   > ```azurecli
   > az term accept \
   >     --publisher vmware-inc 
   >     --product azure-spring-cloud-vmware-tanzu-2 
   >     --plan asa-ent-hr-mtr
   > ```

   ```azurecli
   az spring create \
       --resource-group ${resource-group-name} \
       --name ${Azure-Spring-Apps-instance-name} \
       --sku Enterprise
   ```

1. Use the following commands to create applications with endpoints:

   ```azurecli
   az spring app create \
       --resource-group ${resource-group-name} \
       --service ${Azure-Spring-Apps-instance-name} \
       --name resilience4j \
       --assign-endpoint
   az spring app create \
       --resource-group ${resource-group-name} \
       --service ${Azure-Spring-Apps-instance-name} \
       --name reactive-resilience4j \
       --assign-endpoint
   ```

1. Use the following commands to deploy the applications:


   ```azurecli
   az spring app deploy \
       --resource-group ${resource-group-name} \
       --service ${Azure-Spring-Apps-instance-name} \
       --name resilience4j \
       --env resilience4j.circuitbreaker.instances.backendA.registerHealthIndicator=true \
       --artifact-path ./spring-cloud-circuitbreaker-demo-resilience4j/target/spring-cloud-circuitbreaker-demo-resilience4j-0.0.1-SNAPSHOT.jar
   az spring app deploy \
       --resource-group ${resource-group-name} \
       --service ${Azure-Spring-Apps-instance-name} \
       --name reactive-resilience4j \
       --env resilience4j.circuitbreaker.instances.backendA.registerHealthIndicator=true \
       --artifact-path ./spring-cloud-circuitbreaker-demo-reactive-resilience4j/target/spring-cloud-circuitbreaker-demo-reactive-resilience4j-0.0.1-SNAPSHOT.jar
   ```

::: zone-end

> [!NOTE]
>
> * Include the required dependency for Resilience4j:
>
>   ```xml
>   <dependency>
>       <groupId>io.github.resilience4j</groupId>
>       <artifactId>resilience4j-micrometer</artifactId>
>   </dependency>
>   <dependency>
>       <groupId>org.springframework.cloud</groupId>
>       <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
>   </dependency>
>   ```
>
> * Your code must use the `CircuitBreakerFactory` API, which is implemented as a `bean` automatically created when you include a Spring Cloud Circuit Breaker starter. For more information, see [Spring Cloud Circuit Breaker](https://spring.io/projects/spring-cloud-circuitbreaker#overview).
>
> * The following two dependencies have conflicts with Resilient4j packages.  Be sure you don't include them.
>
>   ```xml
>   <dependency>
>       <groupId>org.springframework.cloud</groupId>
>       <artifactId>spring-cloud-starter-sleuth</artifactId>
>   </dependency>
>   <dependency>
>       <groupId>org.springframework.cloud</groupId>
>       <artifactId>spring-cloud-starter-zipkin</artifactId>
>   </dependency>
>   ```
>
>
> Navigate to the URL provided by gateway applications, and access the endpoint from [spring-cloud-circuit-breaker-demo](https://github.com/spring-cloud-samples/spring-cloud-circuitbreaker-demo) as follows:
>
>   ```console
>   /get
>   /delay/{seconds}
>   /fluxdelay/{seconds}
>   ```

## Locate Resilence4j Metrics on the Azure portal

::: zone pivot="sc-standard"

1. In your Azure Spring Apps instance, select **Application Insights** in the navigation pane and then select **Application Insights** on the page.

   :::image type="content" source="media/how-to-circuit-breaker-metrics/application-insights.png" alt-text="Screenshot of the Azure portal that shows the Azure Spring Apps Application Insights page with Application Insights highlighted." lightbox="media/how-to-circuit-breaker-metrics/application-insights.png":::


   > [!NOTE]
   > If you don't enable Application Insights, you can enable the Java In-Process agent. For more information, see the [Manage Application Insights using the Azure portal](./how-to-application-insights.md#manage-application-insights-using-the-azure-portal) section of [Use Application Insights Java In-Process Agent in Azure Spring Apps](./how-to-application-insights.md).

1. Enable dimension collection for resilience4j metrics. For more information, see the [Custom metrics dimensions and pre-aggregation](../../azure-monitor/app/pre-aggregated-metrics-log-metrics.md#custom-metrics-dimensions-and-pre-aggregation) section of [Log-based and pre-aggregated metrics in Application Insights](../../azure-monitor/app/pre-aggregated-metrics-log-metrics.md).

1. Select **Metrics** in the navigation pane. The **Metrics** page provides dropdown menus and options to define the charts in this procedure. For all charts, set **Metric Namespace** to **azure.applicationinsights**.

   :::image type="content" source="media/how-to-circuit-breaker-metrics/chart-menus.png" alt-text="Screenshot of the Azure portal Application Insights Metrics page, with Metrics highlighted in the navigation pane, and with azure-applicationinsights highlighted in the Metric Namespace dropdown menu." lightbox="media/how-to-circuit-breaker-metrics/chart-menus.png":::

1. Set **Metric** to **resilience4j_circuitbreaker_buffered_calls**, and then set **Aggregation** to **Avg**.

   :::image type="content" source="media/how-to-circuit-breaker-metrics/buffered-calls.png" alt-text="Screenshot of the Azure portal Application Insights Metrics page that shows a chart with Metric set to circuit breaker buffered calls and Aggregation set to Average." lightbox="media/how-to-circuit-breaker-metrics/buffered-calls.png":::

1. Set **Metric** to **resilience4j_circuitbreaker_calls**, and then set **Aggregation** to **Avg**.

   :::image type="content" source="media/how-to-circuit-breaker-metrics/calls.png" alt-text="Screenshot of the Azure portal Application Insights Metrics page that shows a chart with Metric set to circuit breaker calls and Aggregation set to Average." lightbox="media/how-to-circuit-breaker-metrics/calls.png":::

1. Set **Metric** to **resilience4j_circuitbreaker_calls**, and then set **Aggregation** to **Avg**. Select **Add filter** and set **Name** to **Delay**.

   :::image type="content" source="media/how-to-circuit-breaker-metrics/calls-filter.png" alt-text="Screenshot of the Azure portal Application Insights Metrics page that shows a chart with Metric set to circuit breaker calls and Aggregation set to Average, and with Filter set to the name Delay." lightbox="media/how-to-circuit-breaker-metrics/calls-filter.png":::

1. Set **Metric** to **resilience4j_circuitbreaker_calls**, and then set **Aggregation** to **Avg**. Select **Apply splitting** and set **Split by** to **kind**.

   :::image type="content" source="media/how-to-circuit-breaker-metrics/calls-splitting.png" alt-text="Screenshot of the Azure portal Application Insights Metrics page that shows a chart with Metric set to circuit breaker calls and Aggregation set to Average, and with Apply splitting selected with Split by set to kind." lightbox="media/how-to-circuit-breaker-metrics/calls-splitting.png":::

1. Set **Metric** to **resilience4j_circuitbreaker_calls**, and then set **Aggregation** to **Avg**. Select **Add metric** and set **Metric** to **resilience4j_circuitbreaker_buffered_calls**, and then set **Aggregation** to **Avg**. Select **Add metric** again and set **Metric** to **resilience4j_circuitbreaker_slow_calls**, and then set **Aggregation** set to **Avg**.

   :::image type="content" source="media/how-to-circuit-breaker-metrics/slow-calls.png" alt-text="Screenshot of the Azure portal that shows the Application Insights Metrics page with the chart described in this step." lightbox="media/how-to-circuit-breaker-metrics/slow-calls.png":::


::: zone-end

::: zone pivot="sc-enterprise"

1. In your Azure Spring Apps instance, select **Application Insights** in the navigation pane and then select the default **Application Insights** on the page.

   :::image type="content" source="media/how-to-circuit-breaker-metrics/application-insights-enterprise.png" alt-text="Screenshot of the Azure portal that shows the Azure Spring Apps Enterprise Application Insights page with the Application Insights on the button bar highlighted." lightbox="media/how-to-circuit-breaker-metrics/application-insights-enterprise.png":::

   > [!NOTE]
   > If there's no default Application Insights available, you can enable the Java In-Process agent. For more information, see the [Manage Application Insights using the Azure portal](./how-to-application-insights.md#manage-application-insights-using-the-azure-portal) section of [Use Application Insights Java In-Process Agent in Azure Spring Apps](./how-to-application-insights.md).

1. Enable dimension collection for resilience4j metrics. For more information, see the [Custom metrics dimensions and pre-aggregation](../../azure-monitor/app/pre-aggregated-metrics-log-metrics.md#custom-metrics-dimensions-and-pre-aggregation) section of [Log-based and pre-aggregated metrics in Application Insights](../../azure-monitor/app/pre-aggregated-metrics-log-metrics.md).

1. Select **Metrics** in the navigation pane. The **Metrics** page provides dropdown menus and options to define the charts in this procedure. For all charts, set **Metric Namespace** to **azure.applicationinsights**.

   :::image type="content" source="media/how-to-circuit-breaker-metrics/chart-menus.png" alt-text="Screenshot of the Azure portal Application Insights Metrics page, with Metrics highlighted in the navigation pane, and with azure-applicationinsights highlighted in the Metric Namespace dropdown menu." lightbox="media/how-to-circuit-breaker-metrics/chart-menus.png":::

1. Set **Metric** to **resilience4j_circuitbreaker_buffered_calls**, and then set **Aggregation** to **Avg**.

   :::image type="content" source="media/how-to-circuit-breaker-metrics/buffered-calls.png" alt-text="Screenshot of the Azure portal Application Insights Metrics page that shows a chart with Metric set to circuit breaker buffered calls and Aggregation set to Average." lightbox="media/how-to-circuit-breaker-metrics/buffered-calls.png":::

1. Set **Metric** to **resilience4j_circuitbreaker_calls**, and then set **Aggregation** to **Avg**.

   :::image type="content" source="media/how-to-circuit-breaker-metrics/calls.png" alt-text="Screenshot of the Azure portal Application Insights Metrics page that shows a chart with Metric set to circuit breaker calls and Aggregation set to Average." lightbox="media/how-to-circuit-breaker-metrics/calls.png":::

1. Set **Metric** to **resilience4j_circuitbreaker_calls**, and then set **Aggregation** to **Avg**. Select **Add filter** and set **Name** to **Delay**.

   :::image type="content" source="media/how-to-circuit-breaker-metrics/calls-filter.png" alt-text="Screenshot of the Azure portal Application Insights Metrics page that shows a chart with Metric set to circuit breaker calls and Aggregation set to Average, and with Filter set to the name Delay." lightbox="media/how-to-circuit-breaker-metrics/calls-filter.png":::

1. Set **Metric** to **resilience4j_circuitbreaker_calls**, and then set **Aggregation** to **Avg**. Select **Apply splitting** and set **Split by** to **kind**.

   :::image type="content" source="media/how-to-circuit-breaker-metrics/calls-splitting.png" alt-text="Screenshot of the Azure portal Application Insights Metrics page that shows a chart with Metric set to circuit breaker calls and Aggregation set to Average, and with Apply splitting selected with Split by set to kind." lightbox="media/how-to-circuit-breaker-metrics/calls-splitting.png":::

1. Set **Metric** to **resilience4j_circuitbreaker_calls**, and then set **Aggregation** to **Avg**. Select **Add metric** and set **Metric** to **resilience4j_circuitbreaker_buffered_calls**, and then set **Aggregation** to **Avg**. Select **Add metric** again and set **Metric** to **resilience4j_circuitbreaker_slow_calls**, and then set **Aggregation** set to **Avg**.

   :::image type="content" source="media/how-to-circuit-breaker-metrics/slow-calls.png" alt-text="Screenshot of the Azure portal that shows the Application Insights Metrics page with the chart described in this step." lightbox="media/how-to-circuit-breaker-metrics/slow-calls.png":::

::: zone-end

## Next steps

* [Application insights](./how-to-application-insights.md)
* [Circuit breaker dashboard](./tutorial-circuit-breaker.md)
