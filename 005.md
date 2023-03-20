R-Nemes

medium

# Use of ERC20 approve without resetting first

## Summary
It is recommended to reduce the spender allowance to zero before each approve(<SPENDER>, <AMOUNT>) call.

## Vulnerability Detail
`payoutToken.approve(
                address(gnosisAuction()),
                (uint256(batchAuctionParams_.auctionAmount) *
                    (gnosisAuction().feeNumerator() + feeDecimals)) / feeDecimals
            );` function is called without setting the allowance to zero. Some tokens, like USDT, require first reducing the address' allowance to zero by calling approve(_spender, 0).

## Impact
Some tokens, like USDT, require first reducing the address' allowance to zero by calling approve(_spender, 0). Transactions will revert when using an unsupported token like USDT

## Code Snippet
[bonds/src/BondBatchAuctionV1.sol#L167-L171](https://github.com/sherlock-audit/2023-02-bond/blob/main/bonds/src/BondBatchAuctionV1.sol#L167-L171)

## Tool used

Manual review

## Recommendation
Reset the approval to zero first

```diff
diff --git a/bonds/src/BondBatchAuctionV1.sol b/bonds/src/BondBatchAuctionV1.sol
index faf3d49..12932ea 100644
--- a/bonds/src/BondBatchAuctionV1.sol
+++ b/bonds/src/BondBatchAuctionV1.sol
@@ -164,6 +164,7 @@ contract BondBatchAuctionV1 is IBondBatchAuctionV1, ReentrancyGuard, Clone {
             // Approve auction contract for bond tokens (transferred immediately after so no approval override issues)
             // We include the gnosis fee amount in the approval since initiateAuction will transfer this amount
             uint256 feeDecimals = gnosisAuction().FEE_DENOMINATOR();
+            payoutToken.approve(address(gnosisAuction()), 0);
             payoutToken.approve(
                 address(gnosisAuction()),
                 (uint256(batchAuctionParams_.auctionAmount) *

```