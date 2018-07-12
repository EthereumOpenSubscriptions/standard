From https://github.com/ethereum/EIPs/issues/948#issuecomment-376213214

<p>I think that <em>price</em> should be out-of-scope for this standard.  This feels a lot like scope creep to me and doesn't need to be part of the core API.  Ideally whatever protocol rules are decided <em>allow</em> for this type of behavior.</p>
<p>From the subscriber's side:</p>
<ul>
<li>I want strong <strong>guarantees</strong> on when I can cancel my subscription.</li>
<li>I will <strong>normally</strong> want payments to happen automatically without any action on my part.</li>
<li>In some cases it <strong>might</strong> be valuable to require an approval process.</li>
<li>For <em>dynamically</em> priced subscriptions I want to be able to set limits (require authorization if subscription is more than X).</li>
</ul>
<p>From the providers's side:</p>
<ul>
<li>I need the ability to charge a fixed fee per subscription time unit (netflix, pandora, etc).</li>
<li>I need the ability to charge a dynamic fee per subscription time unit (aws, twilio, etc).</li>
<li>I need to be able to create reasonably accurate forecasts for upcoming subscriptions: Programatic checks that subscription accounts have available balance and that subscription is active.</li>
</ul>
<p>And to think a bit about protocol:</p>
<p>Building on the token ERCs seems valuable here since they already setup primatives for transferring and approving.  What's missing is the concept of time based approvals.  I <em>think</em> that we can get very close to hitting <strong>all</strong> of the use cases above with the following API</p>
<h3>Subscription API</h3>
<p>I think we need the following minimal API</p>
<ul>
<li><code>triggerPayment() returns bool</code> : Triggers payment of the subscription.  This spec does not specify the behavior of this function, leaving it up to the implementer.</li>
<li><code>cancel() returns bool</code>: Immediately cancels the subscription.  Implementations <strong>should</strong> ensure that any unpaid subscription payments are paid to the provider as part of cancellation to ensure providers are able to let subscriptions fees <em>fill up</em> for arbitrary lengths of time, allowing them to reduce overhead from transaction costs.</li>
</ul>
<p>These probably have <code>Payment</code> and <code>Cancelled</code> events that would get fired.</p>
<h3>An ERC20 token based subscription contract.</h3>
<p>With the above, we can now think about what a subscription paid in ERC20 tokens might look like.  A minimal implementation would require the following fields.</p>
<ul>
<li><code>address token</code>: defines the token contract which payments are paid from.</li>
<li><code>address provider</code>: the address of the provider</li>
<li><code>uint256 time_unit</code>:  the number of seconds per time unit.</li>
<li><code>uint256 tokens_per_time_unit</code>:  the number of <code>tokens</code> that can be paid towards the subscription per <code>time_unit</code>.</li>
<li><code>uint256 last_payment_at</code>: the timestamp when the last payment was made.</li>
</ul>
<p>The <code>triggerPayment</code> method would call <code>token.transfer(provider, (now - last_payment_at) * tokens_per_time_unit / time_unit</code>)`.</p>
<h3>Closing Thoughts</h3>
<p>Given the wide set of use cases for subscriptions and the wide array of different business requirements, I think this specification will be <strong>most</strong> useful if it sticks to trying to define an interface, and leaves the exact implementation up to the provider.  A provider would either provide their own implementation of a subscription contract, requiring the user to fund the contract once it was created for them, <strong>or</strong> they might delegate to a 3rd party service which offers pre-built subscription contracts that fit their business requirements.</p>