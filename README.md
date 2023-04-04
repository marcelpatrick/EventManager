# EventManager

- We will use a separate class to declare delegates (DelegateDeclarationsContainer) and a separate class to Manage events (EventManager)
- Create a c++ class and a blueprint derived from this class for the Sender, Receiver and EventManager.
- For the EventManager, tag it with "EventManager" tag in project settings and, in its blueprint, create a tag with the same name

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

```cpp
#include "DelegateDeclarationsContainer.h" 

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "Sender.generated.h"


UCLASS()
class MULTICAST_DELEGATES_API ASender : public AActor
{
	GENERATED_BODY()
	
public:	
	// Sets default values for this actor's properties
	ASender();

	void Find();

	void Send(); 

	FMessageEvent& GetMessageEvent();

	void StartTimer();

	UFUNCTION()
	virtual void EndPlay(const EEndPlayReason::Type EndPlayReason);

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

private:
	FMessageEvent MessageEvent; 
	FTimerHandle MyTimerHandle;
};
```
```cpp
#include "Sender.h"
#include "Kismet/GameplayStatics.h" 
#include "GameFramework/Actor.h" 
#include "Engine/World.h"

#include "EventManager.h"


// Sets default values
ASender::ASender()
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

}

// Called when the game starts or when spawned
void ASender::BeginPlay()
{
	Super::BeginPlay();

	Find();

	StartTimer();
	
}

// Called every frame
void ASender::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

}


void ASender::Find()
{
	//Add Sender as an invoker to the event manager
	//Find all actors with tag EventManager and add in the TaggedActors array
	TArray<AActor*> TaggedActors;
	UGameplayStatics::GetAllActorsWithTag(GetWorld(), "EventManager", TaggedActors);

	//If found, cast the first event manager actor to an EventManager type and save into an EventManager object
	//Use the EventManager object to call its AddInvoker() function passing this Sender actor as a parameter
	if (TaggedActors.Num() > 0)
	{
		AEventManager* EventManager = Cast<AEventManager>(TaggedActors[0]);

		EventManager->AddInvoker(this);
	}
	else
	{
		UE_LOG(LogTemp, Warning, TEXT("NO ACTOR FOUND !!!"));

		return;
	}

}

FMessageEvent& ASender::GetMessageEvent()
{
	UE_LOG(LogTemp, Warning, TEXT("GET MESSAGE EVENT !!!"));

	return MessageEvent;
}


void ASender::StartTimer()
{
	UE_LOG(LogTemp, Warning, TEXT("TIMER STARTED !!!"));

	GetWorldTimerManager().SetTimer(MyTimerHandle, this, &ASender::Send, 3, true);
}


void ASender::Send()
{
	UE_LOG(LogTemp, Warning, TEXT("SEND !!!"));

	//Sender broadcasts the delegate function to anyone who wants to subscribe to it (doesn't need to care who will subscribe)
	MessageEvent.Broadcast("This is the message from the send function in the Delegate");
}


void ASender::EndPlay(const EEndPlayReason::Type EndPlayReason)
{
	//UNBIND
	TArray<AActor*> TaggedActors;
	UGameplayStatics::GetAllActorsWithTag(GetWorld(), "EventManager", TaggedActors);

	if (TaggedActors.Num() > 0)
	{
		AEventManager* EventManager = Cast<AEventManager>(TaggedActors[0]);

		EventManager->RemoveInvoker(this);
	}
}
```

# Receiver
- Receiver is the Callback class. It will Find the EventManager actors in the world, Bind the Delegate object to the Callback function and receive the event in the callback function.
- Create a Find() function to find all EventManager objects in the world and use it to pass an instance of the Receiver class to the EventManager class - AdddListener() - so that the EventManager class can be the one responsible for sending the delegate object - MessageEvent - back to the Receiver class. -> So that the transaction of the delegate object MessageEvent doesn't happen directly between the Sender and Receiver classes but rather managed by the event manager: Sender > MessageEvent > EventManager > MessageEvent > Receiver
- Create a binding function AddToMessageEvent()
- Create a callback function Receive()

