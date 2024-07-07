# Connecting Contract with Front-end and cryptocurreny wallet

This Solidity contract defines an error handling system with credit and debit functions, allowing for the increase and decrease of balances. The contract also includes checks to ensure that only the owner can perform these operations and maintains the total balance. Additionally, the contract provides functions to view the balance, gas price, gas left, and current block number, accessible only by the owner. The contract's functionality is designed to ensure secure and controlled management of funds. The accompanying frontend code integrates with this contract, allowing users to interact with it using their MetaMask wallet, providing a user-friendly interface to perform deposit, withdrawal, and balance inquiry operations.

## Description

This program is a comprehensive contract written in Solidity, a programming language used for developing smart contracts on the Ethereum blockchain. This contract implements an advanced error handling system for managing balances, allowing the owner to credit and debit amounts from user accounts. Additionally, it maintains a record of each user's balance and ensures the integrity of the total balance across all accounts.

The contract is designed to provide a secure and controlled environment for managing funds, with built-in checks and balances to prevent unauthorized access or tampering. The owner of the contract has exclusive control over the credit and debit functions, ensuring that only authorized transactions are processed.

One of the key aspects of this contract is its ability to provide real-time information about the current state of the blockchain. The contract includes functions to retrieve the current gas price, the amount of gas left, and the current block number, providing the owner with a comprehensive view of the blockchain's activity.

The accompanying frontend code provides a user-friendly interface for interacting with the contract, allowing users to perform deposit, withdrawal, and balance inquiry operations. This interface is integrated with MetaMask, a popular Ethereum wallet, which enables users to securely connect to the Ethereum network and interact with the contract using their own Ethereum accounts. When a user initiates a transaction through the frontend, MetaMask prompts them to confirm the transaction details and authorize the transaction with their private key, ensuring a secure and trustless interaction with the contract.

### Executing program

To run this program, you can use Gitpod or can run on your local server. To get started, just open the starter template provided at github on https://github.com/MetacrafterChris/SCM-Starter it is a public repository .

Once you are on the Gitpod website, open the terminal and clone the repository or directly open the repository on gitpod using your own github account.
After that just replace the code of "Assessment.sol" with the code:

```javascript
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.9;

//import "hardhat/console.sol";

contract Assessment {
    address payable public owner;
    uint256 public balance;

    event Deposit(uint256 amount);
    event Withdraw(uint256 amount);

    constructor(uint initBalance) payable {
        owner = payable(msg.sender);
        balance = initBalance;
    }

    function getBalance() public view returns(uint256){
        return balance;
    }

    function deposit(uint256 _amount) public payable {
        uint _previousBalance = balance;

        // make sure this is the owner
        require(msg.sender == owner, "You are not the owner of this account");

        // perform transaction
        balance += _amount;

        // assert transaction completed successfully
        assert(balance == _previousBalance + _amount);

        // emit the event
        emit Deposit(_amount);
    }

    // custom error
    error InsufficientBalance(uint256 balance, uint256 withdrawAmount);

    function withdraw(uint256 _withdrawAmount) public {
        require(msg.sender == owner, "You are not the owner of this account");
        uint _previousBalance = balance;
        if (balance < _withdrawAmount) {
            revert InsufficientBalance({
                balance: balance,
                withdrawAmount: _withdrawAmount
            });
        }

        // withdraw the given amount
        balance -= _withdrawAmount;

        // assert the balance is correct
        assert(balance == (_previousBalance - _withdrawAmount));

        // emit the event
        emit Withdraw(_withdrawAmount);
    }

    function getGasPrice() public view returns(uint) {
		return tx.gasprice;
	}

    function getGasLeft() public view returns(uint) {
		return gasleft();	
	}

    function getCurrentBlock() public view returns(uint) {
		return block.number;
	}


}

```
And then open the file "index.js" and replace its code with:

