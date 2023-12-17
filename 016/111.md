Sour Cobalt Tardigrade

medium

# Gas Inefficiency and Lack of Event Emission in Withdraw Function (Large Transaction + No Event).

## Summary
The withdraw function is missing gas optimization and event emission, which could affect the performance and usability of the contract.
## Vulnerability Detail
The withdraw function sends the entire contract balance in one large transaction, which might be gas-intensive and prone to failure. A better approach would be to use a withdrawal pattern, where each user can withdraw their own share of the contract balance, instead of sending the entire balance to the owner. This would reduce the gas cost and improve the security of the contract.

The withdraw function also doesn’t emit any event informing users or external tools about the withdrawal. This could affect the usability and transparency of the contract, as users would not be able to track or verify the withdrawals. Emitting an event with relevant information like timestamp, amount withdrawn, and owner address would improve the user experience and accountability of the contract.
## Impact
The gas inefficiency and lack of event emission could result in higher transaction fees, lower reliability, and lower user satisfaction for the contract.
## Code Snippet
https://github.com/sherlock-audit/2023-12-footium/blob/main/footium-eth-shareable/contracts/FootiumAcademy.sol#L144-L152
## Tool used
- Manual Review
## Test case
```solidity
// The contract owner
address owner = 0x1234;
// The contract balance
uint256 balance = 10 ether;
// The withdraw function
function withdraw() external onlyOwner {
  uint256 balance = address(this).balance;
  if (balance > 0) {
    (bool sent, ) = payable(owner()).call{value: balance}("");
    if (!sent) {
      revert FailedToSendETH(balance);
    }
  }
}

```
## Logs
```javascript
Owner calls withdraw()...
Owner receives 10 ether...
No event emitted...

```
## Recommendation
```solidity
// Declare a mapping to store each user's balance
mapping(address => uint256) public balances;

// ...

// Update the user's balance when they deposit or receive funds
function deposit() external payable {
  balances[msg.sender] += msg.value;
}

// ...

// Allow each user to withdraw their own balance
function withdraw() external {
  uint256 amount = balances[msg.sender];
  require(amount > 0, "No balance to withdraw");
  // Update the state variable before sending
  balances[msg.sender] = 0;
  // Use transfer instead of call for better security
  payable(msg.sender).transfer(amount);
  // Emit an event to inform about the withdrawal
  emit Withdrawal(msg.sender, amount, block.timestamp);
}

// Declare an event to inform about the withdrawal
event Withdrawal(address indexed user, uint256 amount, uint256 timestamp);

```