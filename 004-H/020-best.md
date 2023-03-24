xiaoming90

high

# Incorrect market price returned from `BondBaseFPA.marketPrice` function (FPA Only)

## Summary

The `BondBaseFPA.marketPrice` was not aligned with the specification and returned an incorrect market price.

## Vulnerability Detail

The `MarketParams.formattedPrice` is the price ratio of payout tokens per quote token (P/Q) as per Line 31 below. This means the number of payout tokens for every single quote token (e.g. Payout tokens/Quote token)

```solidity
File: IBondFPA.sol
29:     /// @notice             Parameters to create a new bond market
30:     /// @dev                Note price should be passed in a specific format:
31:     ///                     formatted price = (payoutPriceCoefficient / quotePriceCoefficient)
32:     ///                             * 10**(36 + scaleAdjustment + quoteDecimals - payoutDecimals + payoutPriceDecimals - quotePriceDecimals)
33:     ///                     where:
34:     ///                         payoutDecimals - Number of decimals defined for the payoutToken in its ERC20 contract
35:     ///                         quoteDecimals - Number of decimals defined for the quoteToken in its ERC20 contract
36:     ///                         payoutPriceCoefficient - The coefficient of the payoutToken price in scientific notation (also known as the significant digits)
37:     ///                         payoutPriceDecimals - The significand of the payoutToken price in scientific notation (also known as the base ten exponent)
38:     ///                         quotePriceCoefficient - The coefficient of the quoteToken price in scientific notation (also known as the significant digits)
39:     ///                         quotePriceDecimals - The significand of the quoteToken price in scientific notation (also known as the base ten exponent)
40:     ///                         scaleAdjustment - see below
41:     ///                         * In the above definitions, the "prices" need to have the same unit of account (i.e. both in OHM, $, ETH, etc.)
42:     ///                         If price is not provided in this format, the market will not behave as intended.
..SNIP..
49:     /// @dev                    5. Formatted price (see note above)
..SNIP..
58:     struct MarketParams {
59:         ERC20 payoutToken;
60:         ERC20 quoteToken;
61:         address callbackAddr;
62:         bool capacityInQuote;
63:         uint256 capacity;
64:         uint256 formattedPrice;
65:         uint48 vesting;
66:         uint48 conclusion;
67:         uint48 depositInterval;
68:         int8 scaleAdjustment;
69:     }
```

Per the Bond whitepaper, the market price should be the ratio of quote tokens per payout token (Q/P) instead.

![image-20230310125340939](C:\Users\cme-dell\OneDrive\sherlock\Bond (Oracle)\Reports\Completed\assets\image-20230310125340939.png)

It is also mentioned that the market price should be the ratio of quote tokens per payout token (Q/P) in the interface file.

```solidity
File: IBondFPA.sol
81:     /* ========== VIEW FUNCTIONS ========== */
82: 
83:     /// @notice             Calculate current market price of payout token in quote tokens
84:     /// @param id_          ID of market
85:     /// @return             Price for market in configured decimals (see MarketParams)
86:     /// @dev price is derived from the equation:
87:     //
88:     // p = f_p
89:     //
90:     // where
91:     // p = price
92:     // f_p = fixed price provided on creation
93:     //
94:     function marketPrice(uint256 id_) external view override returns (uint256);
```

However, it was observed that the `BondBaseFPA.marketPrice` function was not aligned with the specification and returned an incorrect market price. The`BondBaseFPA.marketPrice` function returns the market price directly from the `market.price` variable, which is set to the `MarketParams.formattedPrice` during initialization. Recall that the `MarketParams.formattedPrice` is the price ratio of payout tokens per quote token (P/Q).

Thus, the `BondBaseFPA.marketPrice` function returns the price ratio of payout tokens per quote token (P/Q) instead of the ratio of quote tokens per payout token (Q/P).

```solidity
File: BondBaseFPA.sol
340:     function marketPrice(uint256 id_) public view override returns (uint256) {
341:         return markets[id_].price;
342:     }
```

## Impact

If any internal function within the protocol relies on the `BondBaseFPA.marketPrice` function, it will receive the wrong pricing, thus causing computation error, which could potentially lead to loss of assets if it uses the incorrect result to compute the number of payout tokens to be issued (e.g. less payout token being issued to users).

A number of functions (e.g. `maxAmountAccepted`, `maxPayout`, `payoutFor`) within FPA rely on this function to obtain the market price. As a result, these functions are broken.

In addition, since this is a public function, any external or third-party protocol that integrates with this function would also suffer the same problem.

## Code Snippet

https://github.com/sherlock-audit/2023-02-bond/blob/main/bonds/src/bases/BondBaseFPA.sol#L340

## Tool used

Manual Review

## Recommendation

Update the `BondBaseFPA.marketPrice` function to return quote tokens per payout token (Q/P) as per the specification.