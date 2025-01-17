# How APIM supports monetization

[Azure API Management (APIM)](https://docs.microsoft.com/en-us/azure/api-management/) has a range of built-in capabilities that will enable you to de-risk implementation, accelerate project timescales and scale your APIs with confidence.

This document **focuses on the APIM features that will enable your monetization strategy to be implemented**.  It highlights how you can use APIM to provide consumers with a frictionless experience to discover your public APIs, enter payment details, activate their subscription, consume the API, monitor usage and automatically pay for their usage of the API.

The diagram below introduces the key features in APIM that enable this solution:

![](architecture-overview.png)

The remainder of this document describes these features in more detail:

## API Discovery

Use APIM's built-in developer portal (available "out of the box") to launch your API and onboard API consumers.  Place emphasis on developing good quality content for the developer portal that will enable API consumers to explore and use your APIs with as little friction as possible.  Test the content with real API developers to check that the information provided is easy to fine, accurate, complete and intuitive.

For details about how to add content and control the branding of the developer portal, see the [Overview of the developer portal](https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-developer-portal)

## API Packaging

APIM uses the concept of products and policies to manage how your APIs are packaged up and presented to users.

### Products

The APIs are published [via products](https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-add-products). A product allows you to define which APIs a subscriber can access, and specific throttling [policies](https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-policies) (to e.g. limit a specific subscription to a certain quota of calls per month).

When an API consumer subscribes to a product, they are provided with an API key which they can use to make calls. However, the subscription is initially set to a `submitted` state, and needs to be activated before they will be able to use the APIs.

Configure the APIM products to package your underlying API to mirror your revenue model, with a one-to-one relationship between each tier in your revenue model and a corresponding APIM product.

In the sample projects, APIM products are used as the top level means of codifying the monetization strategy.  The APIM products are set up to mirror the tiers in the revenue model and are used to index the specific pricing model that should be applied to each tier.  This enables a flexible, configuration driven approach to setting up the monetization strategy.

### Policies

Apply APIM policies to control the quality of service for each product.  The sample projects use two specific policy features to control quality of service in line with the revenue model:

- Quota - defines the total number of calls the user can make to the API over a specified time period.  For example "100 calls per month".  Once the quota is reached, the calls to the API will fail with the caller receiving a `403 Forbidden` response status code;
- Rate limit - defines the number of calls over a sliding time window that can be made to the API.  For example "200 calls per minute".  This is designed to prevent spikes in API usage by the API consumer beyond the quality of service that they are paying for with the chosen product.  When the call rate is exceeded, the caller receives a `429 Too Many Requests` response status code.

For more details about policies, please refer to the [Policies in Azure API Management](https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-policies) documentation.

## API Consumption

Use API subscriptions to enable API consumers to gain access to your APIs (via the products).

APIM subscriptions are established when an API consumer chooses to sign up to use a specific APIM product.  Use APIM delegation to ingrate the subscription process with the payment provider.  Having successfully provided payment details, the user can then handed back to APIM where a unique security key for the subscription is generated that will enable them to gain access to the API.

For more information about subscriptions, please refer to the [Subscriptions in Azure API Management](https://docs.microsoft.com/en-us/azure/api-management/api-management-subscriptions) documentation.

## API usage monitoring

Use APIM's built-in analytics to gain insights about the usage and performance of your APIs.  This provides reports by API, geography, API operations, product, request, subscription, time, or user.

Review these reports regularly to understand how your monetization strategy is being adopted by API consumers.

See [Get API analytics in Azure API Management](https://docs.microsoft.com/en-us/azure/api-management/howto-use-analytics) for more details.

## Security

Use APIM's products, API policies and subscriptions to control the level of access that users should gain to each product.  In the sample projects, to protect against misuse and abuse, access to the API is only granted via a subscription if the user has successfully authenticated with the payment provider, even if the specific API product is being offered for free.

## Integration

Create a seamless monetization experience for end users through both front end and back end integration between APIM and your chosen payment provider.  Use APIM delegation for front end integration and the REST API for back end integration.

### Delegation

The sample projects use [APIM delegation](https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-setup-delegation) to allow custom integrations to be made with the third-party payment providers.

The demo uses delegation for both the sign up/sign in and product subscription experiences.

For sign up/sign in, it implements the following workflow:

1. Developer clicks on the sign in or sign up link at the API Management developer portal
2. Browser is redirected to the delegation endpoint - which is configured to be a page in the custom billing portal app.
3. Custom app presents a UI asking user to sign in or sign up
4. On success, the user is redirected back to the API Management developer portal page they started from, authenticated.

For the product subscription delegation, it implements the following workflow:

1. Developer selects a product in the API Management developer portal and clicks on the Subscribe button
1. Browser is redirected to the delegation endpoint - which is configured to be a page in the custom billing portal app.
1. Custom app presents a UI which is dependent on the payment provider configured (Stripe or Adyen) and be taken through the relevant checkout process.
2. On success, the user is redirected back to the API Management product page they started from.  The product will be active and the API keys will be available

### REST API

Use the REST API for APIM to automate the operation of your monetization strategy.

The sample projects use the API to programmatically:

- Retrieve APIM products and policies to enable synchronised configuration of similar concepts in payment providers such as Stripe;
- Poll APIM on a regular basis to retrieve API usage metrics for each subscription, and to use this to drive the billing process.

See [Azure API Management](https://docs.microsoft.com/en-us/rest/api/apimanagement/) for an overview of the REST API operations that are available.

## DevOps

Use ARM to version control and automated deployment changes to APIM.  This approach should include the configuration of the APIM features that implement your monetization strategy such as products, policies and the developer portal.

In the sample projects, the ARM scripts are augmented by a JSON file which defines the pricing model for each of the APIM products.  This enables the configuration between APIM and the chosen payment provider to be synchronised.  This approach means that the entire solution is managed under a single source control repository, such that all changes associated with the ongoing evolution of the monetization strategy can be coordinated as a single release and carried out in accordance with governance & auditing requirements.

## Initialisation and deployment

APIM can be deployed either through the [Azure portal](https://portal.azure.com/) user interface or through a "infrastructure as code" approach using [ARM templates(https://azure.microsoft.com/en-gb/services/arm-templates/)].  This repository uses the ARM templates to deploy APIM - see [Deploy demo with Stripe](./documentation/stripe-deploy.md) or [Deploy demo with Adyen](./documentation/adyen-deploy.md) to run this process.
