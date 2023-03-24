# EventManager

- Use a separate class to declare delegates and a separate class to Manage and Bind events
- In the Sender class, use a separate function to Send the event and another one to get the message event - which will be the delegate object MessageEvent. The GetMessageEvent() function will be called from the EventManager.
- In the Receiver classe, use a separate funtion to Receive the event and another ont to add the message event - the delegate object MessageEvent - and bind it the the to the listener function Receive().

- AddListener(): iterates through the Senders / Delegate function and Binds the Listener function to it. 
  - If I am a Receiver, I will add a Listener function to the Binding.
  - In the EventManager class, GetMessageEvent() will be used to fetch the delegate object FMessageEvent and pass it to the Receiver so that it can bind the listener function Receive() to the delegate function Send()
  
- AddInvoker(): itarates through all the Receivers / Listener functions and binds a Delegate function to it.
  - If I am a Sender, I will ad an Invoker / Delegate function to the Binding.
  - In the EventManager class, GetMessageEvent() will be used to fetch the delegate object FMessageEvent and pass it to the Receiver so that it can bind the listener function Receive() to the delegate function Send()

- In Receiver class

- Add a DelegateDeclarationsContainer C++ Class (Object type) to store all the delegate declarations of the game
- Add an Event Manager C++ Class (AActor type)
  - Create a BP based on this class
  - Add it to the scene
  - In project settings, Create and new Gamaplay Tag "Event Manager" and tag this BP with this tag

- Delegate Declarations Container
  - Declare the delegates and their objects here -> delegate object will be FMessageEvent
  
```cpp
DECLARE_MULTICAST_DELEGATE_OneParam(FMessageEvent, FString); 

UCLASS()
class MULTICAST_DELEGATES_API UDelegateDeclarationsContainer : public UObject
{
	GENERATED_BODY()

};
```
  
- Sender class

  - Sender header file

- Receiver class

- Event Manager
  - In the event manager header file, include: 
    - #include "Sender.h"
    - #include "Receiver.h"
    - #include "Delegates/DelegateInstanceInterface.h"
  - Create a TArray of type Sender pointers and expose it with UPROPERTY
  - Create a MAP
    - Maps store series of key-value pairs in which you can search for the key and it returns its value
    - It is more efficient to use maps instead of arrays or vectors because with arrays or vectors you would need to traverse their entire length and look into both the keys and each of their contents to search for the right component. With maps you just need to scan through the keys, and not each of their contents also, making the search faster.
  
  
