<!--
order: 4
-->

# End-Block

Each abci end block call, the operations to update queues and validator set
changes are specified to execute.

## Validator Set Changes

The staking validator set is updated during this process by state transitions
that run at the end of every block. As a part of this process any updated
validators are also returned back to Tendermint for inclusion in the Tendermint
validator set which is responsible for validating Tendermint messages at the
consensus layer. Operations are as following:

- the new validator set is taken as the top `params.MaxValidators` number of
  validators retrieved from the `ValidatorsByPower` index
- the previous validator set is compared with the new validator set:
    - missing validators begin unbonding and their `Tokens` are transferred from the
    `BondedPool` to the `NotBondedPool` `ModuleAccount`
    - new validators are instantly bonded and their `Tokens` are transferred from the
    `NotBondedPool` to the `BondedPool` `ModuleAccount`

In all cases, any validators leaving or entering the bonded validator set or
changing balances and staying within the bonded validator set incur an update
message reporting their new consensus power which is passed back to Tendermint.

The `LastTotalPower` and `LastValidatorsPower` hold the state of the total power
and validator power from the end of the last block, and are used to check for
changes that have occured in `ValidatorsByPower` and the total new power, which
is calculated during `EndBlock`.

## Queues

Within staking, certain state-transitions are not instantaneous but take place
over a duration of time (typically the unbonding period). We refer to these 
transitions as unbonding operations. When these transitions are mature certain 
operations must take place in order to complete the unbonding operation. 
This is achieved through the use of queues which are checked/processed at the 
end of each block.

### Unbonding Validators

When a validator is kicked out of the bonded validator set (either through
being jailed, or not having sufficient bonded tokens) it begins the unbonding
process along with all its delegations begin unbonding (while still being
delegated to this validator). At this point the validator is said to be an
"unbonding validator", whereby it will mature to become an "unbonded validator"
after the unbonding period has passed.

Each block the validator queue is to be checked for mature unbonding validators
(namely with a completion time <= current time and completion height <= current
block height). At this point any mature validators which do not have any
delegations remaining are deleted from state. For all other mature unbonding
validators that still have remaining delegations, the `validator.Status` is
switched from `types.Unbonding` to
`types.Unbonded`.

### Unbonding Delegations

Complete the unbonding of all mature `UnbondingDelegations.Entries` within the
`UnbondingDelegations` queue with the following procedure:

- transfer the balance coins to the delegator's wallet address
- remove the mature entry from `UnbondingDelegation.Entries`
- remove the `UnbondingDelegation` object from the store if there are no
  remaining entries.

### Redelegations

Complete the unbonding of all mature `Redelegation.Entries` within the
`Redelegations` queue with the following procedure:

- remove the mature entry from `Redelegation.Entries`
- remove the `Redelegation` object from the store if there are no
  remaining entries.

## Putting unbonding operations on hold

Unbonding operations can be put on hold by external modules via the `PutUnbondingOnHold(unbondingId)` method. 
As a result, an unbonding operation (e.g., an unbonding delegation) that is on hold, cannot complete 
even if it reaches maturity. For an unbonding operation with `unbondingId` to eventually complete 
(after it reaches maturity), every call to `PutUnbondingOnHold(unbondingId)` must be matched 
by a call to `UnbondingCanComplete(unbondingId)`. 