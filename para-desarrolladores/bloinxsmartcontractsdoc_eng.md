# BloinxSmartContractsDoc\_eng

**Bloinx Smart contracts**

**Terminology**

| Round               | Is the complete cycle of the saving circle that is divided in turns according to the number of participants.             |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Turn                | Is one period of the saving circle. I.E In a weekly round of 5 people, the suns last 1 week and the round lasts 5 weeks. |
| Administrator/admin | The creator of the round.                                                                                                |

**Rules**

1. Create a saving round
   1. The user that creates the saving round will have the administrator role, they must indicate the number of guests, the amount of the payment(per turn), the periodicity and invite others to participate in their round.
   2. Take into account that you will not be able to modify this data and that each guest can only register for one turn of the round.
2. Registration
   1. In the registration stage, users will be able to select a turn and must make a payment in advance as a security deposit that will be reimbursed according to payment fulfillment.
   2. The admin is responsible to verify that the registered addresses correspond to those of their guests. In case a guest is not in the correct round, they will be able to remove them.
   3. Important: once the round has started, no changes in the users is possible.
3. Payments and withdrawals
   1. When the round starts, all the guests will have the days you indicated in periodicity to make the first payment. The person on the first turn does not pay.
   2. Once the payment time has passed, the guest in the first turn will be able to collect the accumulated fund. This is repeated for the number of participants in the round.
   3. If one of your guests does not pay on time, the initial fee will be taken from them to cover their contribution for that turn.
   4. The first time a guest does not pay on time, they will not have his initial fee at the end of the round. And it will not affect the other guests.
   5. From the second time that a guest does not pay on time, they will impact the amount that all guests will receive at the end of the round.
   6. If users have pending payments, they can catch up while the round is running.
4. End of the round
   1. At the end of the round, what is in the safety deposit pool will be distributed among users with timely payments.
5. Round out of funds
   1. If several users miss payments the round could run out of funds to pay the person in turn. If this is the case, the total balance of the round smart contract will be sent to a general account.

**Main.sol**

**This factory pattern smart contract is used to**

| Method      | Parameters  | Required | Type   | Description                       |
| ----------- | ----------- | -------- | ------ | --------------------------------- |
| CreateRound | \_warranty  | Yes      | uint   | Security deposit                  |
|             | \_saving    | Yes      | uint   | Save amount or payment per turn   |
|             | \_groupSize | Yes      | uint   | Number of users size >1 && <=10   |
|             | \_payTime   | Yes      | uint   | Duration of one turn in seconds   |
|             | \_token     | Yes      | IERC20 | Token to user in the saving round |

If the Round creation method is executed correctly, the function returns the address of the new smart contract (SavingGroups).

**Call example:**

contractInstance.methods.createRound(warranty, saving, groupSize, payTime,token)

.send({

from: userAddressWallet,

to: contractAddress

})

**Response example:**

\[

{

"from": "0xd9145CCE52D386f254917e481eB44e9943F39138",

"topic": "0xfdebb0c30fbd3c4fe101775e6bf17ea692258188ce7edb5666366f58f1529ffc",

"event": "RoundCreated",

"args": {

"0": "0xb8f43EC36718ecCb339B75B727736ba14F174d77",

"childRound": "0xb8f43EC36718ecCb339B75B727736ba14F174d77"

}

}

]

The Value of childRound returns the address of the new SavingGroups smart contract. This value changes each time a round is created.

**SavingGroups.sol**

The SavingGroups.sol smart contract features the Stages Enum, enums are a way to create user-defined data types and are implicitly convertible to integers.

| Enum Name | Value          | Description                                                                                                                                                                                            |
| --------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Stage     | Setup -> 0     | In this stage the users register to the saving round. And the admin is able to start the round.                                                                                                        |
|           | Save -> 1      | In this stage the users are able to make payments and withdrawals according to the registration order. No changes in the user list are possible at this stage.                                         |
|           | Finished -> 3  | When the time of the last turn has passed, the saving round can pass to this stage. In this stage the calculation of the security deposit returns is done. No more payments are allowed at this stage. |
|           | Emergency -> 4 | If there are no funds for a withdrawal, the round enters in this stage to indicate the round did not end correctly. The available balance in the SC can be sent to a general account.                  |

The smart contract features a struct with the user information so the smart contract can manage the round correctly. The users are stored in a mapping.

