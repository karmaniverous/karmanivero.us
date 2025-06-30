---
# prettier-ignore
title: "Unexpected Mutations"
excerpt: "An unexpected (and probably unintended) function parameter mutation inside the web3.js library had me questioning my sanity for a couple of hours."
header:
  og_image: /assets/images/logo-web3js.jpg
  teaser: /assets/images/logo-web3js.jpg
tags:
  - web3
  - javascript
toc: false
---

<figure class="align-left drop-image">
    <img src="/assets/images/logo-web3js.jpg">
</figure>

Today was one of those days.

I'm writing a web3 application on top of
[`web3.js`](https://github.com/web3/web3.js). If you don't know, `web3` is
one of several libraries out there that help Javascript developers connect to
and manipulate Ethereum blockchains.

I'm a competent developer, so I've written an abstraction layer to simplify
interaction with my smart contracts. I call the abstraction layer's nice clean
API, and the abstraction layer deals with the `web3` library.

There are two basic ways ways to invoke a smart contract function. A
[`call`](https://web3js.readthedocs.io/en/v1.2.11/web3-eth-contract.html#methods-mymethod-call)
returns a value from the blockchain, and a
[`send`](https://web3js.readthedocs.io/en/v1.2.11/web3-eth-contract.html#methods-mymethod-send)
alters data on the blockchain without returning a value. For either invocation,
`web3` takes an `options` object as an argument. This object sets the sender
address, some stuff related to gas fees, and the amount of money you're sending
if that's what your transaction is doing.

Because of `call`/`send` segregation, you often find yourself invoking several
functions in series in order to accomplish something useful.

Most of those series are pretty short. But as my application has progressed,
I've reached a point where I need to do quite a bit of stuff on the front end to
get the application into a state where I can see what I am working on. So I
decided to create an abstraction layer for my abstraction layer that mocks the
actions a user takes on the front end. Now I can write scripts that save me a
ton of pointing and clicking.

Sweet.

Or so I thought... until the scripts started failing for absolutely no obvious
reason. Under the hood, the scripts are doing the _exact same things_ both my
unit tests and my actual front end are doing, error free. It took me two hours
to figure out what these function invocations in my scripts have in common that
they do _not_ have in common anywhere else. The answer is at once funny and
irritating as hell.

What they have in common is the `options` object.

At the top of each script, I set...

```js
const options = {
  from: web3.eth.accounts.wallet[accountIndex].address,
};
```

...and then I use that options object in each contract function invocation in my
script, like this:

```js
// get farm
const farm = new Farm(web3, farmAddress);
const games = await farm.getGames(options);

// get game
const game = new Game(web3, games[gameIndex]);
const { minLockup, minDeposit } = await game.getParams(options); // Defined below.

// deploy deposit
const deposit = new Deposit(web3);
await deposit.deploy(options); //
```

Each of the asynchronous lines above is actually a call to my `web3` wrapper.
Here are a couple defined:

```js
Farm.getGames = async (options) => {
  return await this.contract.methods.getGames().call(options);
};

Game.getParams = async (options) =>
  this.constructor.decodeParams(
    await this.contract.methods.getParams().call(options)
  );
```

Not a lot going on here... but this code was working just fine on my front end
and in my unit tests, and causing an unexplained revert when I deployed the
`Deposit` contract from my script.

Once I realized my script was sharing the `options` object between the web3
invocations, I console logged it after each one. After `farm.getGames`, the
`options` object still had the expected value:

```js
{
  from: "0xD28D1f59...";
}
```

But after `game.getParams` it looked like this:

```js
{
  from: '0xd28d1f59...',
  data: '0x5e615a6b',
  gasPrice: undefined,
  gas: undefined,
  to: '0xb97da33a...'
}
```

... which, when I passed it to `deposit.deploy`, was causing the reversion.

Hence the title of this post. Apparently, **_the `web3` contract methods call
function is mutating the `options` object you pass to it_**. I never noticed
that before, because until today I wasn't reusing the `options` object.

That is... _weird_. Hard to imagine it is by design. But it's also easy to fix,
once you know about it. Here's how I altered my abstraction layer:

```js
Farm.getGames = async (options) => {
  return await this.contract.methods
    .getGames()
    .call({ ...options });
};

Game.getParams = async (options) =>
  this.constructor.decodeParams(
    await this.contract.methods.getParams().call({ ...options })
  );
```

Now I'm passing a shallow copy of the `options` object to `web3`, which it can
mutate all it wants, and I don't care.

Problem solved!

**Update:** I raised
[an issue](https://github.com/ChainSafe/web3.js/issues/5304) on the `web3`
GitHub project. Interested to see how that goes.
{: .notice--info}

**Closure:** They [fixed it](https://github.com/ChainSafe/web3.js/issues/5304)!
I love it when the system works. 😁
{: .notice--success}
