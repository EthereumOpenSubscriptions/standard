# Specification

## Summary
A standardized interface for interacting with subscriptions.

##  Abstract
This standard defines a contract interface for interacting with subscription data. The scope of this proposal is end-user / wallet interactions. Creation of the subscriptions is out-of-scope of this ERC.

## Our Thoughts
From the subscriber's side:

* I want strong guarantees on when I can cancel my subscription.
* I will normally want payments to happen automatically without any action on my part.
* In some cases it might be valuable to require an approval process.
* For dynamically priced subscriptions I want to be able to set limits (require authorization if subscription is more than X).


From the providers's side:

* I need the ability to charge a fixed fee per subscription time unit (netflix, pandora, etc).
* I need the ability to charge a dynamic fee per subscription time unit (aws, twilio, etc).
* I need to be able to create reasonably accurate forecasts for upcoming subscriptions: Programatic checks that subscription accounts have available balance and that subscription is active.

## Motivation
Standard that covers all usecases of subscription based model with minimal interface suitable for wallets.

### Interface

#### cancel

Immediately cancels the subscription. Implementations should ensure that any unpaid subscription payments are paid to the provider as part of cancellation to ensure providers are able to let subscriptions fees fill up for arbitrary lengths of time, allowing them to reduce overhead from transaction costs.

```SOLIDITY

/** @dev Cancels the subscription
  * @return true if cancellation succedded 
*/

function cancel()
    public
    returns (bool success);
```

#### isFunded

Checkes if there is enough funds available for the provider to collect
 
```SOLIDITY

/** @dev Cancels the subscription
  * @return true if there is enough funds allowed to the contract to pay for subscription 
*/

function isFunded()
    public
    returns (bool funded);
```

#### availableFunds

Returns the amount of ETH or tokens available for the provider to collect

 
```SOLIDITY

/** @dev The amount of ETH or tokens available for the provider
  * @return available funds
*/

function availableFunds()
    public
    returns (uint256 funds);
```

#### metadata

Returns the off-chain location for metadata. 
By *metadata* we define a self-described data not required by on-chain contract logic and thus possible to be kept off-chain. An example provided [here](#metadata-example)

```SOLIDITY

/** @dev Serialized bytes of subscription data
  * @return available funds
*/

function metadata()
    public
    returns (bytes metadata);
```



### Implementation

A minimal implementation would require the following fields.

* `address token`: defines the token contract which payments are paid from.
* `address provider`: the address of the provider
* `uint256 time_unit`: the number of seconds per time unit.
* `uint256 tokens_per_time_unit`: the number of tokens that can be paid towards the subscription per time_unit.
* `uint256 last_payment_at`: the timestamp when the last payment was made.

In order to allow triggering payments by external parties the contract can follow EIP #1228. 

The `execute` method would call `token.transfer(provider, (now - last_payment_at) * tokens_per_time_unit / time_unit)`.

### Metadata example

Basic information about the subscription:
* name = "Monthly subscription"
* description = "Any other extra description to be visible from wallet"
* key = "value"

Which can be JSON serialized as

```Json
{"name":"Monthly subscription","description":"Any other extra description to be visible from wallet","key":"value"}
```

Things to consider:
* limit the amount of keys and values 

### Conclusion
This is our suggestions for the beginning of a standardization on interaction with subscription data on the Ethereum blockchain. We are open to suggestions, concerns, or rejections. Let's collaborate and build a better ecosystem.
