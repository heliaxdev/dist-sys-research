### Heterogeneous Paxos x Tendermint

Basic goal:

Integrate the heterogeneous learner graph model described in the [heterogeneous Paxos paper](https://arxiv.org/abs/2011.08253) into a new version of the Tendermint [algorithm](https://arxiv.org/abs/1807.04938) and [codebase](https://github.com/tendermint/tendermint).

Theoretical questions:

- How do we figure out when heterogeneous consensus is possible, and how do we decide which to do?

    We need an algorithm for taking a learner graph and condensing it to find valid learner subgraphs. The naive way of doing this is simply to generate every possible subgraph and check if each one is valid.
  Then we can prove to an on-chain state machine w/attestations from validators that specify their trust assumptions that we can construct a valid learner subgraph.

    However, we can construct multiple valid subgraphs. Consider a case where we have three validator sets - A, B, and C - where A overlaps enough with B, B overlaps enough with C, but C does not overlap enough with A. Thus these choices are mutually exclusive - and everyone participating needs to agree on which one they are using.
  One option: subgraph operating over needs to be referenced in proposal (block proposal). However, competing proposals could have overlapping non-identical trust subgraphs, so we'd need a prior consensus round to just agree on the subgraph, which is rather inefficient.

    Perhaps a better idea: schedule in advance somewhat. Measure aggregate demand, schedule joint consensus rounds when necessary, or even something simple like every tenth block. Transactions could include constraints describing under which conditions they could be ratified.

Practical questions:

- Can we optimise heterogeneous Paxos for some Tendermint advantages?
   - Message/communication complexity: improve by using leaders.
   - We need a timeout structure
       - Can we avoid the required increasing timeouts which Tendermint has?
       - Maybe three-round (but can be pipelined) like HotStuff.
   - Think about proposer selection mechanism
       - We want to select the proposer from intersection of proposers for all involved chains.
