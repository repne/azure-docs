---
title: Import a SOAP API and convert to REST using the Azure portal | Microsoft Docs
description: Learn how to import a SOAP API, convert it to REST with API Management, and then test the API in the Azure and Developer portals.
services: api-management
documentationcenter: ''
author: dlepow
manager: cfowler
editor: ''

ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: tutorial
ms.date: 11/22/2017
ms.author: danlep

---
# Import a SOAP API and convert to REST

This article shows how to import a SOAP API and convert it to REST. The article also shows how to test the APIM API.

In this article, you learn how to:

> [!div class="checklist"]
> * Import a SOAP API and convert to REST
> * Test the API in the Azure portal
> * Test the API in the Developer portal

## Prerequisites

Complete the following quickstart: [Create an Azure API Management instance](get-started-create-service-instance.md)

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="create-api"> </a>Import and publish a back-end API

1. Select **APIs** from under **API MANAGEMENT**.
2. Select **WSDL** from the **Add a new API** list.

    ![SOAP API](./media/restify-soap-api/wsdl-api.png)
3. In the **WSDL specification**, enter the URL to where your SOAP API resides.
4. Click **SOAP to REST** radio button. When this option is clicked, APIM attempts to make an automatic transformation between XML and JSON. In this case consumers should be calling the API as a restful API, which returns JSON. APIM is converting each request into a SOAP call.

    ![SOAP to REST](./media/restify-soap-api/soap-to-rest.png)

5. Press tab.

    The following fields get filled up with the info from the SOAP API: Display name, Name, Description.
6. Add an API URL suffix. The suffix is a name that identifies this specific API in this APIM instance. It has to be unique in this APIM instance.
9. Publish the API by associating the API with a product. In this case, the "*Unlimited*" product is used.  If you want for the API to be published and be available to developers, add it to a product. You can do it during API creation or set it later.

    Products are associations of one or more APIs. You can include a number of APIs and offer them to developers through the developer portal. Developers must first subscribe to a product to get access to the API. When they subscribe, they get a subscription key that is good for any API in that product. If you created the APIM instance, you are an administrator already, so you are subscribed to every product by default.

    By default, each API Management instance comes with two sample products:

    * **Starter**
    * **Unlimited**   
10. Select **Create**.

## Test the new API in the Azure portal

Operations can be called directly from the Azure portal, which provides a convenient way to view and test the operations of an API.  

1. Select the API you created in the previous step.
2. Press the **Test** tab.
3. Select some operation.

    The page displays fields for query parameters and fields for the headers. One of the headers is "Ocp-Apim-Subscription-Key", for the subscription key of the product that is associated with this API. If you created the APIM instance, you are an administrator already, so the key is filled in automatically. 
1. Press **Send**.

    Backend responds with **200 OK** and some data.

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-append-apis.md)]

[!INCLUDE [api-management-define-api-topics.md](../../includes/api-management-define-api-topics.md)]

## Next steps

> [!div class="nextstepaction"]
> [Transform and protect a published API](transform-api.md)