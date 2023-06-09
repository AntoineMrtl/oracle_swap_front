# Oracle Swap AMM frontend implementation

See https://github.com/AntoineMrtl/oracle_swap_back for the contracts and deployment.

Deployed test site : https://dreamy-paletas-e071e5.netlify.app/
***/!\ ONLY BTC/ETH pair work, the others are just exemples, there aren't contracts deployed for the others pairs. /!\***

Deployed addresses on FUJI Testnet :

**OracleSwap**: 0x05d53cC46a6b33a3a7dd7855F25F75D60f13479b / **MockETH**: 0x542C53Ecb71fA46Ea988aD47E53820D2481DcF09
**MockBTC**: 0x57ac342EAfdd7f46E5d61013697791400610fd5B / **Pyth**: 0xff1a0f4744e8582DF1aE09D5611b887B6a12925C

## How it works (Oracle Swap backend)

The base contract was a fork of : https://github.com/pyth-network/pyth-crosschain/tree/main/target_chains/ethereum/examples/oracle_swap

The contract holds a pool of two ERC-20 tokens, the BASE and the QUOTE, and allows users to swap tokens for the pair BASE/QUOTE. For example, the base could be WETH and the quote could be USDC, in which case you can buy WETH for USDC and vice versa. The pool offers to swap between the tokens at the current Pyth exchange rate for BASE/QUOTE, which is computed from the BASE/USD price feed and the QUOTE/USD price feed.

Users can deposit tokens (both BASE and QUOTE) at a fixed ratio (which will be considered to be close to the token price) to the liquidity pool to allow the swap. As the price is external and do not depend of the contract, the pool can become unbalanced, which can lead to a non optimal efficiency for liquidity providers (e.g. must deposit a 20:1 ratio in the liquidity whereas the price is 2:1, which can lead to a lot of token completly unused and less liquidity deposited)

To fix the imbalance issue, there is an incentive to arbitrate between the pool price and the real price : if the pool price is imbalance (the difference with the real price exceeds a certain threshold), arbiters are allows to buy (or sell according to the imbalance side) directly on the contract liquidity pool to bring the pool price closer to the real price. The base oracle-based swap can remain open or not during an imbalance event at the wish of the operator. Finally, fees are taken for each swap to encourage the deposit of liquidity.

## How to use it

The frontend is divided in three parts to fullfill all the requirements of the dapp : 

### Swap

The swap is the main part of the oracle swap. It allows everyone to swap tokens (only one pair of two tokens per contract), to buy or sell at a fixed price determined by the pyth price feed (external oracle price feed). The swap is based on the liquidity available in the pool and does not perform balancing maneuver on the pool (it simply transfer tokens from the pool to the user and from the user to the pool without further calculation). The swap can be desactivate with a variable check according to the contract operator if the pool is too unbalanced (see 2nd part). In the frontend, the token bought or sold (little BUY/SELL button) is the first of the pair selected. For exemple, if the button is on BUY and the pair selected is BTC/ETH, the swap will buy the amount entered in the input box of BTC.

### Arbitrage

The pool is unbalanced if the virtual price of the pool (ratio of the pool reserves, the "pool price"), is too different from the real price, (10% offset here in the contract). Why does the pool balancing matters ? As explained before, it can leads to unoptimal liquidity reserves, with for exemple a pool reserves of 10 times more tokenA than tokenB, whereas the real price of tokenA can be 2:1 : the liquidity providers must deposit much more tokenA than necessary and a lot of them will never be used, which lead to less liquidity and more tokens vested in the contract.
When the pool is unbalanced, the arbitrate function is available and can be accessed by the front by selecting a pair, and by putting the amount of token we want to buy or sell depending of the side of the imbalance. The contract will then buy (or sell) tokens directly on the pool at the pool price instead of using oracle price and will thus slightly rebalance the pool to the oracle price. The swap at oracle price can be desactivate or not during imbalance phase.

### Pools

Finally, users can deposit and remove liquidity using the third part of the frontend. While adding liquidity, they can put the number they want to add to only one of the input boxes, because the other will be calculated by the contract to fit the pool ratio accordingly to the first input (or second). To remove liquidity, they can click on the button up in the corner on the right and set the percentage (integers number between 1 and 100) they want to withdraw. The fees are automatically removed with the liquidity (you get x% of your total fees earned when you remove x% of your liquidity).
