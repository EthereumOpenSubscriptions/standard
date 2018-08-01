# Subscription Standard (Working Draft)

## Short Summary

A collection of interfaces for Ethereum subscription payment model as currently suggested by the working group:
* ERC948 - read functions
* ERC948Write - write functions
* ERC948Metadata - support for metadata (optional)
* ERC948Processor - support for processors (optional)

## Abstract

The following standard allows for the implementation of a standard API for subscription payment model within smart contracts. This standard provides basic functionality to track, manage and execute subscriptions. The standard supports various subscription models.

Subscriptions represent a recurring payment model with an opt-out option by either individuals or providers.

The standard aims to provide a common interface for user wallets and merchant integration libraries to support recurring payments. It however does not cover subscription creation as this is in a domain of specific implementation.

## Motivation

The goal is to create a minimum standard required to process recurring payments of arbitrary assets (e.g. ERC20 tokens). On-chain business logic should only serve to provide a reliable trustless service of trasfering funds from subscription user's wallet to subscription provider's wallet in predefined recurring events while providing guarantees defined in the [Abstract](#Abstract).

We also believe that the standard should not be expanded beyond minimum requirements only to suppport different business models as this would limit the number of different subscription service solutions. Such solutions should be able to easily extend the standard interface to accomodate additional business logic required.

The standard should support all commonly used subscription models:
* pre-paid / post-paid (at the beginning or at the end of a billing cycle)
* fixed / varied amount
* discounted or trial periods
* limited / unlimited duration

The standard's interfaces should also be designed in a way that enables scalable solutions with minimum transaction costs.

The standard should also adhere to ERC165 for increased adoption.

## Rationale

### Follow best practices of existing standards

ERC721 has been already accepted and has a lot of paralles with ERC948. It handles non-fungible tokens, whereas ERC948 handles subscriptions. Their rationale around the Interface has gone a through long process and we believe we should lean on and learn from their conclusions.

### Number of participants

In subscription based model of payment there are at least two participants: user and provider. The minimal set of read functions has to support both of them. User and provider should be able to query their subscription lists and subscription details. Since these are two different roles subscription interface has to separately support both of them. This also supports edge cases where user and provider are the same address.

### Subscription Identifiers

Each ERC948 contract can have one or more providers which means relationships between users and providers are many-to-many. There is a requirement of every subscription to have its own globally unique ID. The smallest set of parameters to generate globally unique IDs on-chain are provider's address and provider supplied subscription ID (e.g. external ID from their database).

The choice of `uint256` allows a wide variety of applications because UUIDs and sha3 hashes are directly convertible to `uint256`. This follows the same rationale from ERC721 specification.

#### Collisions and attack surface

There are many ways of how to construct globally unique subscription IDs and whether should that happen off-chain or on-chain. Here we examine some of the proposals:

1. Each subscription ID is calculated off-chain by hashing subscription data: `hash(providerAddress, userAddress, externalSubsID, amount, etc.)`. This is then supplied as part of the create subscription transaction. There are two drawbacks to this:
    - Calculating globally unique IDs is left to the provider. This is not reliable and rouge calculations (leaving out provider addres, user address and others) can result in on-chain collisions, which severly degrade user experience and require additional off-chain logic to handle them.
    - Front-running DoS attack. A malicious entity can deprive an honest provider from subscribing new users. Each time a new "create subscription" transaction is broadcast to the network, malicious entity front-runs the transaction and creates a subscription with the same ID with no means of actually paying (e.g. ERC20 allowance set to 0, or setting it to 0 shortly after). The result is that all of the newly created subscriptions fail since they're duplicated by a malicious entity.
    - To solve both, there has to be on-chain code which re-calculates the hash and confirms subscription ID is calculcated with the right parameters and it belongs to the sender. However, it raises a question of why doing it off-chain if on-chain re-calculation has to happen anyway.
1. Each subscription ID is supplied by provider and standard treats them as black-boxes.
    - This is the most loose approach and prone to on-chain collisions. It has the same drawbacks as 1.
1. Each subscription ID is calculated on-chain from a tuple `(providerAddress, externalId)`. External ID can be provider's external subscription ID or hash of subscription parameters or something else entirely.
    - Moving everything to be done on-chain prevents front-running attacks. We pass subscription parameters to the "create subscription" function and globally unique ID is generated on-chain. Using at least (`providerAddress`, `externalSubscriptionID`) also guarantees there are no ID collisions (NOTE: we rely on the fact that `externalSubscriptionID` supplied by provider is unique).


### ERC165 Interface

Standard Interface Detection (ERC165) is used to expose the interfaces that a ERC948 smart contract supports.

### GAS cost and consumption

This proposal supports implementations which can manage arbitrary large number of subscriptions. It avoids defining functions which would require implementations to use for/while loops in the code (e.g. returning all user subscription IDs, etc.). These indicate implementations may not be able to scale and gas costs will rise indefinitely.


## ERC948 Interface

```solidity
  interface ERC948 /* is ERC165 */ {

  /**
   * @notice User is determined by msg.sender.
   * @dev Return user subscription providers if there are any.
   * @param _user User's address.
   * @return Array of provider addresses.
   **/
  function getUserSubscriptionProviders(address _user) external view
    returns (address[]);

  /**
   * @notice User is determined by msg.sender.
   * @dev Return user subscription IDs if there are any.
   * @param _user User's address.
   * @param _provider Provider's address.
   * @return Array of subscription IDs.
   **/
  function getUserSubscriptionIds(address _user,
                                  address _provider) external view
    returns (uint256[]);

  /**
   * @dev Return the number of provider subscriptions.
   * @param _provider Provider's address.
   * @return Number of subscriptions.
   **/
  function getNumberOfProviderSubscriptions(address _provider) external view
    returns (uint256);

  /**
   * @notice Provider is determined by msg.sender
   * @dev Return provider subscription IDs if there are any.
   * @param _index Index number in the user's subscriptions array.
   * @param _number Offset which determines the number of subscriptions to return.
   * @return uint256 array of subscription IDs.
   **/
  function getProviderSubscriptionIds(address _provider,
                                      uint256 _index,
                                      uint256 _number)
    external view
    returns (uint256[]);

  /**
   * @dev Get subscription's details.
   * @param _subscriptionId Subscription's ID.
   * @return Subscription plan as tuple of:
   *  _provider Provider's address
   *  _user User's address
   *  _subscriptionId subscriptionId
   *  _amount Amount to be paid in the next billing period. Amount returned can
   *          be constant or it can vary over time (per usage based subscriptions,
   *          subscriptions supporting trial and discounted periods).
   *  _nextPaymentDate Next payment date as defined in a subscription plan.
   *                   If there's no next payment date this is set to 0.
   *                   This could be the case in a limited subscription (predefined
   *                   number of recurring payment periods).
   *  _timeUnit Predefined constant representing an hour, day, month or a year.
   *  _period Number of _timeUnits between recurring payments.
   *  _asset Payment asset address.
   **/
  function getSubscription(address _provider, uint256 _subscriptionId)
    external view
    returns(address _provider,
            address _user,
            uint256 _subscriptionId,
            uint256 _amount,
            uint256 _nextPaymentDate,
            uint8 _timeUnit,
            uint256 _period,
            address _asset);
}

interface ERC165 {
  /**
   * @notice Query if a contract implements an interface
   * @param interfaceID The interface identifier, as specified in ERC-165
   * @dev Interface identification is specified in ERC-165. This function
   * uses less than 30,000 gas.
   * @return `true` if the contract implements `interfaceID` and
   * `interfaceID` is not 0xffffffff, `false` otherwise
   **/
  function supportsInterface(bytes4 interfaceID) external view returns (bool);
}

```
## ERC948Write Interface
```solidity

interface ERC948Write /* is ERC948 */ {
/**
   * @dev This emits when subscription is created.
   **/
  event Subscription(address indexed _user,
                     address indexed _provider,
                     uint256 indexed _subscriptionId);

  /**
   * @dev This emits when subscription is cancelled. _from address belongs to
   * user or provider.
   **/
  event SubscriptionCancellation(address indexed _from,
                                 address indexed _provider,
                                 uint256 indexed _subscriptionId);

  /**
   * @dev This emits when recurring payment is executed. _from address belongs
   * to provider (or processor).
   **/
  event SubscriptionPayment(address indexed _from,
                            address indexed _provider,
                            uint256 indexed _subscriptionId);

  /**
   * @dev Cancel active subscription. This can be done by user or provider.
   * @param _provider Provider's address.
   * @param _subscriptionId Subscription's ID.
   **/
  function cancelSubscription(address _provider, uint256 _subscriptionId) external;

  /**
   * @notice Called by provider.
   * @dev Execute overdue payment.
   * @param _provider Provider's address.
   * @param _subscriptionId Subscription ID.
   **/
  function executePayment(address _provider, uint256 _subscriptionId) external;
}
```

## ERC948Metadata Interface

The metadata extension is OPTIONAL for ERC948 smart contracts. This allows a smart contract to be quieried about the subscription details.


```solidity
interface ERC948Metadata /* is ERC948 */ {

  /**
   * @notice A distinct Uniform Resource Identifier (URI) for a given subscription.
   * @dev Throws if `_provider` and `_subscriptionID` are not a valid
   * combination. URIs are defined in RFC 3986. The URI may point to a JSON file
   * that conforms to the "ERC948 Metadata JSON Schema" (NOTE: NOT YET DEFINED).
   * @param _provider Provider's address.
   * @param _subscriptionId Subscription's ID.
   **/
    function subscriptionURI(address _provider, uint256 _subscriptionId)
      external view returns (string);
}
```

# Support for Processors

## Short Summary

Delegate payment executions to a third party.

## Abstract

Payment execution can be delegated to a third party subscription processors ("operators"). Providers/merchants want reliable recurring payments and might be willing to pay for that service.

The extended standard offers subscription providers a way to register a third party subscription processor to execute payments on their behalf.

## Motivation

Subscription provider might not want to be the one executing payments since is does not want to deal with interacting with the subscription contract, following events, handling overdue payments and might want to delegate this responsibility to a third party.

## Interface
```solidity

interface ERC948Processor /* is ERC948 */ {

  /**
   * @dev This emits when the approved address for processor is
   * changed or reaffirmed. The zero address indicates there is no
   * approved address.
   **/
  event ApproveSubscriptionProcessor(address indexed _provider,
                                     address indexed _processor);

  /**
   * @notice This gives processor ability to call executePayment in
   * behalf of provider.
   * @dev Approve operator (processor) to execute transactions in the name
   *      of provider. The zero address indicates there is no approved address.
   * @param _processor Operator's address.
   **/
  function approveProcessor(address _processor) external;

  /*
   * @dev Return the current operator (processor) for a given provider.
   * @param _provider Provider's address.
   **/
  function getApprovedProcessor(address _provider) public view
    returns (address);

}
```
