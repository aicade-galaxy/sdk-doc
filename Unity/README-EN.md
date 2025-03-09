# P2 SDK - Unity Version

[TOC]

PS: The term "user" in this document refers specifically to end users.

## 1. Quick Start

### 1.1 Install P2SDK

#### 1.1.1 Download the Installation Package

[P2SDK Installation Package Download](./AicadeSDK.zip "SDK Installation Package")

Download the installation package to your local machine and extract it. The extracted directory contains two files:
* Aicade-SDK.unitypackage
* Aicade.jslib

#### 1.1.2 Install the SDK

* Copy the `Aicade.jslib` file to the `/Assets/Plugins/` directory of your project.
* Open the project and double-click the `AIcadeSDK.unitypackage`. In the pop-up selection dialog, select all and import.

### 1.2 Getting Started

#### 1.2.1 Quick Launch

1. Configure P2SDK Information

Configure application information, including the application ID and encryption details, to allow the platform to recognize the application and ensure its security.

Locate the file `/Assets/Resources/AicadeSettings` and input the configuration obtained from your application registration in the editor window on the right:

* PayMethod: Payment method: Free (free); Once (one-time payment); Subscribe (lifetime subscription);
* Logger Level: Log level. Use Debug during development and Error for production builds;
* Secret: API encryption key;
* IV: API encryption salt.

![Aicade Settings](/Image/1739688410294.jpg "Aicade Settings")

The encryption mechanism primarily ensures security. For example, some merchant operations, such as point consumption, require encryption for processing.

2. Initialize the P2SDK Runtime Environment

To simplify development, we provide a prefab with the `AicadeBootstrap` script attached. Drag the `/Assets/Resources/Aicade/AicadeBootstrap` prefab into the main scene of your project. This game object will automatically initialize the P2SDK environment at runtime.

#### 1.2.2 Using Business Features

For example, querying the balance of a specific token:

``` C#
// Query balance [asynchronous fetcher, remove subscription]
(var fetch, _) = TokenModule.Ins.BalanceOf("Token address");
// Asynchronous operations require callback subscriptions. OnSuccessed is called on success, and OnDone is called on both success and failure.
fetch.OnSuccessed(amount => Debug.Log($"Account balance is {amount}"))
     .OnDone(AicadeBootstrap.Ins.Loaded);
```

## 2. Environment Requirements

* Unity 2021.3.9f1 or later
* WEBGL

## 3. Service Modules

Service modules are implemented as singletons with the naming convention `XXXModule`. They can be accessed using `XXXModule.Ins.XXXX`.

Each service is divided into **Properties**, **Events**, and **Methods**:

* **Properties**: Accessed directly using `XXXModule.Ins.XXX`.
* **Events**: Triggered when important data (usually properties) changes. You need to manually subscribe using `XXXModule.Ins.XXX += OnXXX` and unsubscribe using `XXXModule.Ins.XXX -= OnXXX`.
* **Methods**: Used for active operations and generally involve asynchronous interactions, e.g., `XXXModule.Ins.XXX(xxx, xxx)`.

### 1. Account Services

Module name: `AccountModule`, accessed via `AccountModule.Ins.XXX`.

#### Properties

* `AccountModule.Ins.CurAccount`: Current account information. See SDK for details.
* `AccountModule.Ins.IsLogined`: Indicates if the user is logged in.
* ...

#### [Event] Account Change Listener

If your application needs to handle business logic or UI changes based on user account updates, you can listen to this event.

> `AccountModule.Ins.OnUpdateAccount`

Example:

``` C#
private void Awake() {
  // Add event listener
  AccountModule.Ins.OnUpdateAccount += OnUpdateAccount;
}

private void OnDestroy() {
  // Remove event listener
  AccountModule.Ins.OnUpdateAccount -= OnUpdateAccount;
}

private void OnUpdateAccount() {
  // TODO: Handle account update logic
  AccountModule.Ins.CurAccount;
}
```
---

#### [Method] Get Account Information

In certain scenarios, account information may change without being updated in the SDK. If your application requires strict synchronization of this information, you can use this method to fetch the latest account data.

> `AccountModule.Ins.GetAccountInfo(int timeoutMill = 5000)`

* `timeoutMill`: Timeout duration in milliseconds. If the operation exceeds this time, a failure callback is triggered. Default is 5000 milliseconds.

Example:

``` C#
// Fetch account information [asynchronous fetcher, remove subscription]
(var fetch, _) = AccountModule.Ins.GetAccountInfo();
// Asynchronous operations require callback subscriptions. OnSuccessed is triggered on success, and OnDone is triggered on both success and failure.
fetch.OnSuccessed(account => Debug.Log("Account info is %o", account));
```
----

#### [Method] Data Signing

In scenarios where the application needs to consume user assets, for security reasons, users are required to sign the consumption-related data with their wallets. Both the consumption scenarios and data must be reviewed and registered within the platform before they can be used.