```cpp
#include "DelegateDeclarationsContainer.h"

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "Sender.h"
#include "Receiver.generated.h"


UCLASS()
class MULTICAST_DELEGATES_API AReceiver : public AActor
{
	GENERATED_BODY()
	
public:	
	// Sets default values for this actor's properties
	AReceiver();

	void Find();
	//void Bind();

	FDelegateHandle AddToMessageEvent(FMessageEvent& MessageEvent);

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

	UFUNCTION()
	void Receive(FString Message);

	UFUNCTION()
	virtual void EndPlay(const EEndPlayReason::Type EndPlayReason);

private:
	ASender* MySender;
};
```
```cpp

#include "Receiver.h"
#include "Engine/World.h"
#include "Kismet/GameplayStatics.h"
#include "EventManager.h"


// Sets default values
AReceiver::AReceiver()
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

}

// Called when the game starts or when spawned
void AReceiver::BeginPlay()
{
	Super::BeginPlay();

	Find();
	
}

// Called every frame
void AReceiver::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

}

void AReceiver::Find()
{
	//Find all EventManager actors in the world and save them in the actor array
	TArray<AActor*> TaggedActors;
	UGameplayStatics::GetAllActorsWithTag(GetWorld(), "EventManager", TaggedActors);

	//If actor found, get the first one in the array, cast it to an event manager type and use this event manager object to call its add listener function adding this Receiver as the listener in the event manager
	if (TaggedActors.Num() > 0)
	{
		AEventManager* EventManager = Cast<AEventManager>(TaggedActors[0]);

		EventManager->AddListener(this);
	}
	else
	{
		UE_LOG(LogTemp, Warning, TEXT("No actor found!!!"));

		return;
	}
}

FDelegateHandle AReceiver::AddToMessageEvent(FMessageEvent& MessageEvent)
{
	UE_LOG(LogTemp, Warning, TEXT("ADD TO MESSAGE EVENT!!!"));

	//BINDING: Receiver class subscribes the Receive() function to the Delegate function in the EventManager class
	return MessageEvent.AddUObject(this, &AReceiver::Receive);
}

void AReceiver::Receive(FString String)
{
	UE_LOG(LogTemp, Warning, TEXT("RECEIVE !!!"));

	UE_LOG(LogTemp, Warning, TEXT("The received message is: %s"), *String);
}

void AReceiver::EndPlay(const EEndPlayReason::Type EndPlayReason)
{
	TArray<AActor*> TaggedActors;
	UGameplayStatics::GetAllActorsWithTag(GetWorld(), "EventManager", TaggedActors);

	if (TaggedActors.Num() > 0)
	{
		AEventManager* EventManager = Cast<AEventManager>(TaggedActors[0]);

		EventManager->RemoveListener(this);
	}
}
```

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
```cpp
#include "Sender.h"
#include "Receiver.h"
#include "Delegates/DelegateInstanceInterface.h"

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "EventManager.generated.h"

UCLASS()
class MULTICAST_DELEGATES_API AEventManager : public AActor
{
	GENERATED_BODY()
	
public:	
	// Sets default values for this actor's properties
	AEventManager();

	//Add the Sender actor as a broadcaster / invoker of the message event
	void AddInvoker(ASender* Sender); 

	//Remove Sender actor as a broadcaster / invoker of the message event in case they are removed from the game or level
	void RemoveInvoker(ASender* Sender);

	//Add the Receiver actor as a listener of the message event
	void AddListener(AReceiver* Receiver);

	//Remove the Receiver actor as a listener of the message event in case they get removed from the game or level
	void RemoveListener(AReceiver* Receiver);
	

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

private:
	//Create an group of broadcasters / invokers of events (Senders)
	UPROPERTY()
	TArray<ASender*> MySenders;
	//Create a group of Receivers of events.
		//This group needs to contain the pointer to each Receiver listener function and also their FDelegateHandle that got associated to each respective Sender delegate function (so that we can remove their connection to their delegate function once the games ends)
		//Therefore we have to create a map to store a key-value pair for each Receiver-FDelegateHandle object 
			//And since each FDelegateHandle variable from the Receiver is associated to its respective Sender delegate, we need to pass another map to find them using the key-value pairs: Sender-FDelegateHandle object
	TMap<AReceiver*, TMap<ASender*, FDelegateHandle>> MyReceivers;
};
```

```cpp
  #include "EventManager.h"

// Sets default values
AEventManager::AEventManager()
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

}

// Called when the game starts or when spawned
void AEventManager::BeginPlay()
{
	Super::BeginPlay();
	
}

// Called every frame
void AEventManager::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

}


void AEventManager::AddInvoker(ASender* Sender)
{
	//Add the invoker that will be passed into this function to our array of Sender objects
	MySenders.Add(Sender);

	//Iterate through the listeners (Receivers) and attach this new invoker to one of them so that it can listen when the invoker broadcasts the event
	//auto && - when the compile automatically infers the type of variables in the array
	for (auto& i : MyReceivers) 
	{
		//Add a listener to the message event that is inside the Sender when it is broadcasting
		//Take the key of the current index - which is a Receiver object - call its Receive() function passing in the message event 
			//To get the message event, use the Sender object received by this function - which is a Sender object - and call its Send() function. 
		FDelegateHandle MyDelegateHandle = i.Key->AddToMessageEvent(Sender->GetMessageEvent());

		//Take the value of the current index and add a DelegateHandle to it (finding the correspondent DelegateHandle by its key, the invoker)
		i.Value.Add(Sender, MyDelegateHandle);
	}
}

void AEventManager::RemoveInvoker(ASender* Sender)
{
	//Iterate through the event listeners array and remove its value and key
	for (auto &&i : MyReceivers)
	{
		if (i.Value.Contains(Sender))
		{
			Sender->GetMessageEvent().Remove(i.Value[Sender]);
			i.Value.Remove(Sender);
		}
	}

	//Remove this invoker from the TArray of Sender objects
	MySenders.Remove(Sender);
}

void AEventManager::AddListener(AReceiver* Receiver)
{
	//Add listener to the map of listeners (Receivers)
	MyReceivers.Add(Receiver);

	//Iterate through all the Senders and for each one hook up a new listener to it
	for (auto &&i : MySenders)
	{
		FDelegateHandle MyDelegateHandle = Receiver->AddToMessageEvent(i->GetMessageEvent());

		MyReceivers[Receiver].Add(i, MyDelegateHandle);
	}
}

void AEventManager::RemoveListener(AReceiver* Receiver)
{
	for (auto &&i : MySenders)
	{
		if (MyReceivers[Receiver].Contains(i))
		{
			i->GetMessageEvent().Remove(MyReceivers[Receiver][i]);
			MyReceivers[Receiver].Remove(i);
		}
	}
	//Remove listener (Receiver)
	MyReceivers.Remove(Receiver);
}
```
