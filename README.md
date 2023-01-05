# Astaria contest details
- Total Prize Pool: $90,500 USDC
  - HM awards: $63,750 USDC
  - QA report awards: $7,500 USDC
  - Gas report awards: $3,750 USDC
  - Judge + presort awards: $15,000 USDC
  - Scout awards: $500 USDC
- Join [C4 Discord](https://discord.gg/code4rena) to register
- Submit findings [using the C4 form](https://code4rena.com/contests/2023-01-astaria-contest/submit)
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Starts January 5, 2023 20:00 UTC
- Ends January 19, 2023 20:00 UTC

## C4udit / Publicly Known Issues

The C4audit output for the contest can be found [here](add link to report) within an hour of contest opening.

*Note for C4 wardens: Anything included in the C4udit output is considered a publicly known issue and is ineligible for awards.*

# Overview

Astaria is an NFT lending protocol offering instant liquidity to borrowers. Strategists deploy Vaults and provide updateable loan terms on a per-NFT basis. Borrowers deposit their NFTs into the Astaria protocol and to borrow according to a strategist's terms. PrivateVaults may only accept capital from the strategist that deploys them, while PublicVaults may accept capital from any liquidity providers. PublicVaults operate around an epoch system that requires liquidity providers to signal which epoch they wish to withdraw in advance.

Please see https://docs.astaria.xyz/docs/intro for a detailed description of protocol functionality and smart contract architecture.

# Scope

All contracts in `src/` have accompanying interfaces with natspec documentation in the `interfaces` folder. Points of complexity to thoroughly audit are the validation of loan terms using merkle proofs (in VaultImplementation), and edge cases around liquidity providers withdrawing from PublicVaults through WithdrawProxies.

| Contract Name                     | SLOC | Purpose                                                                                                                                                                                               |
| --------------------------------- | ---- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| src/AstariaRouter.sol             | 556  | A router contract for handling universal protocol behavior and endpoints into other core contracts.                                                                                                   |
| src/AstariaVaultBase.sol          | 33   | Contract with pointers to vault constants and contract implementations.                                                                                                                               |
| src/VaultImplementation.sol       | 243  | Base vault contract with behavior for validating and issuing loan terms.                                                                                                                              |
| src/Vault.sol                     | 37   | PrivateVault contract, where only permissioned lenders can deposit funds.                                                                                                                             |
| src/PublicVault.sol               | 442  | Contract for permissionless-lending Vaults, handling liquidations and withdrawals according to the epoch system.                                                                                      |
| src/LienToken].sol                | 594  | LienTokens are non-fungible tokenized debt owned by Vaults. This contract handles the accounting and liquidation of loans throughout their lifecycle.                                                 |
| src/CollateralToken.sol           | 428  | CollateralTokens are ERC721 certificates of deposit for NFTs being borrowed against on Astaria, giving the owner the right to the underlying asset when all debt is paid off.                         |
| src/WithdrawVaultBase.sol         | 23   | Base contract for WithdrawProxy.                                                                                                                                                                      |
| src/WithdrawProxy.sol             | 166  | A new WithdrawProxy is deployed for each PublicVault when at least one LP wants to withdraw by the end of the next epoch. It handles funds from loan repayments and auction funds.                    |
| src/ClearingHouse.sol             | 134  | ClearingHouses are deployed for each new loan and settle payments between Seaport auctions and Astaria Vaults if a liquidation occurs. It also stores NFTs that borrowers deposit to take out a loan. |
| src/BeaconProxy.sol               | 33   | Beacon contract for upgradeability.                                                                                                                                                                   |
| src/TransferProxy.sol             | 12   | The TransferProxy handles payments to loans (LienTokens).                                                                                                                                             |
| lib/gpl/src/ERC20-Cloned.sol      | 126  | Custom base ERC20 implementation.                                                                                                                                                                     |
| lib/gpl/src/ERC721.sol            | 155  | Slightly modified base ERC721 implementation.                                                                                                                                                         |
| lib/gpl/src/ERC4626-Cloned.sol    | 93   | Custom base ERC4626 implementation.                                                                                                                                                                   |
| lib/gpl/src/ERC4626RouterBase.sol | 33   | ERC4626 base router contract.                                                                                                                                                                         |
| lib/gpl/src/ERC4626Router.sol     | 27   | ERC4626 router contract.                                                                                                                                                                              |
| src/interfaces/IWithdrawProxy.sol         | 79   | WithdrawProxy interface.                                                           |
| src/interfaces/ISecurityHook.sol         | 18   | SecurityHook interface.                                                |
| src/interfaces/IERC165.sol         | 25   | ERC165 interface.  
| src/interfaces/ILienToken.sol         | 346   | LienToken interface.  
| src/interfaces/ICollateralToken.sol         | 187   | CollateralToken interface.  
| src/interfaces/IRouterBase.sol         | 22   | AstariaRouter base contract interface.   
| src/interfaces/IERC4626.sol         | 263   | ERC4626 interface.
| src/interfaces/IPublicVault.sol         | 173   | PublicVault interface.  
| src/interfaces/IERC20Metadata.sol         | 19   | ERC20 metadata interface.
| src/interfaces/IV3PositionManager.sol         | 177   | Uniswap V3 PositionManager interface.
| src/interfaces/IFlashAction.sol         | 26   | FlashAction interface. Anyone may create a contract implementing this interface to define an action performed by a collateralized NFT in a flash loan.
| src/interfaces/IERC721.sol         | 48   | ERC721 interface.
| src/interfaces/IVaultImplementation.sol         | 105   | VaultImplementation interface.                 
| src/interfaces/IERC1155.sol         | 141   | SecurityHook interface.               
| src/interfaces/IBeacon.sol         | 23   | Beacon contract interface. 
| src/interfaces/IAstariaVaultBase.sol         | 32   | Vault base contract interface.
| src/interfaces/IERC721Receiver.sol         | 27   | ERC721Receiver interface.
| src/interfaces/ITransferProxy.sol         | 23   | TransferProxy interface.                            |
| src/interfaces/IERC1155Receiver.sol         | 58   | ERC1155Receiver interface.
| src/interfaces/IAstariaRouter.sol         | 342   | AstariaRouter interface.
| src/interfaces/IStrategyValidator.sol         | 26   | StrategyValidator interface.
| src/interfaces/IERC20.sol         | 85   | ERC20 interface.
| lib/gpl/source/interfaces/IERC4626RouterBase.sol         | 105   | ERC4626-compliant AstariaRouter base contract interface.
| lib/gpl/source/interfaces/IERC4626Router.sol         | 58   | ERC4626-compliant AstariaRouter main contract interface.
| lib/gpl/source/interfaces/IWETH9.sol         | 12   | ERC4626-compliant AstariaRouter base contract interface.
| lib/gpl/source/interfaces/IUniswapV3PoolState.sol         | 116   | Uniswap V3 state contract interface.
| lib/gpl/source/interfaces/IMulticall.sol         | 16   | Multicall interface.
| lib/gpl/source/interfaces/IUniswapV3Factory.sol         | 78   |Uniswap V3 factory contract interface.
| **Total (including interfaces)**  | 3475 |

# Additional Context

*Sponsor, please confirm/edit the information below.*

## Scoping Details 
```
- If you have a public code repo, please share it here: https://github.com/AstariaXYZ/astaria-core
- How many contracts are in scope?: 37 (including interfaces)
- Total SLoC for these contracts?: 3,475
- How many external imports are there?: 7
- How many separate interfaces and struct definitions are there for the contracts within scope?: 25 interfaces, 35 structs  
- Does most of your code generally use composition or inheritance?: Inheritance  
- How many external calls?: 2
- What is the overall line coverage percentage provided by your tests?: ~90 (`forge coverage` currently throws a "stack too deep" error on large codebases)
- Is there a need to understand a separate part of the codebase / get context in order to audit this part of the protocol?: Yes  
- Please describe required context: The protocol contains a few contracts that are interwoven to maintain the entire protocol, so good understanding of one aspect helps understand settlement in other areas.   
- Does it use an oracle?: No
- Does the token conform to the ERC20 standard?: No token
- Are there any novel or unique curve logic or mathematical models?: PublicVaults (open to all liquidity providers) use an epoch system which schedules withdrawals and distributes auction funds in order to ensure solvency and prevent there ever being a run on a Vault.
- Does it use a timelock function?: Not directly, we have a method on the router fileGuardian that will be behind a timelock eventually.
- Is it an NFT?: The protocol tokenizes loans as LienTokens and mints CollateralTokens as certificates of deposit on locked collateral.
- Does it have an AMM?: No
- Is it a fork of a popular project?: No 
- Does it use rollups?: No 
- Is it multi-chain?: No
- Does it use a side-chain?: No
```

# Tests

For more details on the Astaria protocol and its contracts, see the [docs](https://docs.astaria.xyz/docs/intro)

# Astaria Contracts Setup

Astaria runs on [Foundry](https://github.com/foundry-rs/foundry). If you don't have it installed, follow the installation instructions [here](https://book.getfoundry.sh/getting-started/installation).

To install contract dependencies, run:

```sh
forge install
yarn
```

To Deploy on a forked network, update your RPC in docker-compose.yml first and then run:

```
sh scripts/boot-system.sh
```

Tests are located in src/test. To run tests, run:

```sh
forge test --ffi
```

Note that since ```testClaimFeesAgainstV3Liquidity()``` uses on-chain data, it will revert when running in a non-forked test environment. To run this forked test, use:

```sh
forge test --fork-url <YOUR_RPC_URL>
```

To target integration tests following common use paths, run:

```sh
forge test --ffi --match-contract AstariaTest
```

To target tests following disallowed behavior, run:

```sh
forge test --ffi --match-contract RevertTesting
```

Edge cases around withdrawing and other complex protocol usage are found in the WithdrawTest and IntegrationTest contracts.

To view a gas report, run:
```sh
forge snapshot --gas-report --ffi
```