```javascript
import {useState, useEffect} from "react";
import {ethers} from "ethers";
import atm_abi from "../artifacts/contracts/Assessment.sol/Assessment.json";

export default function HomePage() {
  const [ethWallet, setEthWallet] = useState(undefined);
  const [account, setAccount] = useState(undefined);
  const [atm, setATM] = useState(undefined);
  const [balance, setBalance] = useState(undefined);
  const [gasPrice, setGasPrice] = useState(undefined);
  const [gasLeft, setGasLeft] = useState(undefined);
  const [currentBlock, setcurrentBlock] = useState(undefined);

  const contractAddress = "0x5FbDB2315678afecb367f032d93F642f64180aa3";
  const atmABI = atm_abi.abi;

  const getWallet = async() => {
    if (window.ethereum) {
      setEthWallet(window.ethereum);
    }

    if (ethWallet) {
      const account = await ethWallet.request({method: "eth_accounts"});
      handleAccount(account);
    }
  }

  const handleAccount = (account) => {
    if (account) {
      console.log ("Account connected: ", account);
      setAccount(account);
    }
    else {
      console.log("No account found");
    }
  }

  const connectAccount = async() => {
    if (!ethWallet) {
      alert('MetaMask wallet is required to connect');
      return;
    }
  
    const accounts = await ethWallet.request({ method: 'eth_requestAccounts' });
    handleAccount(accounts);
    
    // once wallet is set we can get a reference to our deployed contract
    getATMContract();
  };

  const getATMContract = () => {
    const provider = new ethers.providers.Web3Provider(ethWallet);
    const signer = provider.getSigner();
    const atmContract = new ethers.Contract(contractAddress, atmABI, signer);
 
    setATM(atmContract);
  }

  const getBalance = async() => {
    if (atm) {
      setBalance((await atm.getBalance()).toNumber());
    }
  }

  const deposit = async() => {
    if (atm) {
      let tx = await atm.deposit(1);
      await tx.wait()
      getBalance();
    }
  }

  const withdraw = async() => {
    if (atm) {
      let tx = await atm.withdraw(1);
      await tx.wait()
      getBalance();
    }
  }

  const getGasPrice = async() =>{
    if(atm) {
      let tx= await atm.getGasPrice();
      setGasPrice(tx.toNumber());
    }
  }

  const getGasLeft = async() =>{
    if(atm) {
      let tx= await atm.getGasLeft();
      setGasLeft(tx.toNumber());
    }
  }

  const getCurrentBlock = async() =>{
    if(atm) {
      let tx= await atm.getCurrentBlock();
      setcurrentBlock(tx.toNumber());
    }
  }

  const initUser = () => {
    // Check to see if user has Metamask
    if (!ethWallet) {
      return <p>Please install Metamask in order to use this ATM.</p>
    }

    // Check to see if user is connected. If not, connect to their account
    if (!account) {
      return <button onClick={connectAccount}>Please connect your Metamask wallet</button>
    }

    if (balance == undefined) {
      getBalance();
    }

    return (
      <div>
        <p>Your Account: {account}</p>
        <p>Your Balance: {balance}</p>
        <button onClick={deposit}>Deposit 1 ETH</button>
        <button onClick={withdraw}>Withdraw 1 ETH</button>
        
        <p><button onClick={getGasPrice}>Check Gas Price for current transaction</button></p>
          {
            gasPrice !== undefined && <p> Gas Price: {gasPrice}</p>
          }

        <p><button onClick={getGasLeft}>Check Gas Left</button></p>
          {
            gasLeft !== undefined && <p> Gas Left: {gasLeft}</p>
          }

        <p><button onClick={getCurrentBlock}>Check for current block running</button></p>
          {
            currentBlock !== undefined && <p> Current Block:{currentBlock}</p>
          }

      </div>
    )
  }

  useEffect(() => {getWallet();}, []);

  return (
    <main className="container">
      <header><h1>Welcome to the Metacrafters ATM!</h1></header>
      {initUser()}
      <style jsx>{`
        .container {
          text-align: center
        }
      `}
      </style>
    </main>
  )
}

```
After doing above all steps, you will want to do the following to get the code running on your computer.

1. Inside the project directory, in the terminal type: npm i
2. Open two additional terminals in your VS code
3. In the second terminal type: npx hardhat node
4. In the third terminal, type: npx hardhat run --network localhost scripts/deploy.js
5. Back in the first terminal, type npm run dev to launch the front-end.

After this, the project will be running on your localhost. 
Typically at http://localhost:3000/

Once the contract is deployed, you can interact with it by calling various functions to manage balances. To view the balance of an account, click on the getBalance function and execute the function to see the balance of the specified address. To credit an account, click on the deposit function, pass the required parameter (uint _amount), and execute the function to increase the balance of the specified address. To debit an account, click on the withdraw function, pass the required parameter (uint _withdrawAmount), and execute the function to decrease the balance of the specified address, provided it has sufficient balance. You can also view the current gas price by clicking on the getGasPrice function, view the remaining gas by clicking on the getGasLeft function, and view the current block number by clicking on the getCurrentBlock function. The contract uses require to ensure only the owner can call deposit and withdraw functions, assert to enforce critical conditions, and revert to halt transactions if conditions are not met, ensuring robust error handling and access control.


## Authors

Saurabh Mishra  


## License

This project is licensed under the Unlicensed
