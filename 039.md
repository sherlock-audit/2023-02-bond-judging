spyrosonic10

medium

# Possibility of underflow in calling `initiateBatchAuction`

## Summary
BondBatchAuctionV1.sol has a function `amountWithFee()` which calculate amount with fee from given input and this function has a logic to check fee like this  `_teller.protocolFee() - _teller.createFeeDiscount()`. There is no check to make sure `protocolFee` is higher than `createFeeDiscount` and hence has potential to fail if it is lower.

## Vulnerability Detail
There are multiple instance where code does check `if (protocolFee > createFeeDiscount)` which indicate that it is possible for `protocolFee` to be lower than createFeeDiscount and in such cases failure will occur. Fee and discount are set in BondBaseTeller(which is not in scope) and it does suggest that there is no safe check in place during setting `setProtocolFee` which also validate possible existence of vulnerability.

## Impact
If `protocolFee` is lower than `createFeeDiscount` then `amountWithFee` will fail and which cause `initiateBatchAuction` to fail.

## Code Snippet
https://github.com/sherlock-audit/2023-02-bond/blob/main/bonds/src/BondBatchAuctionV1.sol#L306-L321

## Tool used

Manual Review

## Recommendation
Consider adding `if (protocolFee > createFeeDiscount)` before performing subtraction.
