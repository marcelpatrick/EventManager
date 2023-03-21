# EventManager

- Add a Delegate Declarations C++ Class (UObject or Empty class type)
- Add an Event Manager C++ Class (AActor type)
  - Create a BP based on this class
  - Add it to the scene
  - In project settings, Create and new Gamaplay Tag "Event Manager" and tag this BP with this tag

- Event Manager
  - In the event manager header file, include: 
    - #include "Sender.h"
    - #include "Receiver.h"
    - #include "Delegates/DelegateInstanceInterface.h"
  - Create a TArray of type Sender pointers and expose it with UPROPERTY
  - Create a MAP
    - Maps store series of key-value pairs in which you can search for the key and it returns its value
    - It is more efficient to use maps instead of arrays or vectors because with arrays or vectors you would need to traverse their entire length and look into both the keys and each of their contents to search for the right component. With maps you just need to scan through the keys, and not each of their contents also, making the search faster.
  
  
