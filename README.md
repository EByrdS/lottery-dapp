# A project that involves building a simple decentralized lottery application

Run tests with

```
forge test
```

This Solidity project is a simple Lottery application.

The general idea is:

- The `Admin` contract stores the administrator of the lottery.
- The `Token` contract has the tokens that will be awarded for the winners,
  1 token per player in each round.
- The `Players` contract keeps track of the players who pay `1 ether` per ticket.
- The `Lottery` contract ensures the round is started and ended appropriately,
  and sends tokens to the winner. It doesn't send Ether, but Tokens. In
  order to reward the winner, it has to be funded appropriately.
- Tickets are sold in `ETH`, but the prize is awarded in `Token`s.

The intended steps for a run are:

1. A user deploys the `Admin` contract, becoming the owner. (Ownership is transferable).
1. The same user deploys the `Token` contract, specifying
   the total supply.
1. A `Lottery` contract is deployed, with references to the
   `Admin` and `Token` contracts. (To the `Admin` contract, not to the user's address!)
1. The `Lottery` contract will deploy its own `Players`.
1. The administrator starts the lottery.
1. The administrator opens the `Players` contract to start receiving players.
1. Each player calls `players.play` and sends `1 ether` to
   play in the lottery.
1. The administrator closes the `Players`.
1. The administrator funds the `Lottery` with enough `Token`s.
1. The administrator finishes the `Lottery`, triggering
   transfering of the funds to the winner.
1. The administrator resets the `Players` contract.
1. (At any moment) The administrator `withdraw`s all ETH fees from the `Players` contract.

This setup allows the same `Admin` to have control
over multiple `Lottery`s, and they can all refer the same `Token`. It is also not possible for the same `Players`
group to be playing in more than one `Lottery`, avoiding
awarding multiple prices to the same pool, who payed only once.

For a complete example, take a look at the `test/Lottery.t.sol` tests.

### Suboptimal processes

For the sake of **learning**, some processes are left suboptimal:

- Many utilities, like `transferOwnership` and `noReentrancy` are available
  in `Openzeppelin` libraries, which are not used here other than `ERC20`.
- Testing the `Token` contract, which only inherits from `ERC20` is more a
  visualization and practice exercise than actually testing it.
- Further improvements, like a dynamic fee price, a dynamic minimum number of players, or multiple tickets per address, were left out for simplicity, as it seems they would add little value as a learning exercise.
- The mapping of `Players.members` is erased manually. It would be much more gas efficient to just have a `mapping(uint256 => mapping(uint256 => Player))`, and increase the counter of the first dimension, effectively using a new and clean record of players. (Other variables would need to be in synchrony too). This would limit the number of lotteries to `type(uint256).max + 1` to avoid overflow.
- Since all ETH funds are transfered in the `withdraw` function, and they are all sent to the administrator, there doesn't seem to be an actual need to avoid reentrancy in there.
- The winner is calculated using the last block hash as a source of randomness. It is well known that this strategy is potentially dangerous, as miners have some inherence over it. (They can choose to discard blocks that may not benefit them)
- The `Admin` contract is just storing an address that can be rewritten to transfer ownership. It may not be of a lot of use, since the same could be coded in the `Lottery` itself.
- Finally, since the `Lottery` contract creates its own `Players`, all logic
  of the players could very well be coded in the `Lottery`.
