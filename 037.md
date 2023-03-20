Bauer

medium

# Use call() instead of transfer() on an address payable

## Summary
The use of the deprecated transfer() function for an payable address will inevitably make the transaction

## Vulnerability Detail
The transfer() and send() functions forward a fixed amount of 2300 gas. Historically, it has often been recommended to use these functions for value transfers to guard against reentrancy attacks. However, the gas cost of EVM instructions may change significantly during hard forks which may break already deployed contract systems that make fixed assumptions about gas costs. For example. EIP 1884 broke several existing smart contracts due to a cost increase of the SLOAD instruction.

```solidity

    function emergencyWithdraw(ERC20 token_) external override onlyOwner {
        // Confirm that the token is a contract
        if (address(token_).code.length == 0 && address(token_) != address(0))
            revert BatchAuction_InvalidParams();

        // If token address is zero, withdraw ETH
        if (address(token_) == address(0)) {
            payable(owner()).transfer(address(this).balance);
        } else {
            token_.safeTransfer(owner(), token_.balanceOf(address(this)));
        }
    }

```
## Impact
The use of the deprecated transfer() function for an address will inevitably make the transaction fail when:

The claimer smart contract does not implement a payable function.
The claimer smart contract does implement a payable fallback which uses more than 2300 gas unit.
The claimer smart contract implements a payable fallback function that needs less than 2300 gas units but is called through proxy, raising the call's gas usage above 2300.
Additionally, using higher than 2300 gas might be mandatory for some multisig wallets.

## Code Snippet
https://github.com/sherlock-audit/2023-02-bond/blob/main/bonds/src/BondBatchAuctionV1.sol#L287

## Tool used

Manual Review

## Recommendation
Use call() instead of transfer(), but be sure to respect the CEI pattern and/or add re-entrancy guards, as several hacks already happened in the past due to this recommendation not being fully understood.

More info on; https://swcregistry.io/docs/SWC-134
