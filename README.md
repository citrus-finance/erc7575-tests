# ERC7575 Property Tests

Foundry (dapptools-style) property-based tests for [ERC7575] standard conformance.

[ERC7575]: <https://eips.ethereum.org/EIPS/eip-7575>

Forked from https://github.com/a16z/erc4626-tests/

## Overview

#### What is it?
- Test suites for checking if the given ERC7575 implementation satisfies the **standard requirements**.
- Dapptools-style **property-based tests** for fuzzing or symbolic execution testing.
- Tests that are **independent** from implementation details, thus applicable for any ERC7575 vaults.

#### What isn’t it?
- It does NOT test implementation-specific details, e.g., how to generate and distribute yields, how to compute the share price, etc.

#### Testing properties:

- **Round-trip properties**: no one can make a free profit by depositing and immediately withdrawing back and forth.

- **Functional correctness**: the `deposit()`, `mint()`, `withdraw()`, and `redeem()` functions update the balance and allowance properly.

- The `preview{Deposit,Redeem}()` functions **MUST NOT over-estimate** the exact amount.[^1]

[^1]: That is, the `deposit()` and `redeem()` functions “MUST return the same or more amounts as their preview function if called in the same transaction.”

- The `preview{Mint,Withdraw}()` functions **MUST NOT under-estimate** the exact amount.[^2]

[^2]: That is, the `mint()` and `withdraw()` functions “MUST return the same or fewer amounts as their preview function if called in the same transaction.”

- The `convertTo{Shares,Assets}` functions “**MUST NOT show any variations** depending on the caller.”

- The `asset()`, `share()`, `totalAssets()`, and `max{Deposit,Mint,Withdraw,Redeem}()` functions “**MUST NOT revert**.”

## Usage

**Step 0**: Install [foundry] and add [forge-std] in your vault repo:
```bash
$ curl -L https://foundry.paradigm.xyz | bash

$ cd /path/to/your-erc7575-vault
$ forge install foundry-rs/forge-std
```

[foundry]: <https://getfoundry.sh/>
[forge-std]: <https://github.com/foundry-rs/forge-std>

**Step 1**: Add this [erc7575-tests] as a dependency to your vault:
```bash
$ cd /path/to/your-erc7575-vault
$ forge install a16z/erc7575-tests
```

[erc7575-tests]: <https://github.com/citrus-finance/erc7575-tests>

**Step 2**: Extend the abstract test contract [`ERC7575Test`](ERC7575.test.sol) with your own custom vault setup method, for example:

```solidity
// SPDX-License-Identifier: AGPL-3.0
pragma solidity >=0.8.0 <0.9.0;

import "erc7575-tests/ERC7575.test.sol";

import { ERC20Mock   } from "/path/to/mocks/ERC20Mock.sol";
import { ERC7575Mock } from "/path/to/mocks/ERC7575Mock.sol";

contract ERC7575StdTest is ERC7575Test {
    function setUp() public override {
        _underlying_ = address(new ERC20Mock("Mock ERC20", "MERC20", 18));
        _share_ = address(new ERC20Mock("Mock Share", "MSHR", 18));
        _vault_ = address(new ERC7575Mock(ERC20Mock(__underlying__), ERC20Mock(_share_)));
        _delta_ = 0;
        _vaultMayBeEmpty = false;
        _unlimitedAmount = false;
    }
}
```

Specifically, set the state variables as follows:
- `_vault_`: the address of your ERC7575 vault.
- `_share_`: the address of your Share.
- `_underlying_`: the address of the underlying asset of your vault. Note that the default `setupVault()` and `setupYield()` methods of `ERC7575Test` assume that it implements `mint(address to, uint value)` and `burn(address from, uint value)`. You can override the setup methods with your own if such `mint()` and `burn()` are not implemented.
- `_delta_`: the maximum approximation error size to be passed to [`assertApproxEqAbs()`]. It must be given as an absolute value (not a percentage) in the smallest unit (e.g., Wei or Satoshi). Note that all the tests are expected to pass with `__delta__ == 0` as long as your vault follows the [preferred rounding direction] as specified in the standard. If your vault doesn't follow the preferred rounding direction, you can set `__delta__` to a reasonable size of rounding errors where the adversarial profit of exploiting such rounding errors stays sufficiently small compared to the gas cost. (You can read our [post] for more about the adversarial profit.)
- `_vaultMayBeEmpty`: when set to false, fuzz inputs that empties the vault are ignored.
- `_unlimitedAmount`: when set to false, fuzz inputs are restricted to the currently available amount from the caller. Limiting the amount can speed up fuzzing, but may miss some edge cases.

[`assertApproxEqAbs()`]: <https://book.getfoundry.sh/reference/forge-std/assertApproxEqAbs>

[preferred rounding direction]: <https://eips.ethereum.org/EIPS/eip-4626#security-considerations>

**Step 3**: Run `forge test`

```
$ forge test
```

## Examples

Below are examples of adding these property tests to existing ERC7575 vaults:


## Disclaimer

_These smart contracts are being provided as is. No guarantee, representation or warranty is being made, express or implied, as to the safety or correctness of the user interface or the smart contracts. They have not been audited and as such there can be no assurance they will work as intended, and users may experience delays, failures, errors, omissions or loss of transmitted information. THE SMART CONTRACTS CONTAINED HEREIN ARE FURNISHED AS IS, WHERE IS, WITH ALL FAULTS AND WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING ANY WARRANTY OF MERCHANTABILITY, NON-INFRINGEMENT OR FITNESS FOR ANY PARTICULAR PURPOSE. Further, use of any of these smart contracts may be restricted or prohibited under applicable law, including securities laws, and it is therefore strongly advised for you to contact a reputable attorney in any jurisdiction where these smart contracts may be accessible for any questions or concerns with respect thereto. Further, no information provided in this repo should be construed as investment advice or legal advice for any particular facts or circumstances, and is not meant to replace competent counsel. a16z is not liable for any use of the foregoing, and users should proceed with caution and use at their own risk. See a16z.com/disclosures for more info._
