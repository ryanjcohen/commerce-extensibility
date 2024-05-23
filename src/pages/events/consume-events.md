---
title: Consume Events
description: Learn how to consume events sent from Commerce to Adobe I/O Events.
keywords:
  - Events
  - Extensibility
---

Events sent from Commerce to Adobe I/O events can be consumed in a few different ways. The following options for consuming events can be configured when adding events to your App Builder product and creating an event registration:
* [Using the Journaling API](#using-the-journaling-api)
* [Using a webhook URL](#using-a-webhook-url)
* [Using a runtime action](#using-a-runtime-action)
* [Using Amazon EventBridge](#using-amazon-eventbridge)

## Using the Journaling API
When an Adobe I/O event registration is created, the subscribed events will by default be added to an ordered list, referred to as a journal. These events can be consumed using a unique journaling endpoint URL. For more information on using Journaling to consume events, see the [Introduction to Journaling](https://developer.adobe.com/events/docs/guides/api/journaling_api/)

## Using a webhook URL
When creating or editing an event registration, a webhook URL can be registered. When subscribed Commerce events are sent to Adobe I/O Events, these events will then be forwarded to the specified webhook URL.

## Using a runtime action
An [Adobe I/O Runtime Action](https://developer.adobe.com/runtime/docs/guides/overview/entities/#actions) can be set up to receive Commerce events in an Adobe I/O event registration. Actions can be created from JavaScript functions, as described in [Creating Actions](https://developer.adobe.com/runtime/docs/guides/using/creating_actions/). Within an action, business logic can be executed based on the received event payload and/or an API call can be made back to Adobe Commerce to update data or access additional information.

Below is an example of JavaScript code that could be used to create a runtime action that receives `observer.sales_order_save_after` events. 

```js
const { Core } = require('@adobe/aio-sdk')
const { errorResponse, stringParameters} = require('../utils')
const { getCommerceOauthClient } = require('../oauth1a')
const { sendOrderToErpSystem } = require('../erp')
  
async function main (params) {
  const logger = Core.Logger('main', { level: params.LOG_LEVEL || 'info' })
  
  try {
    event_payload = params.data.value
    if (!event_payload.hasOwnProperty('extension_attributes')) {
      // Fetch extension attributes for order and add to order event payload.
      const oauth = getCommerceOauthClient(
        {
          url: params.COMMERCE_BASE_URL,
          consumerKey: params.COMMERCE_CONSUMER_KEY,
          consumerSecret: params.COMMERCE_CONSUMER_SECRET,
          accessToken: params.COMMERCE_ACCESS_TOKEN,
          accessTokenSecret: params.COMMERCE_ACCESS_TOKEN_SECRET
        },
        logger
      )
      const content = await oauth.get('orders/' + event_payload.entity_id)
      event_payload.extension_attributes = content.extension_attributes
    }

    sendOrderToErpSystem(event_payload)
      
    return {
      statusCode: 200,
      body: event_payload
    }
  } catch (error) {
    // log any server errors
    logger.error(error)
    // return with 500
    return errorResponse(500, 'server error', logger)
  }
}
  
exports.main = main
```

The action first accesses the payload for the `observer.sales_order_save_after` event:
  ```js
  event_payload = params.data.value
  ```

The event payload for this event may not contain the saved order's extension attributes. In that case, the extension attributes could be fetched for the specific order using a Commerce API call. In this example, the `oauth1a` module, as defined in the [adobe-commerce-samples repo](https://github.com/adobe/adobe-commerce-samples/blob/main/sample-extension/actions/oauth1a.js) is used. 
  ```js
    const oauth = getCommerceOauthClient(
      {
        url: params.COMMERCE_BASE_URL,
        consumerKey: params.COMMERCE_CONSUMER_KEY,
        consumerSecret: params.COMMERCE_CONSUMER_SECRET,
        accessToken: params.COMMERCE_ACCESS_TOKEN,
        accessTokenSecret: params.COMMERCE_ACCESS_TOKEN_SECRET
      },
      logger
    )
    const content = await oauth.get('orders/' + event_payload.entity_id)
  ```

## Using Amazon EventBridge
An Adobe I/O event registration can be configured to forward received Commerce events to Amazon EventBridge. See [Adobe I/O Events and Amazon EventBridge Integration](https://developer.adobe.com/events/docs/guides/amazon_eventbridge/) for more details.
