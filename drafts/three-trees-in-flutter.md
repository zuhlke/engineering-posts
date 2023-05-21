---
title: Understanding the three trees in Flutter - Widget tree, Element tree and RenderObject tree
domain: software-engineering-corner.hashnode.dev
tags: flutter, flutter-widgets
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1684687187737/Tgk7dXIfu.png?auto=format
publishAs: gabduss
hideFromHashnodeCommunity: false
---

Flutter, Google's powerful UI framework, operates on a unique architecture that efficiently renders user interfaces. At the core of this architecture are three interconnected trees: the widget tree, element tree, and render object tree.

In this blog post, we will dive into the inner workings of Flutter's three trees and understand their relationships. These trees play a vital role in constructing and rendering UI components, enabling developers to create visually stunning and performant applications.

# The widget tree

In Flutter, a widget tree refers to the hierarchical structure of widgets that are used to compose the user interface of an application. The widget tree represents the visual components of the app and their relationships with each other.

At its core, Flutter follows a declarative approach, where the user interface is described using a tree of immutable widgets. Each widget represents a specific UI element, such as a button, text, image, or even a complex layout. Widgets can be combined and nested within each other to create more complex user interfaces.

Let's make a small example of a widget tree. The follwing example draws a red container with the text "Hello" in it.

```dart
class MyWidget extends StatelessWidget {
  const MyWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(
      color: Colors.red,
      padding: const EdgeInsets.all(8.0),
      child: const Text("Hello"),
    );
  }
}
```

The widget tree of this example is very simple as the following image visualizes.

<img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1684687330012/vuSdPUtnf.png?auto=format" width=60% height=60%>

The first tree shows the defined widget tree, but Flutter created a refined widget tree, that is illustrated on the right side. But why is Flutter adding tree additonal widgets into the widget tree? To address this question, it's crucial to understand Flutter's two categories of widgets for defining UI elements: StatefulWidgets/StatelessWidgets and RenderObjectWidgets. RenderObjectWidgets are responsible for rendering elements on the screen, while StatefulWidgets/StatelessWidgets simplify their usage. As a developer, you typically combine and configure existing widgets within a StatefulWidget/StatelessWidget, rarely creating RenderObjectWidgets directly. Many widgets provided by Flutter are themselves StatefulWidget/StatelessWidget, but ultimately, each of these widgets contributes at least one RenderObjectWidget to the tree, as only RenderObjectWidgets have the knowledge to render elements on the canvas.

The following illustration shows all types of widgets.
<img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1684687367533/AdMsKBnCM.png?auto=format" width=80% height=80%>

The previous example shows a popular example: the container widget. You can set a color and a padding on the widget. Flutter will transform it to a ColoredBox and a Padding RenderObjectWidget. The ColoredBox will be responsible to draw the color, the Padding will be responsible to draw the padding. It is also possible to use the RenderObjectWidget directly, like the following code shows.

```dart
 class MyWidget extends StatelessWidget {
  const MyWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return ColoredBox(
      color: Colors.red,
      child: Padding(
        padding: const EdgeInsets.all(8.0),
        child: RichText(text: const TextSpan(text: "Hello")),
      ),
    );
  }
}
```

In Flutter, a widget gets rebuilt when its internal state changes or when its parent widget requests a rebuild. Here are some common scenarios that trigger widget rebuilds:

- Initial build: When a widget is first inserted into the widget tree-
- State changes: If a widget's internal state changes, because of user interactions or an network respons.
- InheritedWidget changes: Widgets can receive data changes from ancestor widgets.
- Layout changes: If the layout constraints of a widget changes, e.g. the oriantation of a device changes.
- Widget tree updates: If a widget's parent requests a rebuild.
- Animation updates: If a widget includes an animation, it typically rebuilds on every frame.

As you can see, widgets get rebuild a lot. That's why they need to be extremly lightweight.

# The three trees in Flutter

To keep the widgets lightweight Flutter has two additonal trees: the element tree and the RenderObject tree. Those three trees are connected with each other.

Let's check the first example again and take a look of how their Element and RenderObject tree look like.

<img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1684687397242/th4DOl27p.png?auto=format" width=70% height=70%>

As the widget tree is immutalbe and gets rebuild a lot. The Element and RenderObject trees are mutable and dont get recreated that frequently. The Element tree is responsible for the lifecyle and connects the Widget tree with the RenderObject tree. As you can see, every widget generates an Element, but not every Element has a RenderObject. Only RenderObjectElement generate RenderObjects, all other elements send their configuration down to the next RenderObjectElement. The following illustration shows the two different Element types.
 
 <img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1684687427351/X78ENpgNl.png?auto=format" width=50% height=50%>

The previous illustration includes another interesting detail. The Element implements the BuildContext. That means that the BuildContext included in every build method is nothing other than an Element with restricted access. That's why the BuildContext has all the knowlege of an Element, like the lifecycle state or their position in the tree. As the following code shows, the BuildContext can be casted to a Element.

```dart { .customCodeStyle }
Widget build(BuildContext context) {
    var element = context as Element;

    return GestureDetector(
      child: isRed ? widgetRed : widgetBlue,
      onTap: () {
        isRed = !isRed;
        element.markNeedsBuild();
      },
    );
  }
```

The cast to an Element lets you access all the properties of an Element. This can be very interesting for educational reasons, but its not recommended in production. The is a reason why the don't allow direct access and add the properties to the BuildContext. The following tables sums up the most imporant attributes of the three objects.

|             | Widget                       | Element                                                                                                      | RenderObject                |     |
| ----------- | ---------------------------- | ------------------------------------------------------------------------------------------------------------ | --------------------------- | --- |
| Mutable     | false                        | true                                                                                                         | true                        |     |
| Lightweight | true                         | false                                                                                                        | false                       |     |
| Jobs        | - Holds widget configuration | - Responsible for lifecycle <br> - Holds state of StatefulWidgets <br> - Connects Widgets with RenderObjects | - Responsible for rendering |     |

# Conclusion

In conclusion, the widget tree, element tree, and render object tree are three interconnected trees in Flutter that are vital for constructing and rendering UI components efficently. The widget tree represents the hierarchical structure of immutable widgets that describe the UI elements. Render object widgets handle the rendering, while elements are responsible for the lifecycle and connect the widget tree with the render object tree.