Supported scenarios that can be applied for include:
* ...

``` C#
AccountModule.Ins.Sign(
          string scene,
          Dictionary<string, string> data,
          int timeoutMill = 60000) 
```

* scene Scene, which needs to be registered
* data Data required for the scene
* timeoutMill Timeout duration in milliseconds

Example

``` C#
// Test {time}::{a}::345::{b}
var data = new Dictionary<string, string>();
data["a"] = "ta";
data["b"] = "bbbb";
(var fetch, _) = AccountModule.Ins.Sign("Test", data);
// Retrieve signature result
fetch.OnSuccessed(signMessage => Debug.Log($"Test sign message is {signMessage}"));
```

### 2. Token

Service module name: `TokenModule`, accessed via `TokenModule.Ins.XXX`

#### Properties

None

#### [Method] Get Token Balance

Supports retrieving the token balance of a user at a specific address.

``` C#
TokenModule.Ins.BalanceOf(string tokenAddress, int timeoutMill = 5000)
```

* `address` Token address  
* `timeoutMill` Timeout duration in milliseconds  

Example:

``` C#
// Get token balance
(var fetch, _) = TokenModule.Ins.BalanceOf("0xkkkkkkk");
// Asynchronous operations require callback subscriptions. OnSuccessed is triggered on success, and OnDone is triggered on both success and failure.
fetch.OnSuccessed(balance => Debug.Log($"Account balance is {balance}"));
```

#### [Method] Token Swap

If there is a need for token swapping within the application, this method can be used to handle it.

``` C#
TokenModule.Ins.Swap(TokenSwapInfo swapRequest, int timeoutMill = 5000)
```
* `swapRequest` Swap request  
  * `tokenIn` Address of the token to sell  
  * `tokenOut` Address of the token to buy  
  * `amount` Amount of the token to sell  
* `timeoutMill` Timeout duration in milliseconds  

Example:

``` C#
var swapRequest = new TokenModule.TokenSwapInfo()
{
    tokenIn = "0xaaaaaaaaaaaaaaaaa",
    tokenOut = "0xbbbbbbbbbbbbb",
    amountIn = "5.9" // ether unit
};
var (fetch, _) = TokenModule.Ins.Swap(swapRequest);
fetch.OnSuccessed(amountOutEther => Debug.Log($"The amount of tokens exchanged is {amountOutEther}"));
```
### 3. Ticket

Used to check whether the user has permission to use this application.

Service module name: `TicketModule`, accessed via `TicketModule.Ins.XXX`

#### Properties

* `TicketModule.Ins.CanPlay` Indicates whether the current user can use this application
* ...

#### [Method] Force Check Status

Forces a check on the user's application usage status. If it is not free, a payment popup will be automatically triggered.

> ``` TicketModule.Ins.CheckTicket(System.Action _onDone, System.Action _onError = null) ```

* `_onDone` Success callback  
* `_onError` Failure callback  
* ...

Example:

``` C#
TicketModule.Ins.CheckTicket(() => {
    // User already has permission to use
});
```


#### [Scenario] Free Application

Applications where the `Pay Method` is `Free`.

No additional actions required.

#### [Scenario] Lifetime Subscription Application

Applications where the `Pay Method` is `Subscribe`.

> For lifetime subscription applications, the user needs to pay a subscription fee the first time they use the app. After subscribing, they can use it permanently.

Example:

``` C#
// Check permissions. If the user has not subscribed, a subscription popup will be triggered until the subscription is completed.
TicketModule.Ins.CheckTicket(() => {
    // TODO: User permission check succeeded
});
```
#### [Scenario] One-Time Payment Application

Applications where the `Pay Method` is `Once`.

> For `Once` applications, each usage requires a new payment. The standard for determining whether the current usage is complete is defined by the application itself.  
> Process: Purchase a ticket -> Use the application -> Deduct the ticket.

Example:

``` C#
// Check if the last ticket has been used
if (TicketModule.Ins.CanPlay) {
    // Can use the application
    return;
}
// If no ticket is available, a payment popup for the ticket will be triggered
TicketModule.Ins.CheckTicket(() => {
    // TODO: User permission check succeeded
});

// Deduct the ticket
TicketModule.Ins.PurchaseTicket(() => {
    // Ticket deduction successful
});
```
### 4. Application Score Management

Application scores are primarily designed to help service providers implement leaderboard-style operational features in offline scenarios.

Service module name: `AppScoreModule`, accessed via `AppScoreModule.Ins.XXX`.

#### Properties

* `CurScore` Current user's application score.

#### [Event] Score Change

Notification triggered when the score changes.

``` C#
AppScoreModule.Ins.OnUpdateScore += OnUpdate;
```
Example:

