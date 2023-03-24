usmannk

medium

# Transferring ownership might break the market

## Summary
This is identical to issue 41 in the previous Bond protocol audit: https://github.com/sherlock-audit/2022-11-bond-judging/issues/41. The issue is unfixed.

When a bond is purchased in a market with a specified callback, it is checked that the market owner is a valid callback address. If the owner of a market is changed to a new owner who is not a valid callback address, it will cause an unexpected denial of service to bond purchasers.

## Vulnerability Detail

The `callbackAuthorized` mapping is checked on bond purchase. Specifically, the market owner is checked against this mapping. Changing the owner to an unauthorized new owner will cause the bond purchase operation to silently fail.

https://github.com/sherlock-audit/2023-02-bond/blob/main/bonds/src/bases/BondBaseFPA.sol#L272-L273

## Impact

Unexpected denial of service to bond markets. It is not easily recoverable as the new owner is not able to authorize themselves.

## Code Snippet

## Tool used

Manual Review

## Recommendation

For markets with callbacks, verify the authorization of the new owner when ownership is changed.
