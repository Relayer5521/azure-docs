---
author: probableprime
ms.service: azure-communication-services
ms.topic: include
ms.date: 6/20/2025
ms.author: cnwankwo
---
[!INCLUDE [Install SDK](../install-sdk/install-sdk-web.md)]


Call transfer is an extended feature of the core `Call` API. You first need to import calling Features from the Calling SDK:

```js
import { Features} from "@azure/communication-calling";
```

Then you can get the transfer feature API object from the call instance:

```js
const callTransferApi = call.feature(Features.Transfer);
```

Call transfers involve three parties:

- *Transferor*: The person initiating the transfer request.
- *Transferee*: The person being transferred.
- *Transfer target*: The person the call is being transferred to.

### Transfer to participant:

1. There's already a connected call between the *transferor* and the *transferee*. The *transferor* decides to transfer the call from the *transferee* to the *transfer target*.
1. The *transferor* calls the `transfer` operation.
1. The *transfer target* receives an incoming call.

To transfer a current call, you can use the `transfer` operation. The `transfer` operation takes the optional `transferCallOptions`, enabling you to set a `disableForwardingAndUnanswered` flag:

- `disableForwardingAndUnanswered = false`: If the *transfer target* doesn't answer the transfer call, the transfer follows the *transfer target* forwarding and unanswered settings.
- `disableForwardingAndUnanswered = true`: If the *transfer target* doesn't answer the transfer call, the transfer attempt ends.

```js
// transfer target can be an Azure Communication Services user
const id = { communicationUserId: <ACS_USER_ID> };
```

```js
// call transfer API
const transfer = callTransferApi.transfer({targetParticipant: id});
```

### Transfer to call:

1. There's already a connected call between the *transferor* and the *transferee*. 
2. There's already a connected call between the *transferor* and the *transfer target*.
3. The *transferor* decides to transfer the call with the *transferee* to the call with *transfer target*.
4. The *transferor* calls the `transfer` operation.
6. The *transfer target* receives an incoming call.

To transfer a current call, use `transfer`.

```js
// transfer to the target call specifying the call id
const id = { targetCallId: <CALL_ID> };
```

```js
// call transfer API
const transfer = callTransferApi.transfer({ targetCallId: <CALL_ID> });
```

The `transfer` Aobject enables you to subscribe to `stateChanged`. It also comes with a  transfer `state` and `error` properties

```js
// transfer state
const transferState = transfer.state; // None | Transferring | Transferred | Failed

// to check the transfer failure reason
const transferError = transfer.error; // transfer error code that describes the failure if a transfer request failed
```

The *transferee* can listen to a `transferAccepted` event. The listener for this event has a `TransferEventArgs`, which contains the call object of the new transfer call
between the  *transferee* and the *transfer target*. 

```js
// Transferee can subscribe to the transferAccepted event
callTransferApi.on('transferAccepted', args => {
    const newTransferCall =  args.targetCall;
});
```

The *transferor* can subscribe to events for change of the state of the transfer. If the call to the *transferee* was successfully connected with *Transfer target*, *transferor* can hang up the original call with *transferee*.

```js
transfer.on('stateChanged', () => {
   if (transfer.state === 'Transferred') {
       call.hangUp();
   }
});
```

### Transfer to voicemail:

1. There's a connected call between the *transferor* and the *transferee*. 
2. The Teams User Identifier of the *target participant voicemail* is known.
3. The *transferor* decides to transfer the call with the *transferee* to the *target participant's voicemail* using the target participant's Teams User Identifier.
4. The *transferor* calls `transfer`.
5. The *transferee* receives the transfer request.

To transfer a current call, you can use `transfer`.

```js
// transfer to the target participant voicemail specified by their Teams User Identifier
const id: MicrosoftTeamsUserIdentifier = { microsoftTeamsUserId: userId}
```

```js
// call transfer API
const transfer = callTransferApi.transfer({ targetParticipantVoicemail: id });
```

The `transfer` operation enables you to subscribe to `stateChanged`. It also comes with a transfer `state` and `error` properties

```js
// transfer state
const transferState = transfer.state; // None | Transferring | Transferred | Failed

// to check the transfer failure reason
const transferError = transfer.error; // transfer error code that describes the failure if a transfer request failed
```

The *transferee* can listen to a `transferAccepted` event. The listener for this event has a `TransferEventArgs`, which contains the call object of the new transfer call
between the *transferee* and the *target participant voicemail*. 

```js
// Transferee can subscribe to the transferAccepted event
callTransferApi.on('transferAccepted', args => {
    const newTransferCall =  args.targetCall;
});
```

The *transferor* can subscribe to events for change of the state of the transfer. If the call to the *transferee* successfully connected with *target participant voicemail*, *transferor* can hang up the original call with *transferee*.

```js
transfer.on('stateChanged', () => {
   if (transfer.state === 'Transferred') {
       call.hangUp();
   }
});
```

### Initial Caller and Transferor information

When a participant forwards or transfers a call, `transferInfo` is populated with information about the prior call state. This information includes `callerInfo`, which describes the initial caller and `transferorInfo`, which describes the entity transferring or forwarding the call.

For example, if an Azure Communication Services user places a call to a Teams call queue, which then distributes the call to a Microsoft 365 user, the `callerInfo` would specify the Azure Communication Services user and the `transferorInfo` would specify the Teams call queue. Callers and transferors have the ability to update their `displayName`. If they do, the change triggers the `callerInfoChanged` or `transferorInfoChanged` events.

For more information about change events, see [Event: callerInfoChanged](../../events.md?pivots=platform-web#event-callerinfochanged) and [Event: transferorInfoChanged](../../events.md?pivots=platform-web#event-transferorinfochanged). Change events apply to all calls and for any identity, including bring your own identity (BYOI) or Microsoft 365.

```js
const incomingCallHandler = async (args: { incomingCall: IncomingCall }) => {
    const incomingCall = args.incomingCall;
    // Get information about initial caller
    const callerInfo = incomingCall.callerInfo
    // Get information about initial caller
    const transferorInfo = incomingCall.transferorInfo
};
```