``` C#
private void Awake() {
    // Add event listener
    AppScoreModule.Ins.OnUpdateScore += OnUpdateScore;
}

private void OnDestroy() {
    // Remove event listener
    AppScoreModule.Ins.OnUpdateScore -= OnUpdateScore;
}

private void OnUpdateScore() {
    // TODO: Handle specific business logic for score updates
    Debug.Log($"Current Score: {AppScoreModule.Ins.CurScore}");
}
```

#### [Method] Get Real-Time User Rankings Within the Application

``` C#
AppScoreModule.Ins.GetRanks()
```

Example:

``` C#
var (fetch, _) = AppScoreModule.Ins.GetRanks();
fetch.OnSuccessed(ranks => {
    foreach (var rank in ranks) {
        Debug.Log($"avatar  = {rank.avatar}");
        Debug.Log($"ranking = {rank.ranking}");
        Debug.Log($"score   = {rank.score}");
        Debug.Log($"address = {rank.walletAddress}");
    }
});
```

#### [Method] Modify User Score

This method modifies the application score for the user within the application. It can only be used to modify the user's score in the current application.

Typically, it is used for rewarding or consuming points within the app.

``` C#
AppScoreModule.Ins.UpdateScore(int _score, int timeout = 5000)
```
* _score The amount of score to change. Positive values increase the score, and negative values decrease it.


Example:

``` C#
var (fetch, _) = AppScoreModule.Ins.UpdateScore(10);
fetch.OnSuccessed(score => Debug.Log($"Cur score is {score}"));
```


#### [Method] Clear Application Score

Directly clears all scores for the user under the current application.

``` C#
AppScoreModule.Ins.ClearScore()
```

Example:

``` C#
var (fetch, _) = AppScoreModule.Ins.ClearScore();
fetch.OnSuccessed(() => Debug.Log("OK"));
```

---

#### [Method] Convert Application Score to Platform Score (Aicade Score Token)

If the application has operations to encourage users, such as converting application scores into platform scores, this interface can be used to deduct specific application scores and increase the user's platform scores.

> The platform scores added to the user will be deducted from the application merchant's account. Therefore, the application merchant's account balance must be greater than or equal to the amount being deducted.

``` C#
ExchangeAicadeScore(int appScore, int aicadeToken, string scene, int timeout = 5000)
```

* `appScore` The amount of application score to deduct from the user.
* `aicadeToken` The amount of platform score to add to the user, credited to their centralized account.
* `scene` The operation scene, allowing the user to view the record of platform score changes in their account.

Example:

``` C#
// Convert scores
var (fetch, _) = AppScoreModule.Ins.ExchangeAicadeScore(
    AppScoreModule.Ins.CurScore,
    AppScoreModule.Ins.CurScore / 100,
    "Exchange"
);
fetch.OnSuccessed((_1, _2) => Debug.Log("OK"));
```

---

### 5. Platform Point Token Management (Aicade Token Point)

Used to manage the balance of platform point tokens in the user's centralized account. If the user wants to use the point tokens in the app, they must first transfer the tokens from their wallet to the centralized account.

Service module name: `AicadeTokenModule`, accessed via `AicadeTokenModule.Ins.XXX`.

#### Properties

* `CurPoint` The current account balance.

---

#### [Event] Point Update Listener

``` C#
AicadeTokenModule.Ins.OnUpdatePoint += OnUpdatePoint
```

Example:

``` C#
private void Awake() {
    // Add event listener
    AicadeTokenModule.Ins.OnUpdatePoint += OnUpdatePoint;
}

private void OnDestroy() {
    // Remove event listener
    AicadeTokenModule.Ins.OnUpdatePoint -= OnUpdatePoint;
}

private void OnUpdateScore() {
    // TODO: Implement specific listener logic
    Debug.Log($"Current Point: {AicadeTokenModule.Ins.CurPoint}");
}
```

---

#### [Method] Consume Point

Consumes the user's platform point within the application. This operation requires the user to confirm via wallet signature before proceeding.

``` C#
Consume(long amount, string scene, int timeoutMill = 5000)
```

* `amount` The amount to consume.
* `scene` The consumption scene.

Example:

``` C#
// Consume 1000 Aicade token point
var (fetch, _) = AicadeTokenModule.Ins.Consume(1000, "Play");
fetch.OnSuccessed(_ => Debug.Log("OK"));
```

---

#### [Method] Reward Point

If the application has operations to reward points, this interface can be used.

> The platform points rewarded to the user will be deducted from the application merchant's account. Therefore, the application merchant's account balance must be sufficient.

``` C#
AicadeTokenModule.Ins.Obtain(long amount, string scene, int timeoutMill = 5000)
```

* `amount` The amount of point to reward.
* `scene` The reward scene.

Example:

``` C#
// Reward 1000 Aicade token point
var (fetch, _) = AicadeTokenModule.Ins.Obtain(1000, "Rank100");
fetch.OnSuccessed(_ => Debug.Log("OK"));
```