| Struct Name | Parameters         | Type    | Description                                                                                                                                                             |
| ----------- | ------------------ | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| User        | userAddr           | address | Stores the users address                                                                                                                                                |
|             | userTurn           | uint    | Stores the selected turn by the user                                                                                                                                    |
|             | availableCashIn    | uint    | Stores the user’s security deposit balance                                                                                                                              |
|             | availableSavings   | uint    | Stores the user’s fund balance. Represents the deposits other users have done for this user to withdraw                                                                 |
|             | assignedPayments   | uint    | Stores the user’s payments that have been assigned to other users' funds.                                                                                               |
|             | unassignedPayments | uint    | Stores the user’s payments that haven’t been assigned to other users' funds.                                                                                            |
|             | latePayments       | uint    | Stores the number of turns the user has not paid. The user is able to pay later but this value will not be modified                                                     |
|             | owedTotalCashIn    | uint    | Stores how much of the security deposits has been taken because of this user's late payments.                                                                           |
|             | isActive           | bool    | <p>Variable to indicate if the user is registered.<br>If a user registers and is removed, the user is still in the User mapping but has this variable set to false.</p> |
|             | withdrewAmount     | uint    | Stores the amount the user has withdrawn from the round.                                                                                                                |

The smart contract has 6 mandatory parameters shown in the following table.

| Method      | Parameters   | Type            | Description                                               |
| ----------- | ------------ | --------------- | --------------------------------------------------------- |
| constructor | \_cashIn     | uint            | Safety deposit to be paid on registration.                |
|             | \_saveAmount | uint            | Payment in each turn.                                     |
|             | \_groupSize  | uint            | Number of turns >1 && <= 10                               |
|             | \_admin      | Address payable | Wallet address of the round creator, can’t be null (0x0). |
|             | \_payTime    | uint            | Time of one turn in seconds.                              |
|             | \_token      | IERC20          | Token address to be used in the saving round.             |

The constructor is executed only once in the smart contract deployment, that comes from the CreateRound function in Main.sol.

Inside the constructor the amounts of \_saveAmount and \_cashIn are converted to the unit of wei.

The constructor has 3 main validations:

1. Validates that the administrator address is non-zero or null.
2. Validates that the group size is greater than one and less than or equal to ten.
3. Validates that the time given by the user is greater than zero and is multiplied by the seconds in a day.

Example: \_payTime = 2 -> 2 \* 86,400 = 2 days.

Events within Saving Groups.

Events are a simplified way of sending information to external systems like a frontend or a subgraph.

| Event             | Parameter                                          | Description                                                                                                                                       |
| ----------------- | -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| RegisterUser      | address user, uint turn                            | Executed when a user is registered to the saving round. Returns the user address and the turn.                                                    |
| PayCashIn         | address user, bool success                         | Executed when a user is registered to the saving round and the safety deposit is paid. Returns the user address and payment success.              |
| PayFee            | address user, bool success                         | Executed when a user is registered to the saving round and the fee is paid. Returns the address of the remover, user address and payment success. |
| RemoveUser        | address removed by, address user, bool success     | Executed when a user is removed. Returns the user address and payment success.                                                                    |
| PayTurn           | address user , bool success                        | Executed when a user makes a payment to the round. Returns the user address and payment success.                                                  |
| WithdrawFunds     | address user , uint amount, bool success           | Executed when a user whidraws the fund corresponding to their turn. Returns the user address, withdrawn amount and payment success.               |
| EndRound          | address round address , uint start at, uint end at | Executed when the round ended. Returns the round address and the start and end times.                                                             |
| EmergencyWithdraw | Address round address, uint funds transferred      | Executed when the insufficient funds are transferred to a general account. Returns user address and funds transferred.                            |

**Fee Cost.**

The fee cost is currently calculated in the constructor:

feeCost = (saveAmount / 10000) \* 500;

It corresponds to 5%. And it is transferred to the Team Bloinx wallet

**Methods table**

