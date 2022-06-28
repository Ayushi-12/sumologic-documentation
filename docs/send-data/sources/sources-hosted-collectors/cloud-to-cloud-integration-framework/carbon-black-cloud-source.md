---
id: carbon-black-cloud-source
---

# Carbon Black Cloud Source

The Carbon Black Cloud Source provides a secure endpoint to receive data from the Carbon Black Cloud, Enriched Event Search, and Alerts APIs. It securely stores the required authentication, scheduling, and state tracking information.

:::tip
The Event Forwarder is recommended by VMWare Carbon Black over APIs for obtaining large amounts of data from Carbon Black Cloud in real time. Sumo Logic recommends using the Event Forwarder in combination with a Sumo Logic AWS S3 Source instead of a Carbon Black Cloud Source. 
:::

## Authentication

To grant access to your data you'll need to provide credentials from Carbon Black. There are three APIs this Source collects from and you need to ensure the API Key you provide has permissions to access them. Set your permissions exactly as the following list, do not provide any additional permissions:

* **org.alerts.notes** - create, read
* **org.alerts** - read
* **org.alerts.dismiss** - execute
* **livequery.manage** - create, read
* **org.search.events** - create, read

See the following Carbon Black documents for details on how to
authenticate to each API:

* [Carbon Black Cloud API](https://developer.carbonblack.com/reference/carbon-black-cloud/authentication/)
* [Enriched Events Search API](https://developer.carbonblack.com/reference/carbon-black-cloud/cb-defense/latest/platform-search-api-enriched-events/#authentication)
* [Alerts API](https://developer.carbonblack.com/reference/carbon-black-cloud/platform/latest/alerts-api/)

## States

A Carbon Black Cloud Source tracks errors, reports its health, and start-up progress. You’re informed, in real-time, if the Source is having trouble connecting, if there's an error requiring user action, or if it is healthy and collecting by utilizing Health Events. 

A Carbon Black Cloud Source goes through the following states when created:

1. **Pending**: Once the Source is submitted it is validated, stored, and placed in a **Pending** state.
1. **Started**: A collection task is created on the Hosted Collector.
1. **Initialized**: The task configuration is complete in Sumo Logic.
1. **Authenticated**: The Source successfully authenticated with Carbon Black.
1. **Collecting**: The Source is actively collecting data from Carbon Black.

If the Source has any issues during any one of these states it is placed in an **Error** state.

When you delete the Source it is placed in a **Stopping** state, when it has successfully stopped it is deleted from your Hosted Collector.

On the Collection page, the Health and Status for Sources is displayed. Use Health Events to investigate issues with collection. You can click the text in the Health column, such as **Error**, to open the issue in Health Events to investigate.

![Carbon Black Defense error.png](/img/send-data/Carbon-Black-Defense-error.png)

Hover your mouse over the status icon to view a tooltip with details on the detected issue.

![health error generic.png](/img/send-data/health-error-generic.png)

## Create a Carbon Black Cloud Source

When you create a Carbon Black Cloud Source, you add it to a Hosted Collector. Before creating the Source, identify the Hosted Collector you want to use or create a new Hosted Collector. For instructions, see [Configure a Hosted Collector](../../../configure-hosted-collector.md).

To configure A Carbon Black Cloud Source:

1. In the Sumo Logic web app, select **Manage Data \> Collection \> Collection**. 
1. On the Collectors page, click **Add Source** next to a **Hosted Collector**.
1. Select **Carbon Black Cloud**.

    ![CB Cloud icon.png](/img/send-data/CB-Cloud-icon.png)

1. Enter a **Name**for the Source. The description is optional.

    ![CB Cloud input pane.png](/img/send-data/CB-Cloud-input-pane.png)

1. (Optional) For **Source Category**, enter any string to tag the output collected from the Source. Category metadata is stored in a searchable field called `_sourceCategory`.
1. **Forward to SIEM**. Check the checkbox to forward your data to Cloud SIEM Enterprise. When configured with the **Forward to SIEM** option the following metadata fields are set:

   * `\_siemVendor`: CarbonBlack 
   * `\_siemProduct`: Cloud 
   * `\_siemFormat`: JSON 

1. (Optional) **Fields.** Click the **+Add Field** link to define the fields you want to associate, each field needs a name (key) and value.

   * ![green check circle.png](/img/reuse/green-check-circle.png) A green circle with a check mark is shown when the field exists in the Fields table schema. 
   * ![orange exclamation point.png](/img/reuse/orange-exclamation-point.png) An orange triangle with an exclamation point is shown when the field doesn't exist in the Fields table schema. In this case, an option to automatically add the nonexistent fields to the Fields table schema is provided. If a field is sent to Sumo that does not exist in the Fields schema it is ignored, known as dropped. 

1. **CB Cloud Domain**. Enter your Carbon Black Cloud domain, such as, `dev-prod05.conferdeploy.net`. See [this knowledge base article](https://community.carbonblack.com/t5/Knowledge-Base/Carbon-Black-Cloud-What-URLs-are-used-to-access-the-APIs/ta-p/67346) to determine which domain to use.

1. **API Key**. Enter the Carbon Black Cloud API Key you want to use to authenticate requests. Ensure the key is granted the required permissions for all the APIs listed in the above [Authentication section](#authentication).

1. **API ID**. Enter your Carbon Black Cloud API ID correlated to your API key.
1. **Org Key**. Enter your Carbon Black Cloud Org key, found in your Carbon Black product console under **Settings \> API Access \> API Keys.**
1. (Optional) The **Polling Interval** is set to 300 seconds by default, you can adjust it based on your needs.
1. When you are finished configuring the Source click **Submit**.

## Error types

When Sumo Logic detects an issue it is tracked by Health Events. The following table shows the three possible error types, the reason the error would occur, if the Source attempts to retry, and the name of the event log in the Health Event Index.

| Type | Reason | Retries | Retry Behavior | Health Event Name |
|--|--|--|--|--|
| ThirdPartyConfig  | Normally due to an invalid configuration. You'll need to review your Source configuration and make an update. | No retries are attempted until the Source is updated. | Not applicable | ThirdPartyConfigError  |
| ThirdPartyGeneric | Normally due to an error communicating with the third party service APIs. | Yes | The Source will retry for up to 90 minutes, after which it quits. | ThirdPartyGenericError |
| FirstPartyGeneric | Normally due to an error communicating with the internal Sumo Logic APIs. | Yes | The Source will retry for up to 90 minutes, after which it quits. | FirstPartyGenericError |

#### JSON configuration

Sources can be configured using UTF-8 encoded JSON files with the Collector Management API. See [how to use JSON to configure Sources](/docs/send-data/sources/use-json-configure-sources) for details. 

| Parameter | Type | Required? | Description | Access |
|--|--|--|--|--|
| config | JSON Object  | Yes | Contains the configuration parameters for the Source. |   |
| schemaRef | JSON Object  | Yes | Use `{"type":"Carbon Black Cloud"}` for a Carbon Black Cloud Source. | not modifiable |
| sourceType | String | Yes | Use `Universal` for a Carbon Black Cloud Source. | not modifiable |

The following table shows the **config** parameters for a Carbon Black
Cloud Source.

| Parameter | Type | Required |  Description | Access |
|--|--|--|--|--|
| `name` | String | Yes	 | Type a desired name of the Source. The name must be unique per Collector. This value is assigned to the [metadata](../../../../search/get-started-with-search/search-basics/built-in-metadata.md) field `_source`. | modifiable |
| `description` | String | No |  Type a description of the Source. | modifiable |
| `category` | String | No |  Type a category of the source. This value is assigned to the metadata field `_sourceCategory`. See [best practices](../../../design-deployment/best-practices-source-categories.md) for details. | modifiable |
| `fields` | JSON Object | No |  JSON map of key-value fields (metadata) to apply to the Collector or Source. Use the boolean field `_siemForward` to enable forwarding to SIEM. | modifiable |
| `domain` | String | Yes | Enter your Carbon Black Cloud domain, such as, `dev-prod05.conferdeploy.net`. See this [knowledge base article](https://community.carbonblack.com/t5/Knowledge-Base/Carbon-Black-Cloud-What-URLs-are-used-to-access-the-APIs/ta-p/67346) to determine which domain to use. | modifiable |
| `api_key` | String | Yes | The Carbon Black Cloud API Key you want to use to authenticate requests. Ensure the key is granted the required permissions for all the APIs listed in the above [Authentication section](#authentication). | modifiable |
| `api_id` | String | Yes | The Carbon Black Cloud API ID correlated to your API key. | modifiable |
| `org_key` | String | Yes | Your Carbon Black Cloud Org key, found in your Carbon Black product console under Settings > API Access > API Keys. | modifiable |
| `pollingInterval` | Integer | No | This sets how many seconds the Source checks for new data. The default is 60 seconds. | modifiable |

Carbon Black Cloud Source JSON example:

```json
{
  "api.version":"v1",
  "source":{
    "config":{
      "api_id":"********",
      "name":"CB Cloud",
      "domain":"defense.conferdeploy.net",
      "org_key":"ABCDEFG1",
      "polling_interval":300,
      "api_key":"********",
      "fields":{
        "_siemForward":false
      },
      "category":"c2c/cb_cloud"
    },
    "schemaRef":{
      "type":"Carbon Black Cloud"
    },
    "state":{
      "state":"Collecting"
    },
    "sourceType":"Universal"
  }
}
```