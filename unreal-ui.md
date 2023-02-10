# Unreal UI

## Table of Contents

- [Widgets](#widgets)

---

# Fonts

[Google Fonts](https://fonts.google.com/)

# Styling

[Unreal Documentation](https://docs.unrealengine.com/5.1/en-US/umg-styling-in-unreal-engine/)

- Nine-slice image is an image split into 9 segments, in which the outside 8 segments act as a border

# Widgets

- **Tip:** `RemoveFromParent` on `self` from the Widget's Event Graph will remove the widget instance from the scene entirely.

- Widget Editer has two parts:
  
  - **Designer** - visual canvas to design widget
  
  - **Event Graph** - Blueprint event graph

## UWidget vs UUserWidget

UWidget is a single widget which can contain single Slate widget or Slate composition of widgets. UWidget can’t contain composition of UWidgets, they can parent UWidgets and layout them via Slate layout widgets, but it can not compose them ferther then that.

UUserWidget main different is that it can contain free composition of UWidgets.

## Dependency

- Unreal C++ requires adding the **`UMG`** Module Dependency in `Source/<ProjectName>/ProjectName.Build.cs` file:
  
``` c#
PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "GameplayTasks", "UMG" });
```

## Widget Components

- Widgets are made of Widget Components, like text, boxes, images, etc

- **Is Variable** option in the Designer window exposes a widget component in the event graph

- **Bind** 
    - Links component properties to widget variables

## Anchor
  
  - Used to maintain component position.
  
  - **Tip:** Set the anchor point for widget components to their nearest corner or screen center

  - Anchors can be a point or region with min and max values. Ths changes the point *Position* values to *Offset* values in the *Details*. This will make the scale/size of the box in the offset relative to the offset distance to the canvas corners.

### Slots

Widgets can have a number of slots. A Canvas is a container with infinite slots. The Button is a container with one slot.

# UWidget

## Buttons

### Draw As 

  - **Box**: Scales image as a 9 size, maintaining margins

  - **Border**: Draws the image margins according to 9 slice format. Only margins are rendered 

  - **Image**: No margins considered. Entire image is scaled proportionally 

## Spacer

# UUserWidget

## Canvas Panel
  
  - The base canvas for a multi-part widget
  
  - Add it by dragging to the widget hierarchy

## Horizontal/Vertical Box

There are **Vertical** and **Horizontal** box Panels. These operate similar to iOS UICollectionViews.

These lay out thier child components in slot order.

## Size Box

Container that enforces size limits for its child slot.

Can also use as empty spacers, and can automatically *Fill* inside a Horizontal/Vert Box to dynamically and evenly space.

## Scale Box

Container that manages and maintains child proportions when scaled to different sizes

## UScrollBox

- Use for scrollable content. Loads all entries at once.
- Does not natively support selection.

## UListView

- Scrollable box, but loads entries dynamically, as opposed to `UScrollBox` 
- Better for large lists, so content can be reallocated.
- Supports selection

## Overlay

Acts as a kind of sub canvas, but enforces overlap order by the slot order, where bottom-most slot is front-most visually.

*Tip:* User Overlay and Image to give an image a background. The Image can be set to just a solid color

# WidgetSwitcher

Shows only one slot at a time via the **Active Widget Index**

# UUserWidget

- [Documentation](https://docs.unrealengine.com/5.1/en-US/API/Runtime/UMG/Blueprint/UUserWidget/)

- `#include "Blueprint/UserWidget.h`

- Widget Blueprint Base class. 

- Must use `CreateWidget<>()` constructor.

## BindWidget

`UPROPERTY(meta = (BindWidget))`

**meta** property "BindWidget" exposes a Widget superclass property to its Blueprint subclass, and links them via their **Name**

- [Blog Post Reference](https://benui.ca/unreal/ui-bindwidget/)

- `BindWidgetOptional` allows for compilation without bound widget being set. 

- Some warnings:
  1. This can create a dependency cycle
  2. This pattern can create invisible failure points for your UI artist. (i.e. renaming a binded property in the widget will create a build failure and/or break the data connection)

### Creating a Widget

Widget Constructor:
```cpp
template<typename WidgetT, typename OwnerT>
WidgetT * CreateWidget
(
  OwnerT * OwningObject,
  TSubclassOf< UUserWidget > UserWidgetClass,
  FName WidgetName
) 
```

- Can instantiate widget in C++ (in PlayerController) adding a `UPROPERTY TSubclassOf<UUserWidget>` and setting the BP_Widget we created to that property. Then in the PlayerController.cpp, we create and manage it.  

- PlayerControllers are good for managing Player UI

- Widgets are instantiated with `CreateWidget()`

- Once instantiated, made viewable with `AddToViewport()`

### TSharedRef<SWidget> TakeWidget()

Returns the underlying slate widget or constructs it if it doesn't exist.  If you're looking to replace what slate widget gets constructed look for `RebuildWidget`.

### Adding to Viewport

`AddToViewport(int32 ZOrder)` Method adds widget to viewport

Example:
```cpp
  UUserWidget* MenuWidget = CreateWidget<UUserWidget, UPuzzlePlatformGameInstance>(this, MenuWidgetClass, TEXT("MenuWidget"));
```

*NOTE: to interact with Widget, must have `bIsFocusable` set to true*

#### PlayerController

  - [PlayerController Documentation](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/GameFramework/APlayerController/)

  - `SetInputMode(FInputModeDataBase&)` - sets up input mode on Player Controller.
    - Requires `FInputModeDataBase` struct
  
  - `bShowMouseCursor`

  - `WidgetT* CreateWidget(OwnerT* OwningObject, TSubclassOf<UUserWidget> UserWidgetClass = WidgetT::StaticClass(), FName WidgetName = NAME_None)`
    - Can instantiate Blueprint widget in C++ adding a `UPROPERTY TSubclassOf<UUserWidget>` and setting the BP_Widget we created to that property. Then call `CreateWidget(this, GameOverWidgetClass);`
    - PlayerControllers are one of the few classes that can act as Widget Owners 

#### FInputModeDataBase

- `SetWidgetToFocus(SWidget)` sets the focus of the input mode 

- `SetLockMouseToViewportBehavior(EMousLockMode)`

## Widget Methods

- `Initialize()` - override to setup up properties in a widget class. (Don't forget to call Super!)

- `OnLevelRemovedFromWorld()` - called when top level widget is in the viewport and the world is potentially coming to an end. Perform cleanup.

- `AddToViewport()` - See [Adding To Viewport](#adding-to-viewport)

- `RemoveFromViewport()` 