### GiftRocket Embed
-----

Add flexible payouts to your application with a few lines of javascript.

### Overview

The GiftRocket Embed client SDK is the easiest way to add rewards and incentives to your product, while maintaining full control of your user experience.  Within your application, end-users are presented with a white-labeled UI.

Through this module, they can choose to receive funds from among a wide set of options, including <b>direct bank deposit (ACH), PayPal, Visa prepaid cards, and popular brand gift cards</b>. This set of choices is programmatically configurable through the client SDK.


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
  // Available within the GiftRocket dashboard's API settings tab.
  publicKey = "[MY_PUBLIC_KEY]";

  // Use Reward.env.SANDBOX during development
  // vs Reward.env.PRODUCTION when you move to production.
  var env = Reward.env.SANDBOX;
  var reward = Reward(publicKey, env);

  // Launch the rewards modal within your webpage.
  // In this case, we trigger the modal on a button click.
  $("button#launchpad").on("click", function() {

  // Each reward requires a unique externalId which should map
  // to a UUID within your application for a given gift.
  var externalId = "[UUID]";

  reward.redeem(
    externalId,
    {
      gift: {
        amount: 30,
        // If no recipient information is passed as configuration
        // the modal will require that the user input their
        // name and email address in the checkout flow.
        recipient: {
          email: "my_user@application.com",
          name: "User Name"
        },
        // As the developer, you can either omit the catalog property
        // to allow your recipient to choose among all enabled
        // redemption options (i.e. ACH, Visa prepaid card, merchant gift cards)
        // Or, you can limit their options to a subset of choices.
        catalog: ["A2J05SWPI2QG", "HX4U3DQX6GSA", "ET0ZVETV5ILN", "O7VZ5WQOCUQM", "KV934TZ93NQM"]
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
      onSuccess: function(gift) {
        console.log(gift.externalId);
        // Send this ID to your backend to approve the gift
        // via the rest API.  Once approved, the transaction will
        // executed and the user will receive their reward.
      }
    });
  });
  </script>
```

## Reward.redeem Parameters

#### ExternalId

Each redeem call must include a unique externalId.  This guarantees that only a single gift will ever be created for a given externalId, preventing any possible errors or abuse that could produce duplicate rewards.

When redeem is called with a previously created externalId is revisited, the module will automatically redirect to the payout details page, allowing the user to view the status of their reward.

#### Gift

| Property  | Required  | Type        | Description |
|-----------|-----------|-------------|-------------|
| amount    |  Yes      | `Int`       | A whole number, greater than 0 and less than 1,000 |
| recipient |  No       | `Object`      | For compliance and security purposes, GiftRocket requires the email address and name of rewards recipients.   The developer can pass this information via the `name` and `email` properties of the recipient object or, alternatively, the user will be presented with corresponding inputs during the redemption flow. |
| catalog |  No         | `Array`     | By default, their entire GiftRocket rewards catalog will be presented as redemption options to your user.  To constrain this set, pass an array of catalog IDs as the `catalog` property within the gift configuration. |


### Events

The client library will emit events on state change.

#### onLoad

Triggered when the client is successfully mounted.  Passed a single object (the reward) to the handler as a parameter.

#### onSuccess

Triggered when the user completes their redemption selection.  For security purposes, all rewards created through this client SDK must be confirmed on the back-end via the GiftRocket REST API.  The gift object passed to the onSuccess will contain an `id` property which should be passed to your backend and then POST-ed to the GiftRocket REST API to approve the reward.

#### onError

Triggered on any error within the client.  Passed a single error object to the handler as a parameter.

#### onExit

Triggered when the user manually closes the client.


## REST API Integration

At minimum, the REST integration can be limited to approving rewards generated through the client SDK. This will execute the transaction created by the client [(view the approval REST endpoint)](https://www.giftrocket.com/docs).
