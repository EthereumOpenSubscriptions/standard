
# Specification

## Simple Summary

A standard interface for preauthorized and recurring subscription payments.

## Abstract

This standard defines a contract interface enabling subscribers to express their intention to make regular recurring payments to a merchant, and for the contract to then make this payment on behalf of the subscriber at the appropriate interval.

This standard provides basic functionality to initiate, approve and manage subscriptions between a subscriber and merchant. These subscriptions allow a merchant to transfer a pre-authorized amount of ERC-20 tokens within each recurring time interval.

## Motivation

We want wallets to understand that you’re about to sign a recurring payment contract so that they can present you with a UI summarizing the agreement you’re about to enter into. As your wallet now knows you’ve entered into a subscription contract it can also provide appropriate UI for managing and cancelling your subscriptions in future.


## Specification

### Definitions
* Merchant - deploys this contract and typically receives the payments
* Subscriber - commits to a regular payment plan by making a transaction to this contract
* Processor - executes a payment from a subscriber to the merchant by making a transaction to this contract at the appropriate time.
* Payment - the transfer of an ERC-20 token
* Billing period - an interval of time during which the Merchant can pull payments up to a maximum amount as defined by the subscriber when creating the subscription.

### Features
* Supports any ERC-20 token as payment currency
* Merchant can pull up to a maximum authorized amount in each billing period
* First payment can be a different amount to all subsequent recurring payments
* Subscriber can cancel at any time

### Methods

#### createSubscription

Called by the subscriber, using data initiated by the merchant in a checkout flow.

* @param \_payeeAddress The address that will receive payments
* @param \_tokenAddress The address of the token contract that is used for payments
* @param \_amountRecurring The maximum amount that can be paid in each subscription period
* @param \_amountInitial The amount to be paid immediately, can be lower than total allowable amount
* @param \_periodType Can be hour (0), day (1), week (2), month (3), year (4)
* @param \_periodMultiplier The number of periodType that must elapse before the next payment is due
* @param \_startTime Date that the first recurring payment is due
* @return A bytes32 for the created subscriptionId

```
function createSubscription(
    address _payeeAddress,
    address _tokenAddress,
    uint _amountRecurring,
    uint _amountInitial,
    uint _periodType,
    uint _periodMultiplier,
    uint _startTime,
    string _data
    )
    public
    returns (bytes32)
```

The \_amountInitial SHOULD be able to be zero so that free trials can be supported.

Prior to calling this function the subscriber MUST have approved this contract to make transfers on their behalf by calling approve() on the ERC-20 payment token.  The minimum amount tokens that must be approved in order to result in a successful creation are left to the implementation.

#### updateSubscription

Called by the subscriber to update the parameters of an existing subscription.

* @param \_subscriptionId The ID of the subscription to update
* @param \_tokenAddress The address of the token contract that is used for payments
* @param \_amountRecurring The maximum amount that can be paid in each subscription period
* @param \_amountInitial The amount to be paid immediately, can be lower than total allowable amount
* @param \_periodType Can be hour (0), day (1), week (2), month (3), year (4)
* @param \_periodMultiplier The number of periodType that must elapse before the next payment is due
* @param \_startTime Date that the subscription becomes active
* @return A boolean that will be true if the update is successful

```
function updateSubscription(
    bytes32 _subscriptionId,
    address _tokenAddress,
    uint _amountRecurring,
    uint _amountInitial,
    uint _periodType,
    uint _periodMultiplier,
    uint _startTime,
    string _data
    )
    public
    returns (bool)
```


#### updateSubscriptionAddress

Can be called by the merchant to update their payee address.

* @param \_subscriptionId The ID of the subscription to update
* @param \_payeeAddress The address that will receive payments
* @return A boolean to indicate whether the update was successful

```
function updateSubscriptionAddress(
    bytes32 _subscriptionId,
    address _payeeAddress,
    )
    public
    returns (bool)
```

#### processSubscription

Called by or on behalf of the merchant, in order to initiate a scheduled/approved payment. `amount` can be lower than total allowable amount.

* @param \_subscriptionId The subscription ID to process payments for
* @param \_amount Amount to be transferred, can be lower than total allowable amount
* @return A boolean to indicate whether the payment was successful

```
function processSubscription(
    bytes32 _subscriptionId,
    uint _amount
    )
    public
    returns (bool)
```

#### pauseSubscription

Called by subscriber, or by or on behalf of merchant in order to pause an active subscription

* @param \_subscriptionId The subscription ID to process payments for
* @return A boolean to indicate whether the pause operation was successful

```
function pauseSubscription(bytes32 _subscriptionId) returns (bool success)
```


#### cancelSubscription

Called by subscriber, or by or on behalf of merchant in order to cancel an active subscription

* @param  \_subscriptionId The subscription ID to delete
* @return a boolean to indicate whether the subscription was successfully cancelled

```
function cancelSubscription(bytes32 _subscriptionId) returns (bool success)
```

#### amountUnclaimed

Returns the amount, if any, that has been approved but not collected in the current billing period. This is relevant for iTunes or API-style billing (e.g. multiple transactions over a month)

* @param  \_subscriptionId The subscription ID to check amount unclaimed for
* @return remaining amount that can be claimed in this billing period

```
function amountUnclaimed(bytes32 _subscriptionId) view returns (uint amountUnclaimed)
```


### Events

#### NewSubscription

MUST trigger on any successful call to `createSubscription`

```
event NewSubscription(
    bytes32 _subscriptionId,
    address _payeeAddress,
    address _tokenAddress,
    uint _amountRecurring,
    uint _amountInitial,
    uint _periodType,
    uint _periodMultiplier,
    uint _startTime
    );
```

#### UpdateSubscription

MUST trigger on any successful call to `updateSubscription`

```
event UpdateSubscription(
    bytes32 _subscriptionId,
    address _payeeAddress,
    address _tokenAddress,
    uint _amountRecurring,
    uint _amountInitial,
    uint _periodType,
    uint _periodMultiplier,
    uint _startTime
    );
```

#### UpdateSubscriptionAddress

MUST trigger on any successful call to `updateSubscriptionAddress`

```
event UpdateSubscriptionAddress(
    bytes32 _subscriptionId,
    address _payeeAddress
    );
```

#### PauseSubscription

MUST trigger on any successful call to `pauseSubscription`

```
event PauseSubscription(
    bytes32 _subscriptionId,
    );
```

#### CancelSubscription

MUST trigger on any successful call to `cancelSubscription`

```
event CancelSubscription(
    bytes32 _subscriptionId,
    );
```

#### Payment

MUST trigger on any successful call to `processSubscription`

```
event Payment(
  bytes32 _subscriptionId,
  address payeeAddress,
  address tokenAddress,
  uint unitAmount,
  uint timestamp
  );
```


## Implementation

* [Work in progress implementation here](https://github.com/johngriffin/ERC948)

## History

Historical links related to this standard:

*  Original proposal from Kevin Owocki and Github discussion: [ethereum/EIPs#948](https://github.com/ethereum/EIPs/issues/948)

* This spec is based on a [spec by @sb777](https://github.com/sb777/erc-948-draft/issues/1)
