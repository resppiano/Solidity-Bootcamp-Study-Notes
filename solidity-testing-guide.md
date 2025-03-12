# Complete Guide to Testing Solidity Smart Contracts

This comprehensive guide covers everything you need to know about testing Solidity smart contracts using Hardhat with Mocha, Chai, and VS Code for debugging.

## Table of Contents

- [Setting Up Your Environment](#setting-up-your-environment)
- [Writing Effective Tests](#writing-effective-tests)
- [Advanced Testing Techniques](#advanced-testing-techniques)
- [Debugging in VS Code](#debugging-in-vs-code)
- [Strategic Breakpoint Placement](#strategic-breakpoint-placement)
- [Common Testing Patterns](#common-testing-patterns)
- [Gas Optimization Testing](#gas-optimization-testing)
- [Example: Ballot Contract Test Suite](#example-ballot-contract-test-suite)

## Setting Up Your Environment

### Project Initialization

```bash
# Create a new directory for your project
mkdir smart-contract-project
cd smart-contract-project

# Initialize a new npm project
npm init -y

# Install Hardhat and dependencies
npm install --save-dev hardhat @nomiclabs/hardhat-waffle ethereum-waffle chai @nomiclabs/hardhat-ethers ethers

# Optional but recommended tools
npm install --save-dev hardhat-gas-reporter solidity-coverage
```

### Project Structure

```
project/
├── .vscode/
│   ├── launch.json       # VS Code debugging configuration
│   └── tasks.json        # VS Code tasks
├── contracts/
│   └── Ballot.sol        # Your Solidity contracts
├── test/
│   └── Ballot.test.js    # Test files
├── utilities/
│   └── console.js        # Helper functions
├── hardhat.config.js     # Hardhat configuration
└── package.json          # Node.js dependencies
```

### Hardhat Configuration

```javascript
// hardhat.config.js
require("@nomiclabs/hardhat-waffle");
require("@nomiclabs/hardhat-ethers");
require("hardhat-gas-reporter");
require("solidity-coverage");

module.exports = {
  solidity: {
    version: "0.8.4",
    settings: {
      optimizer: {
        enabled: true,
        runs: 200
      }
    }
  },
  networks: {
    hardhat: {
      // For running tests
    },
    localhost: {
      url: "http://127.0.0.1:8545"
    }
  },
  paths: {
    sources: "./contracts",
    tests: "./test",
    cache: "./cache",
    artifacts: "./artifacts"
  },
  mocha: {
    timeout: 40000
  },
  gasReporter: {
    enabled: (process.env.REPORT_GAS) ? true : false,
    currency: 'USD',
    coinmarketcap: process.env.COINMARKETCAP_API_KEY,
    outputFile: 'gas-report.txt',
    noColors: true,
    showMethodSig: true,
    excludeContracts: []
  }
};
```

## Writing Effective Tests

### Test Structure

Mocha provides a structured way to organize your tests:

```javascript
describe("Contract Name", function() {
  // Test group
  
  beforeEach(async function() {
    // Setup code that runs before each test
  });
  
  afterEach(async function() {
    // Cleanup code after each test (if needed)
  });
  
  describe("Function Name", function() {
    // Tests for a specific contract function
    
    it("should do something specific", async function() {
      // Individual test case
    });
  });
});
```

### Common Test Patterns

```javascript
// Testing a successful function call
it("Should allow voting for a valid proposal", async function() {
  // Setup
  await contract.giveRightToVote(voter1.address);
  
  // Action
  await contract.connect(voter1).vote(1);
  
  // Verification
  const voter = await contract.voters(voter1.address);
  expect(voter.voted).to.equal(true);
  expect(voter.vote).to.equal(1);
});

// Testing a failed function call
it("Should revert when unauthorized", async function() {
  await expect(
    contract.connect(voter1).giveRightToVote(voter2.address)
  ).to.be.revertedWith("Only chairperson can give right to vote.");
});

// Testing events
it("Should emit an event when successful", async function() {
  await expect(contract.vote(1))
    .to.emit(contract, "Voted")
    .withArgs(owner.address, 1, 1);
});
```

## Advanced Testing Techniques

### Testing with Different Accounts

```javascript
const [owner, voter1, voter2] = await ethers.getSigners();

// Call as owner (default)
await contract.function();

// Call as another account
await contract.connect(voter1).function();
```

### Testing State Changes

```javascript
// Get state before
const initialState = await contract.stateVariable();

// Perform action
await contract.function();

// Verify state change
const newState = await contract.stateVariable();
expect(newState).to.equal(initialState + 1);
```

### Testing Events

```javascript
// Test exact event arguments
await expect(contract.function())
  .to.emit(contract, "EventName")
  .withArgs(arg1, arg2);

// Or capture and inspect the event
const tx = await contract.function();
const receipt = await tx.wait();
const event = receipt.events.find(e => e.event === "EventName");
expect(event.args.param1).to.equal(expectedValue);
```

### Testing for Errors

```javascript
// Test for specific error message
await expect(
  contract.function()
).to.be.revertedWith("Error message");

// Test for any error
await expect(
  contract.function()
).to.be.reverted;
```

## Debugging in VS Code

### VS Code Configuration Files

#### launch.json

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Hardhat Tests",
      "skipFiles": ["<node_internals>/**"],
      "runtimeExecutable": "${workspaceFolder}/node_modules/.bin/hardhat",
      "args": ["test", "--network", "hardhat"],
      "cwd": "${workspaceFolder}",
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen"
    },
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Current Test File",
      "skipFiles": ["<node_internals>/**"],
      "runtimeExecutable": "${workspaceFolder}/node_modules/.bin/hardhat",
      "args": ["test", "${relativeFile}", "--network", "hardhat"],
      "cwd": "${workspaceFolder}",
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen"
    }
  ]
}
```

#### tasks.json

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Compile Contracts",
      "type": "shell",
      "command": "npx hardhat compile",
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "presentation": {
        "reveal": "always",
        "panel": "new"
      },
      "problemMatcher": []
    },
    {
      "label": "Run All Tests",
      "type": "shell",
      "command": "npx hardhat test",
      "group": {
        "kind": "test",
        "isDefault": true
      },
      "presentation": {
        "reveal": "always",
        "panel": "new"
      },
      "problemMatcher": []
    },
    {
      "label": "Run Current Test File",
      "type": "shell",
      "command": "npx hardhat test ${relativeFile}",
      "group": "test",
      "presentation": {
        "reveal": "always",
        "panel": "new"
      },
      "problemMatcher": []
    },
    {
      "label": "Run Test with Verbose Output",
      "type": "shell",
      "command": "npx hardhat test --verbose",
      "group": "test",
      "presentation": {
        "reveal": "always",
        "panel": "new"
      },
      "problemMatcher": []
    },
    {
      "label": "Start Local Node",
      "type": "shell",
      "command": "npx hardhat node",
      "isBackground": true,
      "presentation": {
        "reveal": "always",
        "panel": "new"
      },
      "problemMatcher": []
    }
  ]
}
```

### Debugging Techniques

1. **Setting Breakpoints**:
   - Click in the gutter next to line numbers in VS Code
   - Set conditional breakpoints with right-click on breakpoint

2. **Running Debug Sessions**:
   - Press F5 to start debugging
   - Choose "Debug Current Test File" configuration
   
3. **During Debugging**:
   - Step through code with F10 (Step Over), F11 (Step Into)
   - Use the VARIABLES panel to inspect variables
   - Add expressions to WATCH panel

4. **Console Debugging**:
   - Use the Debug Console to run commands while paused
   - Call contract functions with `await contract.function()`
   - Inspect state with `await contract.variable()`

### Using Hardhat Console.log in Contracts

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "hardhat/console.sol";

contract MyContract {
    function myFunction(uint x) public {
        console.log("Function called with:", x);
        
        if (x > 100) {
            console.log("X is greater than 100");
        }
        
        // Log addresses
        console.log("Sender:", msg.sender);
        
        // Log multiple values
        console.log("Values:", x, block.timestamp);
    }
}
```

## Strategic Breakpoint Placement

### Key Locations for Breakpoints

1. **Before Contract Interactions**:
   ```javascript
   // Set breakpoint here
   await contract.function();
   ```

2. **After State-Changing Operations**:
   ```javascript
   await contract.function();
   // Set breakpoint here
   const state = await contract.variable();
   ```

3. **Before Assertions**:
   ```javascript
   // Set breakpoint here
   expect(value).to.equal(expected);
   ```

4. **Inside Test Setup**:
   ```javascript
   beforeEach(async function() {
       // Set breakpoint here
       contract = await Contract.deploy();
   });
   ```

5. **For Transaction Receipts**:
   ```javascript
   const tx = await contract.function();
   // Set breakpoint here
   const receipt = await tx.wait();
   ```

### Visual Indicators for Good Breakpoint Locations

- **Function Calls**: Any `contract.function()` calls
- **State Queries**: Lines with `contract.variable()`
- **Event Handling**: Code that processes `receipt.events`
- **Conditional Logic**: `if/else` statements in tests
- **Loops**: Iteration over arrays or mappings

### Enhancing Debug Output

Add helper functions for better debugging:

```javascript
// utilities/console.js
function logVoterState(contract, address, label) {
  return async () => {
    const voter = await contract.voters(address);
    console.log(`${label}:`, {
      weight: voter.weight.toString(),
      voted: voter.voted,
      delegate: voter.delegate,
      vote: voter.vote.toString()
    });
  };
}

function logProposalState(contract, index, label) {
  return async () => {
    const proposal = await contract.proposals(index);
    console.log(`${label}:`, {
      name: ethers.utils.parseBytes32String(proposal.name),
      voteCount: proposal.voteCount.toString()
    });
  };
}

module.exports = {
  logVoterState,
  logProposalState
};
```

## Common Testing Patterns

### Testing With Time Manipulation

```javascript
// Advance time by 1 hour
await ethers.provider.send("evm_increaseTime", [3600]);
await ethers.provider.send("evm_mine");

// Set specific timestamp
await ethers.provider.send("evm_setNextBlockTimestamp", [1625097600]);
await ethers.provider.send("evm_mine");
```

### Testing with ETH Value

```javascript
// Send ETH with transaction
await contract.function({ value: ethers.utils.parseEther("1.0") });

// Check contract balance
const balance = await ethers.provider.getBalance(contract.address);
expect(balance).to.equal(ethers.utils.parseEther("1.0"));
```

### Testing Multi-Contract Interactions

```javascript
const Contract1 = await ethers.getContractFactory("Contract1");
const Contract2 = await ethers.getContractFactory("Contract2");

const contract1 = await Contract1.deploy();
const contract2 = await Contract2.deploy(contract1.address);

// Test interaction
await contract2.interactWithContract1();
const result = await contract1.value();
expect(result).to.equal(expectedValue);
```

## Gas Optimization Testing

### Using Gas Reporter

Enable gas reporting in hardhat.config.js:

```javascript
gasReporter: {
  enabled: true,
  currency: 'USD',
  gasPrice: 100,
  coinmarketcap: process.env.COINMARKETCAP_API_KEY
}
```

Then run tests:

```bash
REPORT_GAS=true npx hardhat test
```

### Measuring Gas Usage in Tests

```javascript
// Get gas used for a function call
const tx = await contract.function();
const receipt = await tx.wait();
console.log("Gas used:", receipt.gasUsed.toString());

// Compare gas between implementations
const tx1 = await contract1.function();
const receipt1 = await tx1.wait();

const tx2 = await contract2.function();
const receipt2 = await tx2.wait();

console.log("Implementation 1 gas:", receipt1.gasUsed.toString());
console.log("Implementation 2 gas:", receipt2.gasUsed.toString());
```

## Example: Ballot Contract Test Suite

Here's a comprehensive test suite for a voting contract:

```javascript
const { expect } = require("chai");
const { ethers } = require("hardhat");

describe("Ballot Contract", function () {
  let Ballot;
  let ballotContract;
  let proposalNames;
  let chairperson;
  let voter1;
  let voter2;
  let voter3;
  
  // Enable this to see console.log output
  let DEBUG = true;
  
  function log(...args) {
    if (DEBUG) console.log(...args);
  }
  
  beforeEach(async function () {
    // Convert proposal names to bytes32 format
    proposalNames = ["Proposal1", "Proposal2", "Proposal3"].map(name => 
      ethers.utils.formatBytes32String(name)
    );
    
    // Get the ContractFactory and Signers
    Ballot = await ethers.getContractFactory("Ballot");
    [chairperson, voter1, voter2, voter3] = await ethers.getSigners();
    
    // Deploy a new Ballot contract before each test
    ballotContract = await Ballot.deploy(proposalNames);
    await ballotContract.deployed();
  });
  
  describe("Deployment", function () {
    it("Should set the right chairperson", async function () {
      expect(await ballotContract.chairperson()).to.equal(chairperson.address);
    });
    
    it("Should set up proposals correctly", async function () {
      const proposal0 = await ballotContract.proposals(0);
      const proposal1 = await ballotContract.proposals(1);
      const proposal2 = await ballotContract.proposals(2);
      
      expect(proposal0.name).to.equal(proposalNames[0]);
      expect(proposal0.voteCount).to.equal(0);
      expect(proposal1.name).to.equal(proposalNames[1]);
      expect(proposal1.voteCount).to.equal(0);
      expect(proposal2.name).to.equal(proposalNames[2]);
      expect(proposal2.voteCount).to.equal(0);
    });
    
    it("Should give chairperson voting rights", async function () {
      const chairpersonVoter = await ballotContract.voters(chairperson.address);
      expect(chairpersonVoter.weight).to.equal(1);
      expect(chairpersonVoter.voted).to.equal(false);
    });
  });
  
  describe("Giving voting rights", function () {
    it("Should allow chairperson to give voting rights", async function () {
      await ballotContract.giveRightToVote(voter1.address);
      const voter = await ballotContract.voters(voter1.address);
      expect(voter.weight).to.equal(1);
    });
    
    it("Should not allow non-chairperson to give voting rights", async function () {
      await expect(
        ballotContract.connect(voter1).giveRightToVote(voter2.address)
      ).to.be.revertedWith("Only chairperson can give right to vote.");
    });
    
    it("Should not allow giving voting rights to voters who already voted", async function () {
      await ballotContract.giveRightToVote(voter1.address);
      await ballotContract.connect(voter1).vote(0);
      
      await expect(
        ballotContract.giveRightToVote(voter1.address)
      ).to.be.revertedWith("The voter already voted.");
    });
    
    it("Should not allow giving voting rights to voters who already have them", async function () {
      await ballotContract.giveRightToVote(voter1.address);
      
      // This will fail because voters[voter1].weight is already 1
      await expect(
        ballotContract.giveRightToVote(voter1.address)
      ).to.be.reverted;
    });
  });
  
  describe("Voting", function () {
    beforeEach(async function () {
      // Give voting rights to voter1
      await ballotContract.giveRightToVote(voter1.address);
    });
    
    it("Should allow voting for a valid proposal", async function () {
      const tx = await ballotContract.connect(voter1).vote(1); // Vote for proposal 1
      const receipt = await tx.wait();
      
      // Debug information
      log("Transaction hash:", tx.hash);
      log("Gas used:", receipt.gasUsed.toString());
      
      const voter = await ballotContract.voters(voter1.address);
      log("Voter state after voting:", {
        weight: voter.weight.toString(),
        voted: voter.voted,
        vote: voter.vote.toString()
      });
      
      expect(voter.voted).to.equal(true);
      expect(voter.vote).to.equal(1);
      
      const proposal = await ballotContract.proposals(1);
      log("Proposal vote count:", proposal.voteCount.toString());
      expect(proposal.voteCount).to.equal(1);
    });
    
    it("Should not allow voting without rights", async function () {
      await expect(
        ballotContract.connect(voter2).vote(1)
      ).to.be.revertedWith("Has no right to vote");
    });
    
    it("Should not allow voting twice", async function () {
      await ballotContract.connect(voter1).vote(0);
      
      await expect(
        ballotContract.connect(voter1).vote(1)
      ).to.be.revertedWith("Already voted.");
    });
    
    it("Should automatically revert when voting for a non-existent proposal", async function () {
      await expect(
        ballotContract.connect(voter1).vote(99) // Non-existent proposal
      ).to.be.reverted; // Will revert without a specific message
    });
  });
  
  describe("Delegation", function () {
    beforeEach(async function () {
      // Give voting rights to voters
      await ballotContract.giveRightToVote(voter1.address);
      await ballotContract.giveRightToVote(voter2.address);
      await ballotContract.giveRightToVote(voter3.address);
    });
    
    it("Should allow delegation to another voter", async function () {
      await ballotContract.connect(voter1).delegate(voter2.address);
      
      const voter1Data = await ballotContract.voters(voter1.address);
      const voter2Data = await ballotContract.voters(voter2.address);
      
      expect(voter1Data.voted).to.equal(true);
      expect(voter1Data.delegate).to.equal(voter2.address);
      expect(voter2Data.weight).to.equal(2); // original weight + delegated weight
    });
    
    it("Should not allow delegation without voting rights", async function () {
      // voter4 doesn't have voting rights
      const [,,,,voter4] = await ethers.getSigners();
      
      await expect(
        ballotContract.connect(voter4).delegate(voter1.address)
      ).to.be.revertedWith("You have no right to vote");
    });
    
    it("Should not allow delegation after voting", async function () {
      await ballotContract.connect(voter1).vote(0);
      
      await expect(
        ballotContract.connect(voter1).delegate(voter2.address)
      ).to.be.revertedWith("You already voted.");
    });
    
    it("Should not allow self-delegation", async function () {
      await expect(
        ballotContract.connect(voter1).delegate(voter1.address)
      ).to.be.revertedWith("Self-delegation is disallowed.");
    });
    
    it("Should handle delegation to someone who already voted", async function () {
      // Voter2 votes for proposal 0
      await ballotContract.connect(voter2).vote(0);
      
      // Voter1 delegates to Voter2
      await ballotContract.connect(voter1).delegate(voter2.address);
      
      // Check that proposal 0's vote count increased by voter1's weight
      const proposal = await ballotContract.proposals(0);
      expect(proposal.voteCount).to.equal(2); // voter2 (1) + voter1 (1)
    });
    
    it("Should detect delegation loops", async function () {
      // Create a potential loop: voter3 -> voter1 -> voter3
      await ballotContract.connect(voter3).delegate(voter1.address);
      
      // This should detect the loop
      await expect(
        ballotContract.connect(voter1).delegate(voter3.address)
      ).to.be.revertedWith("Found loop in delegation.");
    });
    
    it("Should handle multi-level delegation", async function () {
      // Create delegation chain: voter1 -> voter2 -> voter3
      await ballotContract.connect(voter2).delegate(voter3.address);
      await ballotContract.connect(voter1).delegate(voter2.address);
      
      // Check that voter3 received weight from both voter1 and voter2
      const voter3Data = await ballotContract.voters(voter3.address);
      expect(voter3Data.weight).to.equal(3); // Original (1) + voter2 (1) + voter1 (1)
    });
  });
  
  describe("Winning Proposal", function () {
    beforeEach(async function () {
      // Give voting rights
      await ballotContract.giveRightToVote(voter1.address);
      await ballotContract.giveRightToVote(voter2.address);
      await ballotContract.giveRightToVote(voter3.address);
    });
    
    it("Should correctly determine the winning proposal", async function () {
      // Chairperson votes for proposal 0
      await ballotContract.connect(chairperson).vote(0);
      
      // Voter1 votes for proposal 1
      await ballotContract.connect(voter1).vote(1);
      
      // Voter2 and Voter3 vote for proposal 2
      await ballotContract.connect(voter2).vote(2);
      await ballotContract.connect(voter3).vote(2);
      
      // Proposal 2 should win with 2 votes
      expect(await ballotContract.winningProposal()).to.equal(2);
      expect(await ballotContract.winnerName()).to.equal(proposalNames[2]);
    });
    
    it("Should handle a tie by selecting the first proposal with max votes", async function () {
      // Chairperson and Voter1 vote for proposal 0
      await ballotContract.connect(chairperson).vote(0);
      await ballotContract.connect(voter1).vote(0);
      
      // Voter2 and Voter3 vote for proposal 1
      await ballotContract.connect(voter2).vote(1);
      await ballotContract.connect(voter3).vote(1);
      
      // It's a tie, but the first proposal with max votes should win (proposal 0)
      expect(await ballotContract.winningProposal()).to.equal(0);
      expect(await ballotContract.winnerName()).to.equal(proposalNames[0]);
    });
    
    it("Should work with weighted votes from delegation", async function () {
      // Voter2 and Voter3 delegate to Voter1
      await ballotContract.connect(voter2).delegate(voter1.address);
      await ballotContract.connect(voter3).delegate(voter1.address);
      
      // Chairperson votes for proposal 0
      await ballotContract.connect(chairperson).vote(0);
      
      // Voter1 votes for proposal 1 with weight 3 (own + delegate from voter2 and voter3)
      await ballotContract.connect(voter1).vote(1);
      
      // Proposal 1 should win because it has weight 3 vs proposal 0's weight 1
      expect(await ballotContract.winningProposal()).to.equal(1);
      expect(await ballotContract.winnerName()).to.equal(proposalNames[1]);
    });
  });
});
```

## Step-by-Step Debugging Process

Here's a practical example of debugging the delegation logic:

### Problem Statement
Let's say we're debugging why delegation weight isn't being properly transferred.

### Debugging Flow

1. **Set Initial Breakpoint**: Place a breakpoint at the beginning of the delegation test
   ```javascript
   it("Should handle delegation to someone who already voted", async function () {
     // BREAKPOINT HERE
   ```

2. **Verify Initial State**: When the debugger stops, add these watch expressions:
   - `ballotContract.address`
   - `voter1.address`
   - `voter2.address`

3. **Step Through Rights Assignment**: Press F10 to step over each line.
   
   Check voter state in debug console:
   ```javascript
   await ballotContract.voters(voter1.address)
   await ballotContract.voters(voter2.address)
   ```

4. **Set Breakpoint Before Delegation**: Continue execution until just before the delegation call.

5. **Verify Post-Delegation State**: Set breakpoint after delegation and check:
   ```javascript
   await ballotContract.voters(voter1.address) // Should show voted=true, delegate=voter2.address
   await ballotContract.voters(voter2.address) // Should show increased weight
   ```

6. **Check Proposal State**: If voter2 already voted, verify the proposal's vote count.

7. **Log Call Trace**: Enable verbose logging if needed:
   ```bash
   npx hardhat test --verbose
   ```

## Conclusion

Effective testing and debugging of Solidity contracts requires:

1. **Well-Structured Tests**: Organized by functionality and covering both happy paths and edge cases
2. **Strategic Breakpoints**: Placed at key state-changing operations and verification points
3. **Contract Instrumentation**: Using Hardhat's console.log for internal contract visibility
4. **VS Code Debugging Tools**: Utilizing watch expressions, variable inspection, and step-through debugging

Following these practices will lead to more reliable and secure smart contracts by catching issues early in the development cycle.
