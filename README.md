
# Optimism 2024 contest details

- Join [Sherlock Discord](https://discord.gg/MABEWyASkp)
- Submit findings using the issue page in your private contest repo (label issues as med or high)
- [Read for more details](https://docs.sherlock.xyz/audits/watsons)

# Q&A

### Q: On what chains are the smart contracts going to be deployed?
Ethereum Mainnet.
___

### Q: If you are integrating tokens, are you allowing only whitelisted tokens to work with the codebase or any complying with the standard? Are they assumed to have certain properties, e.g. be non-reentrant? Are there any types of <a href="https://github.com/d-xo/weird-erc20" target="_blank" rel="noopener noreferrer">weird tokens</a> you want to integrate?
Of the contracts that are in scope for this contest, the only token integrated directly is a modified version of WETH9 called DelayedWETH. The DelayedWETH contract is directly in the scope of this contest.
___

### Q: Are the admins of the protocols your contracts integrate with (if any) TRUSTED or RESTRICTED? If these integrations are trusted, should auditors also assume they are always responsive, for example, are oracles trusted to provide non-stale information, or VRF providers to respond within a designated timeframe?
N/A (no external integrations).
___

### Q: Are there any protocol roles? Please list them and provide whether they are TRUSTED or RESTRICTED, or provide a more comprehensive description of what a role can and can't do/impact.
Two roles exist for contracts in scope of this contest.

The Proxy Admin Owner is a TRUSTED role that can:
- Upgrade all smart contracts that sit behind a Proxy contract.
- Set the implementation contract for any dispute game type within the DisputeGameFactory.
- Modify the initial bond cost for any dispute game type within the DisputeGameFactory.
- Remove ETH from the DelayedWETH contract.

The Proxy Admin Owner is assumed to be honest and responsive with an SLA of 72 hours.

The Guardian is a RESTRICTED role that can ONLY be permitted to:
- Trigger the SuperchainConfig.pause function to pause withdrawals from several system contracts.
- Trigger the SuperchainConfig.unpause function to unpause withdrawals from several system contracts.
- Trigger the OptimismPortal2.blacklistDisputeGame function to block a specific FaultDisputeGame for withdrawals.
- Trigger the OptimismPortal2.setRespectedGameType function to change which game type can be used for withdrawals.

The Guardian must not be able to:
- Remove ETH from the DelayedWETH contract.
- Remove ETH from the OptimismPortal2 contract.
- Cause an invalid message to be executed by the OptimismPortal2 contract.

The Guardian is assumed to be honest and responsive with an SLA of 24 hours.
___

### Q: For permissioned functions, please list all checks and requirements that will be made before calling the function.
Permissioned functions can be used in any situation.
___

### Q: Is the codebase expected to comply with any EIPs? Can there be/are there any deviations from the specification?
N/A.
___

### Q: Are there any off-chain mechanisms or off-chain procedures for the protocol (keeper bots, arbitrage bots, etc.)?
Off-chain mechanisms exist as part of the system but are not in scope for this competition. Assume that comprehensive monitoring exists that will detect most obviously detectable malicious activity.
___

### Q: Are there any hardcoded values that you intend to change before (some) deployments?
N/A.
___

### Q: If the codebase is to be deployed on an L2, what should be the behavior of the protocol in case of sequencer issues (if applicable)? Should Sherlock assume that the Sequencer won't misbehave, including going offline?
N/A.
___

### Q: Should potential issues, like broken assumptions about function behavior, be reported if they could pose risks in future integrations, even if they might not be an issue in the context of the scope? If yes, can you elaborate on properties/invariants that should hold?
Yes, but this should be limited to the OptimismPortal2 contract. Contracts other than the OptimismPortal2 contract are not intended for external integrations and risks for future integrations into these contracts will likely not be considered valid.

For the OptimismPortal2, the following high-level invariants must hold:
- Users must not be able to execute messages that have not actually been sent and included in the state of the L2ToL1MessagePasser contract on the L2 blockchain.
- Users must be able to execute messages that have actually been sent and included in the state of the L2ToL1MessagePasser contract on the L2 blockchain. Users must not be permanently prevented from executing a valid message.
- Users must not be able to cause a transfer of ETH out of the OptimismPortal2 except during the execution of a valid message sent from the L2 blockchain.
- Users must not be able to execute the same message more than once.
- Users should not be able to modify the content, sender, or target of a message once it has been sent.
___

### Q: Please discuss any design choices you made.
- IMPORTANT NOTE: OptimismPortal2 is included in this contest but OptimismPortal is not. OptimismPortal is the current implementation of the contract that would be replaced to the new implementation defined in OptimismPortal2. Review OptimismPortal2, not OptimismPortal!

- Our design philosophy has been to focus on fundamental safety mechanisms first. We acknowledge that gaining certainty in the correctness of the complex logic found within the FaultDisputeGame contract, its dependencies, and the offchain op-challenger software will take time. Therefore, we have included a number of fallback mechanisms designed to maintain the safety of the system even in the event of a complete failure of those complex components. Our primary goal with this particular contest is to gain confidence in the correctness of these safety mechanisms so that we can safely work on gaining total confidence in the more complex components over time.

- DelayedWETH includes a mapping that associates attempts to withdraw bonded ETH with specific recipient addresses. However, these specific recipient addresses are not actually required to participate in the process of claiming these withdrawals from the DelayedWETH contract. The FaultDisputeGame is written to be restrained in such a way that this "soft" accounting (no real restriction that forces users to make use of the accounting system) is actually "hard" accounting and cannot be circumvented by users (other than by a bug in the FaultDisputeGame). This decision was made so that users would not have to interact with the DelayedWETH contract directly and the contract could therefore be removed at a later date without impacting challenger software.

- DelayedWETH and the DisputeGameFactory contract both have "owner" addresses. For simplicity, these owner addresses are set during the initialization process instead of being pulled from a specific contract to guarantee that they are the same as the Proxy Admin Owner. However, for the sake of this contest, you should assume that these owner addresses are the same as the Proxy Admin Owner.
___

### Q: Please list any known issues/acceptable risks that should not result in a valid finding.
- FaultDisputeGame resolution logic is not included in the scope of this contest. Participants should assume that the FaultDisputeGame can resolve incorrectly (i.e.g, can resolve to DEFENDER_WINS when it should resolve to CHALLENGER_WINS or vice versa). Reports that demonstrate an incorrect resolution of the FaultDisputeGame are appreciated but will not be considered valid rewardable findings for this specific contest. However, there should be no function in the FaultDisputeGame that can cause an airgap breach (up to High). Additionally there should be no function that can cause valid withdrawals to not be finalizable (at most Medium) except for the case that the “resolve” function sets the result to CHALLENGER_WINS (invalid proposal) when it should be DEFENDER_WINS.
___

### Q: We will report issues where the core protocol functionality is inaccessible for at least 7 days. Would you like to override this value?
N/A (no override).
___

### Q: Please provide links to previous audits (if any).
Of the contracts in scope for this contest, only the OptimismPortal contract has been previously audited.

References to audits that include the OptimismPortal can be found below:

- https://github.com/ethereum-optimism/optimism/blob/a039f5a7c1fadd680e95d53e315d8fca2cff39a6/docs/security-reviews/2022_05-Bedrock_Contracts-Zeppelin.pdf 
- https://github.com/ethereum-optimism/optimism/blob/a039f5a7c1fadd680e95d53e315d8fca2cff39a6/docs/security-reviews/2022_09-Bedrock_and_Periphery-Zeppelin.pdf
- https://github.com/ethereum-optimism/optimism/blob/a039f5a7c1fadd680e95d53e315d8fca2cff39a6/docs/security-reviews/2023_01-Bedrock_Updates-TrailOfBits.pdf
- https://github.com/sherlock-protocol/sherlock-reports/blob/61e3689eee4d74bcfedbb2a5b7bb60da6e76a707/audits/2023.03.03%20-%20Final%20-%20Optimism%20Bedrock%20Audit%20Report.pdf
___

### Q: Please list any relevant protocol resources.
- https://specs.optimism.io
- https://docs.optimism.io 

___

### Q: Additional audit information.
A detailed contest handbook can be found here: https://oplabs.notion.site/Public-OP-Stack-Fault-Proofs-Sherlock-Competition-Handbook-e4cfdf210a5c45c79230af19653163cc

Please note that this Q&A takes precedence over the contest handbook. If discrepancies exist, information in this Q&A supersedes the runbook. Feedback about any discrepancies is much appreciated.
___



# Audit scope


[optimism @ 5137f3b74c6ebcac4f0f5a118b0f4909df03aec6](https://github.com/ethereum-optimism/optimism/tree/5137f3b74c6ebcac4f0f5a118b0f4909df03aec6)
- [optimism/packages/contracts-bedrock/src/L1/OptimismPortal2.sol](optimism/packages/contracts-bedrock/src/L1/OptimismPortal2.sol)
- [optimism/packages/contracts-bedrock/src/dispute/DisputeGameFactory.sol](optimism/packages/contracts-bedrock/src/dispute/DisputeGameFactory.sol)
- [optimism/packages/contracts-bedrock/src/dispute/FaultDisputeGame.sol](optimism/packages/contracts-bedrock/src/dispute/FaultDisputeGame.sol)
- [optimism/packages/contracts-bedrock/src/dispute/weth/DelayedWETH.sol](optimism/packages/contracts-bedrock/src/dispute/weth/DelayedWETH.sol)
- [optimism/packages/contracts-bedrock/src/dispute/weth/WETH98.sol](optimism/packages/contracts-bedrock/src/dispute/weth/WETH98.sol)


