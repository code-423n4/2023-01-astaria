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

|File|[SLOC](#nowhere "(nSLOC, SLOC, Lines)")|Description|Libraries|
|:-|:-:|:-|:-|
|_Contracts (9)_|
|[src/TransferProxy.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/TransferProxy.sol)|[17](#nowhere "(nSLOC:12, SLOC:17, Lines:36)")|The TransferProxy handles payments to loans (LienTokens).| `solmate/*` |
|[src/BeaconProxy.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/BeaconProxy.sol) [🖥](#nowhere "Uses Assembly") [💰](#nowhere "Payable Functions") [👥](#nowhere "DelegateCall")|[33](#nowhere "(nSLOC:33, SLOC:33, Lines:88)")|Beacon contract for upgradeability.|  `clones-with-immutable-args/*`|
|[src/Vault.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/Vault.sol)|[63](#nowhere "(nSLOC:37, SLOC:63, Lines:93)")|PrivateVault contract, where only permissioned lenders can deposit funds.| `solmate/*` |
|[src/ClearingHouse.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/ClearingHouse.sol) [🖥](#nowhere "Uses Assembly") [🧮](#nowhere "Uses Hash-Functions") [Σ](#nowhere "Unchecked Blocks")|[181](#nowhere "(nSLOC:137, SLOC:181, Lines:232)")|ClearingHouses are deployed for each new loan and settle payments between Seaport auctions and Astaria Vaults if a liquidation occurs. It also stores NFTs that borrowers deposit to take out a loan.|  `solmate/*` `clones-with-immutable-args/*` `seaport/*`|
|[src/WithdrawProxy.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/WithdrawProxy.sol) [🖥](#nowhere "Uses Assembly") [🧮](#nowhere "Uses Hash-Functions") [Σ](#nowhere "Unchecked Blocks")|[234](#nowhere "(nSLOC:166, SLOC:234, Lines:318)")|A new WithdrawProxy is deployed for each PublicVault when at least one LP wants to withdraw by the end of the next epoch. It handles funds from loan repayments and auction funds.| `solmate/*` `gpl/*` |
|[src/CollateralToken.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/CollateralToken.sol) [🖥](#nowhere "Uses Assembly") [🧪](#nowhere "Experimental Features") [🧮](#nowhere "Uses Hash-Functions") [Σ](#nowhere "Unchecked Blocks")|[499](#nowhere "(nSLOC:426, SLOC:499, Lines:608)")|CollateralTokens are ERC721 certificates of deposit for NFTs being borrowed against on Astaria, giving the owner the right to the underlying asset when all debt is paid off.|  `solmate/*` `gpl/*` `seaport/*` `clones-with-immutable-args/*`|
|[src/PublicVault.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/PublicVault.sol) [🖥](#nowhere "Uses Assembly") [🧮](#nowhere "Uses Hash-Functions") [Σ](#nowhere "Unchecked Blocks")|[552](#nowhere "(nSLOC:442, SLOC:552, Lines:725)")|Contract for permissionless-lending Vaults, handling liquidations and withdrawals according to the epoch system.| `solmate/*` `gpl/*`  `clones-with-immutable-args/*`|
|[src/AstariaRouter.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/AstariaRouter.sol) [🖥](#nowhere "Uses Assembly") [💰](#nowhere "Payable Functions") [🧮](#nowhere "Uses Hash-Functions") [Σ](#nowhere "Unchecked Blocks")|[677](#nowhere "(nSLOC:533, SLOC:677, Lines:803)")|A router contract for handling universal protocol behavior and endpoints into other core contracts.| `solmate/*`  `gpl/*` `clones-with-immutable-args/*` `seaport/*`|
|[src/LienToken.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/LienToken.sol) [🖥](#nowhere "Uses Assembly") [🧪](#nowhere "Experimental Features") [🧮](#nowhere "Uses Hash-Functions") [Σ](#nowhere "Unchecked Blocks")|[773](#nowhere "(nSLOC:595, SLOC:773, Lines:919)")|LienTokens are non-fungible tokenized debt owned by Vaults. This contract handles the accounting and liquidation of loans throughout their lifecycle.| `solmate/*` `gpl/*` |
|_Abstracts (8)_|
|[src/WithdrawVaultBase.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/WithdrawVaultBase.sol)|[25](#nowhere "(nSLOC:25, SLOC:25, Lines:45)")|Base contract for WithdrawProxy.|  `clones-with-immutable-args/*`|
|[src/AstariaVaultBase.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/AstariaVaultBase.sol)|[41](#nowhere "(nSLOC:35, SLOC:41, Lines:65)")|Contract with pointers to vault constants and contract implementations.|  `clones-with-immutable-args/*`|
|[lib/gpl/src/ERC4626Router.sol](https://github.com/AstariaXYZ/astaria-gpl/blob/4b49fe993d9b807fe68b3421ee7f2fe91267c9ef/src/ERC4626Router.sol) [💰](#nowhere "Payable Functions")|[46](#nowhere "(nSLOC:29, SLOC:46, Lines:60)")|ERC4626 router contract.| `gpl/*`  `solmate/*`|
|[lib/gpl/src/ERC4626RouterBase.sol](https://github.com/AstariaXYZ/astaria-gpl/blob/4b49fe993d9b807fe68b3421ee7f2fe91267c9ef/src/ERC4626RouterBase.sol) [💰](#nowhere "Payable Functions")|[53](#nowhere "(nSLOC:33, SLOC:53, Lines:67)")|ERC4626 base router contract.|  `gpl/*` `solmate/*`|
|[lib/gpl/src/ERC4626-Cloned.sol](https://github.com/AstariaXYZ/astaria-gpl/blob/4b49fe993d9b807fe68b3421ee7f2fe91267c9ef/src/ERC4626-Cloned.sol)|[119](#nowhere "(nSLOC:96, SLOC:119, Lines:168)")|Custom base ERC4626 implementation.| `solmate/*` `clones-with-immutable-args/*` `gpl/*` |
|[lib/gpl/src/ERC20-Cloned.sol](https://github.com/AstariaXYZ/astaria-gpl/blob/4b49fe993d9b807fe68b3421ee7f2fe91267c9ef/src/ERC20-Cloned.sol) [🖥](#nowhere "Uses Assembly") [🧮](#nowhere "Uses Hash-Functions") [🔖](#nowhere "Handles Signatures: ecrecover") [Σ](#nowhere "Unchecked Blocks")|[146](#nowhere "(nSLOC:126, SLOC:146, Lines:197)")|Custom base ERC20 implementation.| |
|[lib/gpl/src/ERC721.sol](https://github.com/AstariaXYZ/astaria-gpl/blob/4b49fe993d9b807fe68b3421ee7f2fe91267c9ef/src/ERC721.sol) [🖥](#nowhere "Uses Assembly") [🧮](#nowhere "Uses Hash-Functions") [Σ](#nowhere "Unchecked Blocks")|[198](#nowhere "(nSLOC:163, SLOC:198, Lines:285)")|Slightly modified base ERC721 implementation.| |
|[src/VaultImplementation.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/VaultImplementation.sol) [🖥](#nowhere "Uses Assembly") [🧮](#nowhere "Uses Hash-Functions") [🔖](#nowhere "Handles Signatures: ecrecover") [Σ](#nowhere "Unchecked Blocks")|[294](#nowhere "(nSLOC:241, SLOC:294, Lines:410)")|Base vault contract with behavior for validating and issuing loan terms.| `solmate/*`  `gpl/*`|
|_Interfaces (28)_|
|[src/interfaces/IBeacon.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/interfaces/IBeacon.sol)|[4](#nowhere "(nSLOC:4, SLOC:4, Lines:23)")|Beacon contract interface.||
|[src/interfaces/IERC165.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/interfaces/IERC165.sol)|[4](#nowhere "(nSLOC:4, SLOC:4, Lines:25)")|ERC165 interface.||
|[src/interfaces/ISecurityHook.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/interfaces/ISecurityHook.sol)|[4](#nowhere "(nSLOC:4, SLOC:4, Lines:18)")|SecurityHook interface.||
|[lib/gpl/src/interfaces/IMulticall.sol](https://github.com/AstariaXYZ/astaria-gpl/blob/4b49fe993d9b807fe68b3421ee7f2fe91267c9ef/src/interfaces/IMulticall.sol) [💰](#nowhere "Payable Functions")|[6](#nowhere "(nSLOC:4, SLOC:6, Lines:16)")|Multicall interface.||
|[lib/gpl/src/interfaces/IWETH9.sol](https://github.com/AstariaXYZ/astaria-gpl/blob/4b49fe993d9b807fe68b3421ee7f2fe91267c9ef/src/interfaces/IWETH9.sol) [💰](#nowhere "Payable Functions")|[6](#nowhere "(nSLOC:6, SLOC:6, Lines:12)")|ERC4626-compliant AstariaRouter base contract interface.||
|[src/interfaces/IRouterBase.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/interfaces/IRouterBase.sol)|[6](#nowhere "(nSLOC:6, SLOC:6, Lines:22)")|AstariaRouter base contract interface.| |
|[src/interfaces/IERC20Metadata.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/interfaces/IERC20Metadata.sol)|[7](#nowhere "(nSLOC:7, SLOC:7, Lines:19)")|ERC20 metadata interface.| |
|[src/interfaces/IERC721Receiver.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/interfaces/IERC721Receiver.sol)|[9](#nowhere "(nSLOC:4, SLOC:9, Lines:27)")|ERC721Receiver interface.||
|[src/interfaces/ITransferProxy.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/interfaces/ITransferProxy.sol)|[9](#nowhere "(nSLOC:4, SLOC:9, Lines:23)")|TransferProxy interface.||
|[src/interfaces/IFlashAction.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/interfaces/IFlashAction.sol)|[11](#nowhere "(nSLOC:9, SLOC:11, Lines:26)")|FlashAction interface. Anyone may create a contract implementing this interface to define an action performed by a collateralized NFT in a flash loan.||
|[src/interfaces/IStrategyValidator.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/interfaces/IStrategyValidator.sol)|[11](#nowhere "(nSLOC:6, SLOC:11, Lines:26)")|StrategyValidator interface.| |
|[src/interfaces/IAstariaVaultBase.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/interfaces/IAstariaVaultBase.sol)|[12](#nowhere "(nSLOC:12, SLOC:12, Lines:32)")|Vault base contract interface.| |
|[src/interfaces/IERC1155Receiver.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/interfaces/IERC1155Receiver.sol)|[18](#nowhere "(nSLOC:6, SLOC:18, Lines:58)")|ERC1155Receiver interface.| |
|[src/interfaces/IERC20.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/interfaces/IERC20.sol)|[18](#nowhere "(nSLOC:11, SLOC:18, Lines:85)")|ERC20 interface.||
|[lib/gpl/src/interfaces/IERC4626Router.sol](https://github.com/AstariaXYZ/astaria-gpl/blob/4b49fe993d9b807fe68b3421ee7f2fe91267c9ef/src/interfaces/IERC4626Router.sol) [💰](#nowhere "Payable Functions")|[20](#nowhere "(nSLOC:7, SLOC:20, Lines:58)")|ERC4626-compliant AstariaRouter main contract interface.| |
|[src/interfaces/IWithdrawProxy.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/interfaces/IWithdrawProxy.sol)|[23](#nowhere "(nSLOC:18, SLOC:23, Lines:79)")|WithdrawProxy interface.| |
|[lib/gpl/src/interfaces/IUniswapV3Factory.sol](https://github.com/AstariaXYZ/astaria-gpl/blob/4b49fe993d9b807fe68b3421ee7f2fe91267c9ef/src/interfaces/IUniswapV3Factory.sol)|[26](#nowhere "(nSLOC:18, SLOC:26, Lines:78)")|Uniswap V3 factory contract interface.||
|[lib/gpl/src/interfaces/IERC4626RouterBase.sol](https://github.com/AstariaXYZ/astaria-gpl/blob/4b49fe993d9b807fe68b3421ee7f2fe91267c9ef/src/interfaces/IERC4626RouterBase.sol) [💰](#nowhere "Payable Functions")|[32](#nowhere "(nSLOC:12, SLOC:32, Lines:105)")|ERC4626-compliant AstariaRouter base contract interface.| |
|[src/interfaces/IERC721.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/interfaces/IERC721.sol)|[36](#nowhere "(nSLOC:23, SLOC:36, Lines:48)")|ERC721 interface.| |
|[src/interfaces/IERC1155.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/interfaces/IERC1155.sol)|[51](#nowhere "(nSLOC:30, SLOC:51, Lines:141)")|SecurityHook interface.| |
|[lib/gpl/src/interfaces/IUniswapV3PoolState.sol](https://github.com/AstariaXYZ/astaria-gpl/blob/4b49fe993d9b807fe68b3421ee7f2fe91267c9ef/src/interfaces/IUniswapV3PoolState.sol)|[52](#nowhere "(nSLOC:12, SLOC:52, Lines:116)")|Uniswap V3 state contract interface.||
|[src/interfaces/IERC4626.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/interfaces/IERC4626.sol)|[61](#nowhere "(nSLOC:34, SLOC:61, Lines:263)")|ERC4626 interface.| |
|[src/interfaces/IVaultImplementation.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/interfaces/IVaultImplementation.sol)|[68](#nowhere "(nSLOC:52, SLOC:68, Lines:105)")|VaultImplementation interface.| |
|[src/interfaces/IPublicVault.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/interfaces/IPublicVault.sol)|[78](#nowhere "(nSLOC:68, SLOC:78, Lines:173)")|PublicVault interface.| |
|[src/interfaces/IV3PositionManager.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/interfaces/IV3PositionManager.sol) [💰](#nowhere "Payable Functions")|[98](#nowhere "(nSLOC:61, SLOC:98, Lines:177)")|Uniswap V3 PositionManager interface.||
|[src/interfaces/ICollateralToken.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/interfaces/ICollateralToken.sol)|[112](#nowhere "(nSLOC:103, SLOC:112, Lines:187)")|CollateralToken interface.|  `seaport/*`|
|[src/interfaces/IAstariaRouter.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/interfaces/IAstariaRouter.sol)|[195](#nowhere "(nSLOC:165, SLOC:195, Lines:342)")|AstariaRouter interface.|  `solmate/*` `gpl/*` `seaport/*`|
|[src/interfaces/ILienToken.sol](https://github.com/code-423n4/2023-01-astaria/blob/main/src/interfaces/ILienToken.sol)|[208](#nowhere "(nSLOC:151, SLOC:208, Lines:346)")|LienToken interface.| |
|Total (over 45 files):| [5136](#nowhere "(nSLOC:3970, SLOC:5136, Lines:7749)") | |

# Additional Context

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

Note that since this codebase uses Foundry remappings for imports, cmd/ctrl-click to jump to different files is not currently supported.

If you're using Slither, ensure you're using the latest version to avoid this sourceMap issue: crytic/crytic-compile#281.

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

Note that since ```testClaimFeesAgainstV3Liquidity()``` uses Goerli on-chain data, it will revert when running in a non-forked test environment. To run this forked test, run this with a Goerli RPC URL:

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

When fork testing testClaimFeesAgainstV3Liquidity use the following to ensure the state data is accurate  
```sh
forge test --ffi --match-test testClaimFeesAgainstV3Liquidity --fork-url <YOUR_RPC_HERE> --fork-block-number 15934974
```


Edge cases around withdrawing and other complex protocol usage are found in the WithdrawTest and IntegrationTest contracts.

To view a gas report, run:
```sh
forge snapshot --gas-report --ffi
```

When fork testing testClaimFeesAgainstV3Liquidity use the following to ensure the state data is accurate, USE A MAINNET ARCHIVE RPC  
```sh
forge test --ffi --match-test testClaimFeesAgainstV3Liquidity --fork-url <YOUR_RPC_HERE> --fork-block-number 15934974
```

# Quickstart

export FORK_URL="<your-mainnet-rpc-url-goes-here>" && rm -Rf 2023-01-astaria || true && git clone https://github.com/code-423n4/2023-01-astaria.git -j8 --recurse-submodules && cd 2023-01-astaria && nvm install 16.0 && foundryup && forge install && yarn install && forge test --ffi --fork-url $FORK_URL --fork-block-number 15934974 --gas-report
