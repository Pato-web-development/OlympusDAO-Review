**Olympus DAO Review**

The Olympus DAO was built on the Ethereum blockchain. Olympus is a decentralized reserve currency protocol that is based on the OHM token. It was launched in May 2021 with a goal to build community-owned DeFi infrastructure and bring more transparency and stability in the financial space.

The Olympus DAO Treasury.sol handles the logic for the finance or keeping record of assets on the Olympus DAO protocol. The treasury system is an accounting and financial management mechanism that manages the cash and other financial assets of the organization. It manages the funds of the Olympus DAO. The Treasury.sol contract contains functions that allow users to deposit funds into the treasury, withdraw funds, and transfer funds to other contracts or wallets.

The Olympus treasury contract uses safemath.sol library for bascic calculations. The library provides basic addition, subtraction, multiplication, division, and modulo mathematical calculation functions that are being used in contract.

Another library used in the contract is the SafeERC20.sol library. This is a wrapper library around ERC-20 calls. It helps for secure interaction with your token. SafeERC20 is a wrapper around the interface that eliminates the need to handle boolean return values. It’s a helper to make safe the interaction with someone else’s ERC20 token in your contracts.

**The following are interfaces that the Olympus DAO treasury uses:**
The iOwnable. sol contract interface included in the Olympus treasury contract provides the most basic single account ownership to a contract. Only one account will become the owner of the contract and can perform administration-related tasks. The current owner of the contract can either transfer or renounce the ownership of the contract.

The IERC20. sol interface is an interface that defines the functions and events that are required for the ERC20 token standard.
ierc20metadata is Interface for the optional metadata functions from the ERC20 standard.

It also makes use of the IOHM.sol which contains the mint and burning functions of the OHM token.

The IsOHM.sol interface contains functions that show the current supply of the OHM token and debt balances of each user.

The Olympus Treasury.sol has an IBondingCalculator interface, a mathematical tool that allows investors to calculate Bond Yield at a specific price or Bond Price at a desired level of yield.

An ITreasury.sol interface that uses external functions (mint, withdraw, manage) that can be called outside the ontract and view functions to show the token Value, excess reserve and base supply.

The DAO also has the OlympusAccessControlled.sol file that contains modifiers that control access to specific functions in the DAO. For example, if you are a governor in the DAO, you will have access to an only governor-controlled function. Olympus DAO controls access by using the following modifiers for members according to their ranks in the DAO, "OnlyGovernor, onlyGuardian(), onlyPolicy(), onlyVault()".

**From here, we will deep dive into other features of the Treasury.sol file of the Olympus DAO.**

Events are emitted for some functions when they execute i.e, for Deposit, an event is emitted in the event log which includes the address of the token, amount and value of the token. Other functions that emit events are Withdrawal, CreateDebt, Minted etc.
**The contract also contains an enum of STATUS(constants) which are:** 
        RESERVEDEPOSITOR,
        RESERVESPENDER,
        RESERVETOKEN,
        RESERVEMANAGER,
        LIQUIDITYDEPOSITOR,
        LIQUIDITYTOKEN,
        LIQUIDITYMANAGER,
        RESERVEDEBTOR,
        REWARDMANAGER,
        SOHM,
        OHMDEBTOR
        
    **Structs in the contract also collect the following data:**
        STATUS managing;
        address toPermit;
        address calculator;
        uint256 timelockEnd;
        bool nullify;
        bool executed;

The contract contains state variables such as OHM and sOHM etc.

The contract mapped status to address with the name "registry".
It also mapped STATUS to a mapping of an address to a boolean with the name "permissions", thereby creating a 2D mapping.

A constructor in the Treasury.sol file takes in an address _ohm, a uint256 _timelock, and the address of the authority _authority.

From here we dive into the functions of the Treasury.sol file of the Olympus DAO.

**The Deposit Function:**
This function enables the Treasury.sol contract to receive deposits and in particualr and restricted to ERC20 tokens using the IERC20 interface. The deposit function contains a require function that prevents users from depositing tokens that are not initially in the liquidity pool. An event is emitted whenever a deposit from a user into the contract is successful, so it emits the token deposited, the amount of that token and the value of the token.

