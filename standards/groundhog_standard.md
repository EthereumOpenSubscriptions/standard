

# Specification

## Summary
A standardized interface for interacting with recurring subscriptions.

##  Abstract
Groundhog believes that the creation of a subscription should have a defined set of criteria, 
we should be able to define a standardized public method that ensures that a subscription 
can be created from any type of user interface without worry about specific implementation implements.

## Our Thoughts

Interoperability between implementaitons should be the highest of priorties when it comes to developing the specification,
abstract enough that we aren't bound by the needs of others, but specific enough that we aren't fragmented when it comes to how the specification gets utilized.

## Motivation
Recurring payments are the bedrock of SaSS and countless other businesses, a robust specification for defining this interaction will enable a broad spectrum of revenue generation and business models.

#### Public View Functions




###### isValidSubscription
```SOLIDITY

/** @dev Checks if the subscription is valid.
  * @param bytes _subscriptionHash is the identifier of the customer's subscription with its relevant details.
  * @return success is the result of whether the subscription is valid or not.
*/

function isValidSubscription(bytes _subscriptionHash) view public returns (bool success);
```




###### getSubscription
```SOLIDITY

/** @dev handles the incoming data to define subscription rules
  * @param bytes _subscriptionHash is the identifier of the customer's subscription with its relevant details.
  * @return (
 address destination,
 address recipient,
 uint value,
 bytes data,
 uint created,
 uint expires,
 uint cycle,
 uint interval,
 uint lastCompleted,
 uint nextAvailable,
 uint externalId,
 uint status,
 bytes[] meta
  )
*/
function getSubscription(bytes _subscriptionHash) view public returns (
 address destination,
 address recipient,
 uint value,
 bytes data,
 uint created,
 uint expires,
 uint cycle,
 uint interval,
 uint lastCompleted,
 uint nextAvailable,
 uint externalId,
 uint status,
 uint fee,
 bytes[] meta
);
```

###### getSubscriptionValue
```SOLIDITY

/** @dev returns the value of the subscription
  * @param bytes _subscriptionHash is the identifier of the customer's subscription with its relevant details.
  * @return value is the value that has been subscribed for the requested subscription
*/
function getSubscriptionValue(bytes _subscriptionHash) view public returns (uint value);
```

###### getSubscriptionMeta
```SOLIDITY

/** @dev returns the value of the subscription
  * @param bytes _subscriptionHash is the identifier of the customer's subscription with its relevant details.
  * @return meta is the meta data associated with that subscription
*/
function getSubscriptionMeta(bytes _subscriptionHash) view public returns (bytes[] meta);
```


###### getSubscriptionData
```SOLIDITY

/** @dev returns the value of the subscription
  * @param bytes _subscriptionHash is the identifier of the customer's subscription with its relevant details.
  * @return data is the data that has is required as apart of the txn for the requested subscription
*/
function getSubscriptionData(bytes _subscriptionHash) view public returns (bytes data);
```

###### getSubscriptionInterval
```SOLIDITY

/** @dev returns the interval of the subscription 
  * @param bytes _subscriptionHash is the identifier of the customer's subscription with its relevant details.
  * @return interval is the interval for re-billing of the requested subscription
*/
function getSubscriptionInterval(bytes _subscriptionHash) view public returns (uint interval);
```

###### getSubscriptionExternalId
```SOLIDITY

/** @dev returns the value of the subscription
  * @param bytes _subscriptionHash is the identifier of the customer's subscription with its relevant details.
  * @return externalId is a hex encoded external identifier for the customers subscription
*/
function getSubscriptionExternalId(bytes _subscriptionHash) view public returns (uint externalId);

```

###### getSubscriptionDestination
```SOLIDITY

/** @dev returns the value of the subscription
  * @param bytes _subscriptionHash is the identifier of the customer's subscription with its relevant details.
  * @return destination is the value that has been subscribed for the requested subscription
*/
function getSubscriptionDestination(bytes _subscriptionHash) view public returns (address destination);

```

###### getSubscriptionRecipient
```SOLIDITY

/** @dev returns the value of the subscription
  * @param bytes _subscriptionHash is the identifier of the customer's subscription with its relevant details.
  * @return recipient is the address that the subscription pays out to.
*/
function getSubscriptionRecipient(bytes _subscriptionHash) view public returns (address recipient);
```

###### getSubscriptionExpire
```SOLIDITY

/** @dev returns the value of the subscription
  * @param bytes _subscriptionHash is the identifier of the customer's subscription with its relevant details.
  * @return expires is the timestamp that this subscription can no longer be re-billed
*/
function getSubscriptionExpire(bytes _subscriptionHash) view public returns (uint expires);

```

###### getSubscriptionStatus
```SOLIDITY

/** @dev returns the value of the subscription
  * @param bytes _subscriptionHash is the identifier of the customer's subscription with its relevant details.
  * @return status is the enumerated status of the current subscription, 0 expired, 1 active, 2 paused, 3 cancelled
*/
function getSubscriptionStatus(bytes _subscriptionHash) view public returns (uint status);

```
#### Public Functions


###### createSubscription
```SOLIDITY

/** @dev handles the incoming data to define subscription rules
  * @param address _destination
  * @param address _recipient
  * @param uint _value
  * @param bytes _data
  * @param uint _expires
  * @param uint _interval
  * @param uint _externalId
  * @param bytes[] _meta
  * @return success is the result of whether the subscription is valid or not.
*/

function createSubscription(
 address _destination,
 address _recipient,
 uint _value,
 bytes _data,
 uint _expires,
 uint _interval,
 uint _externalId,
 bytes[] _meta
 );
```


###### cancelSubscription
```SOLIDITY

/** @dev Cancels the current subscription
  * @param bytes _subscriptionHash is the identifier of the customer's subscription with its relevant details.
  * @return success is the result of the subscription being cancelled
*/
function cancelSubscription(bytes _subscriptionHash) public returns (bool success);
```

###### pauseSubscription
```SOLIDITY

/** @dev pauses the current subscription
  * @param bytes _subscriptionHash is the identifier of the customer's subscription with its relevant details.
  * @return success is the result of the subscription being paused
*/
function pauseSubscription(bytes _subscriptionHash) public returns (bool success);
```

### Conclusion
Subscriptions can and should encompass more than just token transfers, it should include all types of recurring transactions of ETH, tokens, payable smart contract methods)
