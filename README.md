
# Falco Unreal 5 Coding Conventions

This document summarizes the high-level coding conventions for writing Unreal client code at Falco. They are based on the [Elysium-Game-Studio coding conventions](https://github.com/Elysium-Game-Studio/unreal-coding-conventions).

The goal is to make it easier to work in similar teams inside and outside the company, as well as have client code blend in with other code of the Unreal API. We are providing a complete summary here in order to allow people to understand the conventions at a glance, instead of having to open multiple documents. Our coding conventions are numbered, which makes it easier to refer to them in code reviews.


## 1. Namespaces

1.1. __DO NOT__ use namespaces to organize your game classes. Namespaces are not supported by UnrealHeaderTool, so they can't be used when defining `UCLASS`es, `USTRUCT`s etc., and we don't want to put half of our game code in a namespace and the other half in global scope.

1.2. __DO__ add a prefix to all class and struct names, based on your project name.


## 2. Files

2.1. __DO__ use PascalCase for file names, with the project name prefix, but without the type prefix (e.g. `AHOATCharacter` goes into `HOATCharacter.h`).

2.2. __DO__ sort includes alphabetically between the ``CoreMinimal`` and the generated class header files

2.3. __DO__ write header files with the following structure:

* copyright notice
* line break
* `#pragma once`
* line break
* `#include` of the `CoreMinimal` file
* `#include` of the base class header, if any (e.g. `#include "GameFramework/Character.h"`)
* line break
* `#include` of the generated class header (e.g. `#include "HOATCharacter.generated.h"`)
* line break
* forward declarations for any referenced engine or game types with line break between each of them
* line break
* delegate declarations
* line break
* type definition

```cpp
// Copyright FALCO CAYMAN. All Rights Reserved.

#pragma once

#include "CoreMinimal.h"
#include "AbilitySystemInterface.h"
#include "GameFramework/Character.h"

#include "THCharacter.generated.h"

class UAbilitySystemComponent;

class UStateTreeComponent;

DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FTHCharacterDamageSufferedSignature, ATHCharacter*, Character);

DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FTHCharacterDeathSignature, ATHCharacter*, Character);

USTRUCT(BlueprintType)
struct THEHARVEST_API FTHAbilityDefinition
{
    // ...
};

UCLASS()
class THEHARVEST_API ATHCharacter : public ACharacter, public IAbilitySystemInterface
{
    // ...
};
```

2.3. __DO__ define classes with the following structure:


* Variables Section
    * public
        * constants
        * static
        * fields
        * `UPROPERTY`s

    * protected fields
        * same order as public

    * private fields
        * same order as public

* Functions Section
    * public functions
        * ctor and dtor
        * operators
        * statics
        * virtual overrides
        * virtuals
        * other functions
        * event handlers - functions starting with `Handle`

    * protected functions
        * same order as public functions

    * private functions
        * same order as public functions

Always put blank lines between each of the members.

Within each of these groups, order members by logical groups when appropriate.

2.4. __DO NOT__ cover more than a single feature in each file. Don't define more than one public type per file

## 3. Includes

3.1. __DO NOT__ include unused headers. This will generally help reduce the compilation time, especially for developers when just one header has been modified. It may also avoid errors that can be caused by conflicts between headers. If an object in the class is only used by pointer or by reference, avoid `include`ing it and just add a forward declaration before the class. 

3.2. __DO NOT__ rely on a header that is included indirectly by another header you include.


## 4. Classes & Structs

4.1. __DO__ use PascalCase for class names, e.g. `AHOATCharacter`.

4.2. __DO__ add type prefixes to class names to distinguish them from variable names. For example, `FSkin` is a type name, and `Skin` is an instance of a `FSkin`. UnrealHeaderTool requires the correct prefixes in most cases, so it's important to provide them.

* Template classes are prefixed by `T`. 
* Classes that inherit from `UObject` are prefixed by `U`. 
* Classes that inherit from `AActor` are prefixed by `A`. 
* Classes that inherit from `SWidget `are prefixed by `S`. 
* Classes that are abstract interfaces are prefixed by `I`. 
* Enums are prefixed by `E`. 
* Most other classes are prefixed by `F`, though some subsystems use other letters. 

4.3. __DO__ prefix boolean variables by `b` (e.g. `bPendingDestruction` or `bHasFadedIn`).

4.4. __DO__ uppercase the first letter of acronyms, only, e.g. `FHOATXmlStreamReader`, not `FHOATXMLStreamReader`.

4.5. __DO__ add a virtual destructor to classes with virtual member functions.

4.6. __DO__ mark classes that are not meant to be derived from as `final`. This should be the default for non-interface classes. Care has to be taken when removing the `final` keyword from a class when inheritance is required. Classes that are already derived don't need to be marked as `final` by default: In the most common case there is no reason to prevent further inheritance.

4.7. __DO__ use a non-virtual destructor in `final` classes unless they are already derived.

4.8. __DO__ use `struct`s for data containers, only. They shouldn't contain any logic beyond simple validation or need any destructors.

4.9. __DO__ use `TObjectPtr` instead of raw pointers for UObject pointer properties and container classes found in `UCLASS` and `USTRUCT` types. See [Migration Guide](https://docs.unrealengine.com/5.0/en-US/unreal-engine-5-migration-guide/).

## 5. Constructors

5.1. __DO__ mark each constructor that takes exactly one argument as `explicit`, unless it's a copy constructor or the whole point of the constructor is to allow implicit casting. This minimizes wrong use of the constructor.

5.2 __DO NOT__ define empty constructors/destructors that do nothing (with an empty body or with `= default`) unless required.

5.3 __CONSIDER__ defining a deleted copy constructor/copy assignment operator for classes and structs that should not be copied (e.g. ones that contain a lot of data).

## 6. Functions

6.1. __DO__ use PascalCase for functions names.

6.2. __DO NOT__ add a space between function name and parameter list:

      // Right:
      void Tick(float DeltaSeconds);

      // Wrong:
      void Tick (float DeltaSeconds);

6.3. __DO__ pass each object parameter that is not a basic type (`int`, `float`, `bool`, `enum`, or pointers) by reference-to-const. This is faster, because it is not required to do a copy of the object. Also, this is required for exposing the property in blueprints:

      bool GetObjectsAtWorldPosition(const FVector& WorldPositionXY, TArray<FHitResult>& OutHitResults);

6.4. __DO__ prefix function parameter names with `Out` if they are passed by reference and the function is expected to write to that value. This makes it obvious that the value passed in this argument will be replaced by the function. If an `Out` parameter is also a boolean, put the `b` before the `Out` prefix, e.g. `bOutResult`. 

6.5. __DO__  flag methods as `const` if they do not modify the object.

6.6. __CONSIDER__ writing functions with six parameters or less. For passing more arguments, pass instead a `struct` containing all the parameters (which sensible defaulted values), and/or refactor your function.

6.7. __CONSIDER__ using enum values instead of boolean function parameters.

      // Right:
      ShowMessageBox(TEXT("Nice Title"), TEXT("Nice Text"), MessageBox::MESSAGEBOX_BUTTONS_OK);

      // Wrong: Meaning of third parameter is not immediately obvious.
      ShowMessageBox(TEXT("Nice Title"), TEXT("Nice Text"), false);

6.8. __AVOID__ providing function implementations in header files. Use inline functions judiciously, as they force rebuilds even in files which don't use them. Inlining should only be used for trivial accessors and when profiling shows there is a benefit to doing so. Be even more conservative in the use of FORCEINLINE. All code and local variables will be expanded out into the calling function and will cause the same build time problems caused by large functions. Don't use inlining or templates for functions which are likely to change over a hot reload.

6.9. __AVOID__ using BlueprintPure for functions that do non-trivial amount of work. Keep in mind that Pure functions don't cache their results, so it's easy to do a lot of duplicated work by misusing them in Blueprints.

## 7. Variables

7.1. __DO__ use Pascal Case (ClassVariable) for class variable names. Use Camel Case (myVariable) for local variables names. This will make it easy to differentiate local vs class variables when glancing at code and when doing code-reviews.

7.2. __AVOID__ short or meaningless names (e.g. `A`, `Rbarr`, `Nughdeget`). Single character variable names are only okay for counters and temporaries, where the purpose of the variable is obvious. An exception to this rule are mathematical functions where the meaning of the parameters is "universally" known, such as `Lerp(float A, float B, float T)` or `Sin(float X)`.

7.3. __AVOID__ using negative names for boolean variables.

    // Right:
    if (bVisible)

    // Wrong: Double negation is hard to read.
    if (!bInvisible)

7.4. __DO__ declare each variable on a separate line so that a comment on the meaning of the variable can be provided.  Also, the JavaDoc style requires it.

7.5. __DO__ use portable aliases for basic C++ types:

* `bool` for boolean values (_never_ assume the size of `bool`). `BOOL` will not compile. 
* `TCHAR` for a character (_never_ assume the size of `TCHAR`). 
* `uint8` for unsigned bytes (1 byte). 
* `int8` for signed bytes (1 byte). 
* `uint16` for unsigned "shorts" (2 bytes). 
* `int1` for signed "shorts" (2 bytes). 
* `uint32` for unsigned ints (4 bytes). 
* `int32` for signed ints (4 bytes). 
* `uint64` for unsigned "quad words" (8 bytes). 
* `int64` for signed "quad words" (8 bytes). 
* `float` for single precision floating point (4 bytes). 
* `double` for double precision floating point (8 bytes). 
* `PTRINT` for an integer that may hold a pointer (_never_ assume the size of `PTRINT`). 

7.6. __DO__ put a single space between the `*` or `&` and the variable name for pointers or references, and don't put a space between the type and `*` or `&`. For us, the fact that we are declaring a pointer or reference variable here much more belongs to the type of the variable than to its name:

      AController* Instigator
      const FDamageEvent& DamageEvent

7.7. __DO__ test whether a pointer is valid before dereferencing it. `nullptr` should be used instead of the C-style `NULL` macro in all cases. If the pointer points to any `UOBJECT`, use `IsValid` to ensures the pointer is not null and the referenced object is not marked for destruction. 


## 8. Enums & Constants

8.1. __CONSIDER__ using `enum class` or `static constexpr` over `static const` variables over `#define` when defining constants.

8.2. __DO__ use ALL_CAPS with underscores between words for constant names.

## 9. Indentation & Whitespaces

9.1. __DO NOT__ put multiple statements on one line.


## 10. Line Breaks

10.1 __DO__ separate function bodies with at least one newline.
     // Right
     int Foo()
     {
         // ...
     }
     
     void Bar() {}
     
     // Wrong
     int Foo()
     {
         // ...
     }
     void Bar() {}
    
Exception: when you group together multiple related class methods you should omit the newline:
    
    // Also right
    class C
    {
        ...
        
        int GetHeight() const { return Height; }
        int GetWidth() const { return Width; }
    };

## 11. Braces

11.1. __DO__ use curly braces, even if the body of a conditional statement contains just one line:

      // Right:
      if (bVisible)
      {
          Hide();
      }

      // Wrong: Can lead to subtle bugs in the future, if the body is extended to span multiple statements.
      if (bVisible)
          Hide();


## 12. Parentheses

12.1. __DO__ use parentheses to group expressions:

      // Right:
      if ((a && b) || c)

      // Wrong: Operator precedence is not immediately clear.
      if (a && b || c)

12.3. __DO NOT__ use spaces after parentheses:

      // Right:
      if ((a && b) || c)

      // Wrong:
      if ( ( a && b ) || c )


## 13. Control Flow

13.1. __DO__ add a `break` (or `return`) statement at the end of every `case`, or use `[[fallthrough]]`

      switch (MyEnumValue)
      {
        case Value1:
            DoSomething();
            break;

        case Value2:
        case Value3:
            DoSomethingElse();
            [[fallthrough]];

        default:
            DefaultHandling();
            break;
      }
      
13.1.1. __DO__ wrap switch cases in parentheses whenever you declare variables inside them

      switch (MyVar)
      {
        case 42:
        {
            int X = 0;
            Foo(X);
            break;
        }
        
        default:
            break;
      }
      
13.1.2. __DO__ add a `default` case at the end of every switch. If empty, either do `default: /* newline */ break;` or `default:;`.

13.2. __DO NOT__ put `else` after jump statements:

      // Right:
      if (ThisOrThat)
      {
          return;
      }

      SomethingElse();


      // Wrong: Causes unnecessary indentation of the whole else block.
      if (ThisOrThat)
      {
          return;
      }
      else
      {
          SomethingElse();
      }


13.3. __DO NOT__ mix const and non-const iterators.

      // Right:
      for (Container::const_iterator it = c.cbegin(); it != c.cend(); ++it)

      // Wrong: Crashes on some compilers.
      for (Container::const_iterator it = c.begin(); it != c.end(); ++it)

## 14. General Style

14.1. __DO__ use the short form `if (Pointer)` over `if (Pointer != nullptr)`.

14.2. __DO__ use the short form `if (bBool)` and `if (!bBool)` over `if (bBool == true)` or `if (bBool == false)`.

14.3. __DO NOT__ use Yoda-style expressions:

       // Good
       if (MyHealth == 0) { ... }
       if (MyHealth == SomeThing()) { ... }
       
       // Bad
       if (0 == MyHealth) { ... }
       if (SomeThing() == MyHealth) { ... }
       
14.4. __AVOID__ too many indentation levels. If your function has a lot of indentation, you have 2 options:

* if the high nesting is just due to error checking, consider replacing nesting with early returns/continues:

        // Bad
        if (Actor) 
        {
           if (IsAvailable(Actor))
           {
               for (AActor* OtherActor : Others)
               {
                   if (IsAvailable(OtherActor))
                   {
                       // ...
                   }
               }
           }
        }
        
        // Good
        if (!Actor)
        {
            return;
        }
        
        if (!IsAvailable(Actor))
        {
            return;
        }
        
        for (AActor* OtherActor : Others)
        {
            if (!IsAvailable(OtherActor))
            {
                continue;
            }
        }
        
* if the nesting instead derives from complex subroutines of your function, consider splitting it into multiple functions.

## 15. Language Features

15.1. __AVOID__ using the `auto` keyword except when required by the language (e.g. assigning lambdas) or when assigning long iterator types. If in doubt, for example if using `auto` could make the code less readable, do not use `auto`.

      // Bad
      auto* HealthComponent = FindComponentByClass<USOCHealthComponent>();
      
      // Good
      USOCHealthComponent* HealthComponent = FindComponentByClass<USOCHealthComponent>();
      
      TMap<UAVeryVeryLongTypeHere, FAnotherVeryLongTypeThere> Map;
      for (auto it = Map.begin(); it != Map.end(); it++) { ... }

15.2. __DO__ use `auto*` for auto pointers, to be consistent with references, and to add additional guidance for the reader. Remember that `const auto*` and `const auto` mean different things when the type resolves to a pointer (and you usually want `const auto*`).

15.3. __DO__ use proprietary types, such as `TArray` or `TMap` where possible. This avoids unnecessary and repeated type conversion while interacting with the Unreal Engine APIs.

15.4. __DO__ use the `TEXT()` macro around string literals. Without it, code which constructs `FString`s from literals will cause an undesirable string conversion process. 

15.5. __CONSIDER__ using file-local helper functions where it makes sense. In that case, declare them inside an anonymous namespace to indicate that they're private to that cpp file.

15.6. __DO__ use `[[maybe_unused]]` for functions and variables that are only used in conditionally-compiled code (e.g. some that are only used inside `check` macros).

## 16. Casting

16.1. __DO__ use C++-style casts (`static_cast`, `const_cast`, `reinterpret_cast`) over C-style casts.

16.2. __DO NOT__ use `dynamic_cast` ever. Use `Cast` instead.

16.3. __CONSIDER__ using `CastChecked` over `Cast` when casting to a type that is guaranteed to be correct (i.e. if you don't need to check whether the object is really of that type AND the object is guaranteed not to be null).


## 17. Events & Delegates

17.1. __DO__ define two functions when exposing an event to a subclass. The first function should be virtual and its name should begin with `Notify`. The second function should be a `BlueprintImplementableEvent UFUNCTION` and its name should begin with `Receive`. The default implementation of the virtual function should be to call the `BlueprintImplementableEvent` function (see `AActor::NotifyActorBeginOverlap` and `AActor::ReceiveActorBeginOverlap` for example).

17.2. __DO__ call the virtual function before broadcasting the event, if both are defined (see `UPrimitiveComponent::BeginComponentOverlap` for example).

Example:
    // Multicast Delegate signature and optional typedef for the delegate - must include comment for parameters in delegate signature
    DECLARE_MULTICAST_DELEGATE_TwoParams(FAuthOnLoginCompleteSignature, bool /*bWasSuccessful*/, const FString& /*ErrorMessage*/);
    typedef FAuthOnLoginCompleteSignature::FDelegate FAuthOnLoginCompleteDelegate;
    
    // Delegate and event variables should be prepended with On
    FAuthLoginCompleteDelegate OnLoginComplete;
    
    // Handlers for delegates should be prepended with Handle
    void FAuthLoginCompleteDelegate HandleLoginComplete(bool bWasSuccessful, const FString& ErrorMessage);
    
Example:

    DECLARE_DYNAMIC_MULTICAST_DELEGATE_ThreeParams(FHoatActorGraphConnectivityChangedSignature, AActor*, Source, AActor*, Target, float, Distance);
    
    /** Event when the connectivity of an observed source vertex has changed. */
    virtual void NotifyOnConnectivityChanged(AActor* Source, AActor* Target, float Distance);

    /** Event when the connectivity of an observed source vertex has changed. */
    UFUNCTION(BlueprintImplementableEvent, Category = Graph, meta = (DisplayName = "OnConnectivityChanged"))
    void ReceiveOnConnectivityChanged(AActor* Source, AActor* Target, float Distance);

    /** Event when the connectivity of an observed source vertex has changed. */
    UPROPERTY(BlueprintAssignable)
    FHoatActorGraphConnectivityChangedSignature OnConnectivityChanged;

    void ASOCActorGraph::NotifyOnConnectivityChanged(AActor* Source, AActor* Target, float Distance)
    {
        ReceiveOnConnectivityChanged(Source, Target, Distance);
        OnConnectivityChanged.Broadcast(Source, Target, Distance);

        SOC_LOG(hoat, Log, TEXT("%s changed the connectivity of vertex %s: Distance to target %s changed to %f."), *GetName(), *Source->GetName(), *Target->GetName(), Distance);
    }

## 18. Comments

18.1. __DO__ add a space after `//`.

18.2. __DO__ place the comment on a separate line, not at the end of a line of code.

18.3. __DO__ write API documentation with [Javadoc comments](https://docs.unrealengine.com/latest/INT/Programming/Development/CodingStandard/index.html#exampleformatting).


## 19. Additional Naming Conventions

19.1 (for SOCS2): prefix all game-specific C++ public types with `SOC`.