**The Withdraw Function:**
It takes in the amount and contract of the token to be withdrawn.
 ***require(permissions[STATUS.RESERVETOKEN][_token], notAccepted); // Only reserves can be used for ***redemptions require(permissions[STATUS.RESERVESPENDER][msg.sender], notApproved);***
The first require statement in the withdraw function checks if the token a user is trying to withdraw is in the contract, and if no, goes on to the next require statement.
Another require function checks to see if that user has invested in the contract, and if yes, executes the withdraw function.
 ***uint256 value = tokenValue(_token, _amount);****
  *** OHM.burnFrom(msg.sender, value); ***
As the user is withdrawing, a percentage of the withdrawal is burned using a burning mechanism or logic.
That updates the Reserve balance of the DAO.
Then the withdraw function is executed using the IERC20 interface and an event is emitted which shows the token, amount and value of the token being withdrawn.

**The Manage Function:**
This is a withdrawal function that can only be executed by the Manager of the DAO Treasury.
It takes in the contract of the token to be withdrawn and the amount.
It checks if the token to be withdrawn is in the contract's liquidity using a require statement.
It also requires that the manager must be the person executing the function, else, the transaction reverts.
An if statement has also been included to check if the token is available in the Liquidity or Reserves of the DAO. If yes, the withdrawal function is triggered, sening the amount to the manager's address and an event is emitted logging the token and the amount.

**The Mint Function:**
It's a RewardManager only function which can be initialized by only the Rewards Manager.
The mint function mints new OHM tokens as rewards to a recipient and therefore subsequently increases the total supply of the token. It takes in address of the recipient and the amount to be minted.
It uses the require statement and checks to see if it's the Reward manager that is tring to mint.
It aslso requires that the amount input should be less than the amount in the excessReserves.
Then it mints to the recipient's address the amount specified.
An event is emitted which logs the Reward Manager address, the recipient and the amount.
    function mint(address _recipient, uint256 _amount) external override {
        require(permissions[STATUS.REWARDMANAGER][msg.sender], notApproved);
        require(_amount <= excessReserves(), insufficientReserves);
        OHM.mint(_recipient, _amount);
        emit Minted(msg.sender, _recipient, _amount);
    }

**The incurDebt Function:**
The function takes in a contract address and the amount.
This function allows an approved user of the DAO to borrow from the Reserves.
 It contains an if statement that checks to see if the user address input is the same as the OHM contract address and a require function to check if the user address is approved. Another if and else statement checks to see if the input address is a contract address that exists in the reserves.
 A require statement is also there to check if the debt balance limited has not been exceeded by the user.
  ***require(sOHM.debtBalances(msg.sender) <= debtLimit[msg.sender], "Treasury: exceeds limit");***
  If the above requirements are met, the contract mints the amount of tokens to the user's wallet address if OHM, and transfers the amount equivalent if another token. It emits an event logging the borrower, the token borrowed, amount and value of the borrowed token.

  **The repayDebtWithReserve Function:**
  This function allows a debtor to repay his debt. It takes in the contract address and the amount.
  It has two require functions. One checks if the user borrowed money from the reserve and the other checks if the token the user wants to repay is in the reserve.
If the rquirements are met, the user can deposit the amount using the IERC20 interface and the user's debt balance will be updated. An event is emitted logging the address of the debtor, address of the contract, amount and value of the token paid.

**The repayDebtWithOHM Function:**
A require function with an "or" operator helps to check if the user borrowed OHM or other tokens from the reserve.
A burn function initializes when the user deposits OHM and the user's debt balance is updated.
In other words, as the user deposits OHM, the whole debt balance including the one for other tokens is updated. An event is emitted logging the address of the debtor, contract address of OHM, the amount and the value.

**Managerial Functions:**
The auditReserves Function:
This function can only be called by the governor of the treasury. The function helps to register new tokens in the Reserves. Tokens have to be ERC20 because they are using the IERC20 interface.
An event is emitted which logs the tokens in the Reserves. 