| Method                                                    | Parameters    | Type    | Modifier                              | Validations                                                                   | Description                                                                                                                                                                                                                                                                                               |
| --------------------------------------------------------- | ------------- | ------- | ------------------------------------- | ----------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <p>registeUser</p><p>(public)</p>                         | \_userTurn    | uint    | atStage                               | <p>Is user already registered,</p><p>Is group already full, is turn taken</p> | User inputs the turn for registration ion the round. The user must have already approved the token smart contract.                                                                                                                                                                                        |
| <p>removeUser</p><p>(public)</p>                          | \_userTurn    | uint    | atStage                               | If the caller is the admin, is the turn empty, is the deleted user the admin  | Admin can remove users before in the SetUp stage.                                                                                                                                                                                                                                                         |
| <p>stratRound</p><p>(public)</p>                          |               |         | <p>onlyAdmin</p><p>atStage</p>        | Is group full                                                                 | <p>When all the turns are taken, the admin can start the round.<br>This function advances the stage to Saving.<br>The block time is recorded as the start time of the round from which the turns start counting.<br>No changes in the registered accounts are allowed once this function is executed.</p> |
| <p>addPayment</p><p>(external)</p>                        |               |         | <p>isRegisteredUser</p><p>atStage</p> | Is pay amount correct                                                         | Users can pay any amount up to all planned payments.                                                                                                                                                                                                                                                      |
| <p>withdrawTurn</p><p>(public)</p>                        |               |         | <p>isRegisteredUser</p><p>atStage</p> | Is it time to withdraw for the user, has the user withdrawn                   | This function transfers the fund to the assigned user.                                                                                                                                                                                                                                                    |
| <p>completeSavingsAndAdvanceTurn<br>(private)</p>         | turn          | uint    | atStage                               |                                                                               | <p>Called from addPayment, withdrawTurn and endRound.<br>Executed once in a turn by each user.<br>Transfers funds to the corresponding variables of User and advances turn if the time determines it should.</p>                                                                                          |
| <p>payLateFromSavings<br>(private)</p>                    | \_userAddress | address | atStage                               |                                                                               | <p>If the user owes the security deposit it is taken from the assigned fund to withdraw.<br></p>                                                                                                                                                                                                          |
| <p>emergencyWithdraw<br>(public)</p>                      |               |         | atStage                               | Is stuck balance more than cero                                               | If the round is out of funds to make payments the stuck balance can be sent to a general account.                                                                                                                                                                                                         |
| endRound                                                  |               |         | atStage                               | Is the round finished                                                         | Transfers the corresponding amount of the security deposit to each user                                                                                                                                                                                                                                   |
| <p>futurePayments<br>(public)<br>(view)</p>               |               |         |                                       |                                                                               | Returns the total amount left to pay in the round by the caller                                                                                                                                                                                                                                           |
| <p>obligationAtTime<br>(public)<br>(view)</p>             | userAddress   | address |                                       |                                                                               | Returns the amount the user is required to have paid at the called time.                                                                                                                                                                                                                                  |
| <p>getRealTurn</p><p>(public)<br>(view)</p>               |               |         |                                       |                                                                               | Returns the turn according to the block time when the function is called.                                                                                                                                                                                                                                 |
| <p>getUserAvailableCashIn</p><p>(public)<br>(view)</p>    | \_userTurn    | uint    |                                       |                                                                               | Returns availableCashIn from user struct.                                                                                                                                                                                                                                                                 |
| <p>getUserAvailableSavings</p><p>(public)<br>(view)</p>   | \_userTurn    | uint    |                                       |                                                                               | Returns availableSavings from user struct.                                                                                                                                                                                                                                                                |
| <p>getUserAmountPaid</p><p>(public)<br>(view)</p>         | \_userTurn    | uint    |                                       |                                                                               | Returns assignedPaymentsfrom user struct.                                                                                                                                                                                                                                                                 |
| <p>getUserUnassignedPayments</p><p>(public)<br>(view)</p> | \_userTurn    | uint    |                                       |                                                                               | Returns unassignedPayments from user struct.                                                                                                                                                                                                                                                              |
| <p>getUserLatePayments</p><p>(public)<br>(view)</p>       | \_userTurn    | uint    |                                       |                                                                               | Returns latePayments from user struct.                                                                                                                                                                                                                                                                    |
| <p>getUserOwedTotalCashIn</p><p>(public)<br>(view)</p>    | \_userTurn    | uint    |                                       |                                                                               | Returns owedTotalCashIn from user struct.                                                                                                                                                                                                                                                                 |
| <p>getUserIsActive</p><p>(public)<br>(view)</p>           | \_userTurn    | uint    |                                       |                                                                               | Returns isActive from user struct.                                                                                                                                                                                                                                                                        |

**Modifiers table**

| Method           | SC               | Parameters | Type    | Description                                                                                                                                 |
| ---------------- | ---------------- | ---------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| atStage          | SavingGroups.sol | \_stage    | Stages  | <p>Verifies the current stage.<br>Is used to limit functions to determined stages.</p>                                                      |
| onlyAdmin        | Modifiers.sol    | admin      | address | <p>Verifies if the function caller is the admin.<br>Is used to define the functions available only to the admin.</p>                        |
| isRegisteredUser | Modifiers.sol    | user       | bool    | <p>Verifies if the caller is registered on this round.<br>Is used to define functions only available for users registered to the round.</p> |
