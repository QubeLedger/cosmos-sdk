<!--
order: 8
-->

# Parameters

The staking module contains the following parameters:

| Key                       | Type             | Example           |
|---------------------------|------------------|-------------------|
| UnbondingTime             | string (time ns) | "259200000000000" |
| MaxValidators             | uint16           | 100               |
| KeyMaxEntries             | uint16           | 7                 |
| HistoricalEntries         | uint16           | 3                 |
| BondDenom                 | string           | "stake"           |
| PowerReduction            | string           | "1000000"         |
| ValidatorBondFactor       | string           | "250"             |
| globalLiquidStakingCap    | string           | "100"             |
| validatorLiquidStakingCap | string           | "100"             |