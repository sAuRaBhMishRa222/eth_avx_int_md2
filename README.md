# Connecting Contract with Front-end and cryptocurreny wallet

This Solidity contract defines an error handling system with credit and debit functions, allowing for the increase and decrease of balances. The contract also includes checks to ensure that only the owner can perform these operations and maintains the total balance. Additionally, the contract provides functions to view the balance, gas price, gas left, and current block number, accessible only by the owner. The contract's functionality is designed to ensure secure and controlled management of funds. The accompanying frontend code integrates with this contract, allowing users to interact with it using their MetaMask wallet, providing a user-friendly interface to perform deposit, withdrawal, and balance inquiry operations.

## Description

This program is a comprehensive contract written in Solidity, a programming language used for developing smart contracts on the Ethereum blockchain. This contract implements an advanced error handling system for managing balances, allowing the owner to credit and debit amounts from user accounts. Additionally, it maintains a record of each user's balance and ensures the integrity of the total balance across all accounts.

The contract is designed to provide a secure and controlled environment for managing funds, with built-in checks and balances to prevent unauthorized access or tampering. The owner of the contract has exclusive control over the credit and debit functions, ensuring that only authorized transactions are processed.

One of the key aspects of this contract is its ability to provide real-time information about the current state of the blockchain. The contract includes functions to retrieve the current gas price, the amount of gas left, and the current block number, providing the owner with a comprehensive view of the blockchain's activity.

The accompanying frontend code provides a user-friendly interface for interacting with the contract, allowing users to perform deposit, withdrawal, and balance inquiry operations. This interface is integrated with MetaMask, a popular Ethereum wallet, which enables users to securely connect to the Ethereum network and interact with the contract using their own Ethereum accounts. When a user initiates a transaction through the frontend, MetaMask prompts them to confirm the transaction details and authorize the transaction with their private key, ensuring a secure and trustless interaction with the contract.

### Executing program

To run this program, you can use Gitpod or can run on your local server. To get started, just open the.

Once you are on the Remix website, create a new file by clicking on the "+" icon in the left-hand sidebar. Save the file with a .sol extension (e.g., myToken.sol). Copy and paste the following code into the file:

```javascript
// SPDX-License-Identifier: MIT
pragma solidity 0.8.26;

contract errorhandling{
    address public owner;
    string public owner_name="Saurabh Mishra";
    uint public total_balance=0;

    mapping (address=>uint) public balances;

    constructor() {
        owner=msg.sender;
    }

    function credit(address _address, uint _amount) public {
        require(msg.sender==owner,"Only Owner has access to it...!!!!");
        total_balance+= _amount;
        balances[_address]+= _amount;

        assert(total_balance >= balances[_address]);
    } 

    function debit(address _address, uint _amount) public {
        require(msg.sender==owner,"Only Owner has access to it...!!!!");
        if(balances[_address]<_amount){
            revert("Balance is Insufficient");
        }
        total_balance-= _amount;
        balances[_address]-= _amount;

        assert(total_balance >= 0);
    }

    function get_balance(address _address) public view returns(uint ){
        assert(msg.sender==owner);
        return balances[_address];
    } 
}

```

To compile the code, click on the "Solidity Compiler" tab in the left-hand sidebar. Make sure the "Compiler" option is set to "0.8.26" (or another compatible version), and then click on the "Compile eth_avx_md1.sol" button.

Once the code is compiled, you can deploy the contract by clicking on the "Deploy & Run Transactions" tab in the left-hand sidebar. Select the "errorhandling" contract from the dropdown menu, and then click on the "Deploy" button.

Once the contract is deployed, you can interact with it by calling various functions to manage balances. Click on the deployed `errorhandling` contract in the left-hand sidebar. To credit an account, click on the `credit` function, pass the required parameters (address `_address` and uint `_amount`), and execute the function to increase the balance of the specified address and update the total balance. To debit an account, click on the `debit` function, pass the required parameters (address `_address` and uint `_amount`), and execute the function to decrease the balance of the specified address, provided it has sufficient balance, and update the total balance. To view the balance of an account, click on the `get_balance` function, pass the required parameter (address `_address`), and execute the function to see the balance of the specified address. The contract uses `require` to ensure only the owner can call `credit` and `debit` functions, `assert` to enforce critical conditions, and `revert` to halt transactions if conditions are not met, ensuring robust error handling and access control.

## Authors

Saurabh Mishra  


## License

This project is licensed under the MIT License
