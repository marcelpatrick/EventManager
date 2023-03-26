# EventManager

- We wil use a separate class to declare delegates (DelegateDeclarationsContainer) and a separate class to Manage events (EventManager)

# Delegate Declarations Container
- It will declare the multicast delegate type object in its header file
```cpp
#include "CoreMinimal.h"
#include "UObject/NoExportTypes.h"
#include "DelegateDeclarationsContainer.generated.h"


DECLARE_MULTICAST_DELEGATE_OneParam(FMessageEvent, FString); 

UCLASS()
class MULTICAST_DELEGATES_API UDelegateDeclarationsContainer : public UObject
{
	GENERATED_BODY()

};
```

# Sender
- Sender is the Delegate class. It will broadcast the event and provide the multicast delegate object so that the EventManger class can use it to bind to the Callback function.
- Create a Find() function to find the all EventManager object in the world and use it to pass an instance of the Sender class to the EventManager class - AddInvoker() so that the EventManager class can use this instance to receive the delegate object from the Sender class.
- Create a Send() Delegate function which will broadcast the event - GetMessageEvent()
- Create the GetMessageEvent() function which will return the multicast delegate object - MessageEvent - of the type that we declared in the DelegateDeclarationsContainer class.
- Create a StartTimer() function to call the Send() delegate function after 3 sec. If no timer is used, de delegate function will try to broadcast the event before the binding occurs in the callback class and it won't work.
- Create an EndGame() function to unbind the delegate from the callback function once the game has ended


# Receiver
- Receiver is the Callback class. It will Find the EventManager actors in the world, Bind the Delegate object to the Callback function and receive the event in the callback function.
- Create a Find() function to find all EventManager objects in the world and use it to pass an instance of the Receiver class to the EventManager class - AdddListener() - so that the EventManager class can be the one responsible for sending the delegate object - MessageEvent - back to the Receiver class. -> So that the transaction of the delegate object MessageEvent doesn't happen directly between the Sender and Receiver classes but rather managed by the event manager: Sender > MessageEvent > EventManager > MessageEvent > Receiver
- Create a binding function AddToMessageEvent()
- Create a callback function Receive()

# Event Manager
- It will get the delegate object, send it to the bind function in the callback class so that it can bind it to its callback function.
  - In the event manager header file, include: 
    - #include "Sender.h"
    - #include "Receiver.h"
    - #include "Delegates/DelegateInstanceInterface.h"
  - Create a TArray of type Sender pointers and expose it with UPROPERTY
  - Create a MAP
    - Maps store series of key-value pairs in which you can search for the key and it returns its value
    - It is more efficient to use maps instead of arrays or vectors because with arrays or vectors you would need to traverse their entire length and look into both the keys and each of their contents to search for the right component. With maps you just need to scan through the keys, and not each of their contents also, making the search faster.

- AddListener(): iterates through the Senders / Delegate function and Binds the Listener function to it. 
  - Adds a callback object to the MyReceivers map.
  - If I am a Receiver, I will add a Listener function to the Binding.
  - In the EventManager class, GetMessageEvent() will be used to fetch the delegate object FMessageEvent and pass it to the Receiver so that it can bind the listener function Receive() to the delegate function Send()
  
- AddInvoker(): itarates through all the Receivers / Listener functions and binds a Delegate function to it.
  - Adds a delegate object to the Senders array.
  - If I am a Sender, I will ad an Invoker / Delegate function to the Binding.
  - In the EventManager class, GetMessageEvent() will be used to fetch the delegate object FMessageEvent and pass it to the Receiver so that it can bind the listener function Receive() to the delegate function Send()

- Then add RemoveInvoker() and RemoveListener() to remove the callback and sender objects to their respective maps and arrays.
  
  
