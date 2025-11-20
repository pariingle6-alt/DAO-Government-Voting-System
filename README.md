// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract DAOGovernanceVotingSystem {
    
    struct Proposal {
        uint256 id;
        string description;
        uint256 voteYes;
        uint256 voteNo;
        uint256 deadline;
        bool executed;
    }

    uint256 public nextProposalId;
    mapping(uint256 => Proposal) public proposals;
    mapping(uint256 => mapping(address => bool)) public hasVoted;

    event ProposalCreated(uint256 proposalId, string description, uint256 deadline);
    event Voted(uint256 proposalId, address voter, bool voteYes);
    event ProposalExecuted(uint256 proposalId, bool approved);

    modifier proposalExists(uint256 _proposalId) {
        require(_proposalId < nextProposalId, "Proposal does not exist");
        _;
    }

    modifier beforeDeadline(uint256 _proposalId) {
        require(block.timestamp < proposals[_proposalId].deadline, "Voting period ended");
        _;
    }

    modifier onlyAfterDeadline(uint256 _proposalId) {
        require(block.timestamp >= proposals[_proposalId].deadline, "Voting still active");
        _;
    }

    /// @notice Create a new governance proposal
    function createProposal(string calldata _description, uint256 _durationInMinutes) external {
        require(_durationInMinutes > 0, "Duration must be positive");

        uint256 proposalId = nextProposalId++;
        proposals[proposalId] = Proposal({
            id: proposalId,
            description: _description,
            voteYes: 0,
            voteNo: 0,
            deadline: block.timestamp + (_durationInMinutes * 1 minutes),
            executed: false
        });

        emit ProposalCreated(proposalId, _description, proposals[proposalId].deadline);
    }

    /// @notice Cast a vote on a proposal
    function vote(uint256 _proposalId, bool _voteYes) 
        external 
        proposalExists(_proposalId) 
        beforeDeadline(_proposalId) 
    {
        require(!hasVoted[_proposalId][msg.sender], "Already voted");

        hasVoted[_proposalId][msg.sender] = true;

        if (_voteYes) {
            proposals[_proposalId].voteYes++;
        } else {
            proposals[_proposalId].voteNo++;
        }

        emit Voted(_proposalId, msg.sender, _voteYes);
    }

    /// @notice Execute proposal after deadline
    function executeProposal(uint256 _proposalId) 
        external 
        proposalExists(_proposalId) 
        onlyAfterDeadline(_proposalId) 
    {
        Proposal storage proposal = proposals[_proposalId];
        require(!proposal.executed, "Proposal already executed");

        proposal.executed = true;

        bool approved = proposal.voteYes > proposal.voteNo;

        emit ProposalExecuted(_proposalId, approved);
    }
}
0xA6a8E552d431f8b231b1fF0356bfF249C787402b
<img width="1920" height="1033" alt="image" src="https://github.com/user-attachments/assets/cc84a03e-18f6-4550-af28-8214bd060274" />
