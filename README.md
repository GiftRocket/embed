### GiftRocket Embed
-----

Add flexible payouts to your application with a few lines of javascript.

### Overview

The Embed client SDK is the easiest way to add rewards and incentives to your product, while maintaining full control of your user experience.  

Within your application, end-users are presented with a white-labeled interface wherein they can choose to receive funds from among a wide set of options. This payout set includes <b>direct bank deposit (ACH), PayPal, Visa prepaid cards, and popular brand gift cards</b>. This catalog is programmatically configurable through the client SDK.


### Access

You can get started immediately with your integration using our sandbox environment. First, sign up through the [GiftRocket rewards website](https://www.giftrocket.com/rewards/).  Within your account API settings page, you will be able to view your sandbox public key for the Embed client SDK as well as the sandbox private key for the REST API. Once you are ready to move to production, production keys will be published.


### Integration


#### Add the client script to your webpage

```html
<script type="text/javascript" src="https://cdn.giftrocket.com/embed/v1/client.js" />
```

#### Launch the rewards modal

```html
<script type="text/javascript">
  // The sandbox public_key displayed within the GiftRocket dashboard's API settings tab.
  publicKey = "[MY_PUBLIC_KEY]";

  // Use Reward.env.SANDBOX during development
  // vs Reward.env.PRODUCTION when you move to production.
  var reward = Reward(publicKey, Reward.env.SANDBOX);

  // Launch the rewards modal within your webpage.
  // In this case, we trigger the modal on a button click.
  $("button#launchpad").on("click", function() {

  // Each reward requires a unique JWT token.  
  // See the token generation step outlined below.
  var jwt = "[YOUR_UNIQUE_TOKEN]";

  reward.redeem(
    jwt,
    {
      // As the developer, you can either omit the catalog property
      // to allow your recipient to choose among all enabled
      // redemption options (i.e. ACH, Visa prepaid card, merchant gift cards)
      // Or, you can limit their options to a subset of choices.
      // Go to https://www.giftrocket.com/rewards/catalog-ids to view the entire catalog.
      config: {
        catalog: ["A2J05SWPI2QG", "HX4U3DQX6GSA", "ET0ZVETV5ILN", "O7VZ5WQOCUQM"]
      },

      onLoad: function(state) {
        console.log(state);
        console.log("Modal loaded");
      },
      onExit: function() {
        console.log("User exited modal");
      },
      onError: function(err) {
        console.log(JSON.stringify(err))
      },
      onSuccess: function(results) {
        // `results.id` is the unique gift ID within the GiftRocket system.
        // Send it to your backend to associate it with your database record
        // for your reward.
        console.log(results.id);
      }
    });
  });
  </script>
```

### JWT

Each redeem call must include a unique JWT (json web token).  This guarantees idempotence -- only a single gift will ever be created for a given token.  Through the JWT standard (RFC 7519), we can secure this client and prevent abuse.

You should create a JWT in your backend and pass it to the `reward` method as the first paramter.  Our Ruby, Python, and Node client libraries each contain a tokenize function which will create the JWT for you. For all other languages, you can find a JWT library at [https://jwt.io](https://jwt.io). 

In the ruby client, the tokenize call looks like the following:

```ruby
  require 'jwt'

  payload = {
    amount: 50, // Required: an integer for your denomination
    external_id: "[UUID]",  // Required: unique string for your reward as stored in your system
    recipient: {
      name: "[RECIPIENT_NAME]",  // Optional: string
      email: "[RECIPIENT_EMAIL]",  // Optional: string
    }
  }

  // We encrypt the token using our sandbox or production GiftRocket api key.
  token = JWT.encode(
    payload,
    "[GIFTROCKET_SANDBOX_PRIVATE_KEY]",
    'HS256'  // Cryptographically sign with HS256 - HMAC using SHA-256 hash algorithm
  )
```

### Create vs. Retrieve Reward

Each JWT should be uniquely associated with a single reward in your system. For a fresh JWT which has not yet been redeemed, the embed client `reward` call will initiate a new redemption flow.

When an existing JWT is detected (a completed redemption already exists for this token), the embed client will instead open the redemption details view on the `reward` call so that your end-user can retrieve their historical information (i.e. what is the gift card code or the bank account to which I sent my funds).


### config

| Property  | Required  | Type        | Description |
|-----------|-----------|-------------|-------------|
| catalog |  No         | `Array`     | By default, the entire GiftRocket rewards catalog will be presented as redemption options to your user.  To constrain this set, pass an array of catalog IDs as the config objects `catalog` property. You can view the entire catalog [here](https://www.giftrocket.com/rewards/catalog-ids).|


#### onLoad

Triggered when the client is successfully mounted.  Passed a single config object to the handler as a parameter.

#### onSuccess

Triggered when the user completes their redemption selection. The object passed to the onSuccess handler will contain an `id` property which is the ID of the reward within the GiftRocket system.  You should pass this ID to your back-end to associate it with your internal UUID for the reward.

#### onError

Triggered on any error within the client.  An error object is passed to the handler as a parameter.

#### onExit

Triggered when the user manually closes the client.


## REST API Integration

At minimum, the embedded SDK does not require any integration with the GiftRocket REST API.  However, an integration may be helpful for fetching historical orders, adding webhooks, etc.  To learn more, [(view the REST documentation)](https://www.giftrocket.com/docs).
