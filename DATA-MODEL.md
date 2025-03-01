
# Data Model

When integrating multiple systems together, the Data Model, i.e. how different objects from different systems are mapped to each other, is super important.

For the purpose of this project, the data model is based on assumptions below. This is just one approach, if your data model needs are different, you may need to adjust some of the code.

## Moesif

- Company
  - A company represent an organization that may have many users, and it may have one more subscription.
  - Since company object owns subscriptions in Moesif, company_id is mapped to Stripe's customer id.
- User
  - This represents single user.
  - For this project, we assume there is only one user per company, (if you may have a concept of users for each company in your system, then you may adjust the mapping as needed.).
  - Because of above assumption, right now, company_id is also mapped to Stripe's customer id.
- Subscription
  - The subscription is associated with the Company. In Moesif, the subscription objects is usually synced from Billing Providers.
- Plans
  - Same definition as Stripe, and synced with billing provider
- Price
  - Same definition as Stripe, and synced with billing provider

## Stripe (one of the most popular billing providers)

- Plans
  - Plans define the base price, currency, and billing cycle for recurring purchases of products.
- Price
  - Defines a specific price or rate for a usage.
- Subscription
  - The subscription is associated with a customer. Synced to Moesif Subscription object above.
- Customer
  - A customer can have multiple subscription.
  - For this project, the `stripe_customer_id` mapped to Moesif `company_id`, and Moesif `user_id` because it is an simplification. And most use cases is only one user anyways.
  - If your have use cases where there is multiple "users" that uses same subscription, then you should still evaluation your data mapping.
  - We also ensure there is ONLY one stripe customer per email.
  - While stripe do not enforce unique Customer per email, for this project, we'll assume and enforce one email per stripe customer, see reason below


## Auth/Identity Provider

- User
  - There is concept of user_id in auth provider.
  - Assume each user is uniq by email.
  - For Okta and Auth0, the jwt generated for idToken include the email as part of the claim, which will be used look up corresponding "Stripe Customer".
  - Thus, email is required as part of the idToken or accessToken generated.

## API gateways

- Customer (usually a concept of a customer)
  - That represent one API key. For purposes of this project, the customer id is mapped to `stripe_customer_id` and `moesif_company_id`

## Your Company

Evaluate object mapping and adjust for your use case and your business:

  - You may have different entity mapping needs, for example:
    -  You may have concept of company and users object in your system. And they each may already an id. Sometimes those id are better suited for mapping, and you need to adjust mapping to ids in different systems as needed.
    -  You may decide to use user_id or company_id from identify provider or your system as source of truth. In which case you may need to add the id from identify provider to stripe customer's metadata. And adjust your company_id mapping in your Stripe Plugin configuration for Moesif,
  - It is important to be be aware of the data model in different systems, and how you want to map them together.
  - It is possible that you decide not to use email to look up Stripe Customer, there are other ways of linking entity ids from the Identify Provider to Stripe Customer. For example, you could have a local database that maps between identify provider id to stripe customer id. But it is up to you to implement.

