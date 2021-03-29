## Genesis Parameters

This document is derived from the quantities first described in the release of the [Cosmos-Hub](https://github.com/cosmos/mainnet/blob/master/params/README.md). 

Here the more subjective parameter choices are documented with the reasons behind their recommendation. Text in *italics*,
in the document below, represents justifications from [Fetch.ai](https://fetch.ai) for either persisting with the default Cosmos parameters 
or for modifications. 

Note that all durations are specified in nanoseconds.

### Staking Module

- `"unbonding_time": "1814400000000000"`. The unbonding time determines the duration for which bonded stake is
  held accountable for any discovered equivocations, specified in nanoseconds. 3 weeks was chosen to balance
  the concerns of a sufficient unbonding period for light-client safety and a modicum of staking token liquidity.
  *This bonding period has proven successful for the Cosmos-hub and many other projects. The Fetch.ai staking
    program has an identical bonding period. The quantity of tokens staked indicates that this incentive design provides
    the network with sufficient crypto-economic security.*

- `"max_validators": "50"`. The maximum validator count is the total number of validators which can be bonded
  and voting in consensus for any given block - which validators are in this set is dynamically determined
  to be the top hundred validator candidates sorted by delegated stake.
  *The Fetch.ai validator set has been deliberately restricted compared with Cosmos (which has 100)  to ensure
    that validator nodes are not operated at a loss by the community. The Fetch.ai random beacon provides greater 
    security but potentially at greater communication cost. This will be mitigated in the future by using a 
    a light-node, full-node consensus as specified in the minimal agency consensus whitepaper to increase the 
    decentralization of the network without compromising throughput.* 

### Minting Module

- `"inflation": "0.03"`. *The Fetch.ai network has a per annum issuance rate of 3%. The Fetch.ai staking program 
  has shown that block rewards at this level are sufficient to incentivise the operation of a secure network. This 
  issuance rate is also capable of offsetting the reduction in the true token supply caused by lost keys and 
  transaction "dust" (i.e. small residual token amounts that are uneconomic to transfer).*

- *The Fetch.ai network has a fixed issuance rate as previous experience has shown that network security
  can be achieved with a similar reward profile. A potential problem with variable rewards is that a loss of confidence
  in the project could leads to a reduction in the quantity of staked tokens which then triggers higher issuance. 
  This could lead to a positive feed-back loop that accelerates further loss of confidence.*

### Distribution Module

- `"community_tax": "0.02"`. The tax on inflation and fees levied to fund the public goods pool will be 2%. *These
   funds will be used for the operation of the Fetch.ai foundation and the funding of projects to further develop the
   ecosystem.*
   
- `"base_proposer_reward": "0.01"`. 1% of inflation and fees (flat) will be allocated to the block proposer. This 
    provides an incentive for validators to be good proposers by being available when it's their turn to propose, 
    including lots of transactions in their proposed block, and gossiping the proposed block quickly.
- `"bonus_proposer_reward": "0.04"`. 4% of inflation and fees (varying according to the fraction of precommits included)
    will be allocated to the block proposer to incentivize them to include as many precommits from other validators as 
    possible.
- `"withdraw_addr_enabled": false`. Changing reward withdrawal addresses will be initially disabled. *We see no reason 
    to enable this capability.* 

### Governance Module

- `"min_deposit": 2048FET`. The minimum deposit to bring a proposal up for a vote is 2048FET. This value should be high 
   enough to prevent spam proposals, while not being too expensive. As a note, the deposit can be crowd-funded, so the
   proposer doesn't have to provide the whole thing. All deposits are refunded for proposals that are passed. *This 
   value is consistent with the cost of proposals at the launch of the Cosmos-hub.*
- `"max_deposit_period": "1209600000000000"`. The duration in which a proposal can collect deposits is 14 days. This
   value should be long enough for a proposal to have time to gain support from the community.
- `"voting_period": "1209600000000000"`. The duration in which a proposal can be voted upon is 14 days. The voting 
   period should be long enough that all staked FET token holders have time to participate.
- `"quorum": "0.4"`. A minimum quorum of 40% of bonded stake must vote on a proposal in order for it to be considered
   for passage. This is to ensure that proposals don't pass that have support from only a small segment of the community.
- `"threshold": "0.5"`. Over half the voting stake must vote in favor of a proposal in order for it to pass.
- `"veto": "0.334"`. 1/3 of voting stake vetoing a proposal prevents it from passing. This is necessitated by the 1/3 
  BFT safety bound, since 1/3 of stake could also elect to halt the chain or compromise safety.

### Slashing Module

- `"max_evidence_age": "1814400000000000"`. The maximum age of evidence possibly considered valid is 3 weeks
  (it must be the same as the unbonding period).
- `"signed_blocks_window": "10000"`. The rolling window for uptime measurement is 10,000 blocks. *For a block time of 6 
  seconds this amounts to around 16 hours. A validator can be unavailable for 4 hours and easily reach the threshold of 
  5% of blocks signed.*
- `"min_signed_per_window": "0.05"`. A minimum of 5% of the blocks in the last window must have been signed or
  else a validator will be slashed for downtime. To nurture network launch, a lenient uptime requirement is recommended 
  that can later be increased by governance.
- `"downtime_jail_duration": "600000000000"`. Validators slashed for downtime are jailed for ten minutes. This provides
  a disincentive for validator downtime.
- `"slash_fraction_double_sign": "0.005"`. Validators who equivocate (double-sign a block, and thereby compromise 
  safety) and are caught are slashed by 0.5% of their bonded stake. *Since this behaviour results in the validator being 
  immediately removed from the consensus, security is still maintained. Cosmos' penalty of 5% seems harsh for an action
  that is more likely to be an error than an attack.*
- `"slash_fraction_downtime": "0.0001"`. Validators who are slashed for downtime and thereby compromise the availability
  of the network are slashed by 0.01% of their bonded stake. This is to provide additional disincentive for validator
  downtime.

