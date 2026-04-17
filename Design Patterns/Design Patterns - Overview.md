---
tags: 
- design
- pattern
- overview
---

# Design Patterns - Detailed Notes

- **Design patterns** are reusable solutions to commonly occurring problems in software design. They are not code you copy-paste — they are templates for how to solve a problem that can be adapted to many situations.
- The concept was popularized by the **Gang of Four (GoF)** — Erich Gamma, Richard Helm, Ralph Johnson, and John Vlissides — in their 1994 book *Design Patterns: Elements of Reusable Object-Oriented Software*.
- Patterns are grouped into three categories: **Creational**, **Structural**, and **Behavioral**.

```ad-important
A pattern is not always the right answer. Applying patterns where they are not needed adds unnecessary complexity. Use a pattern when the problem it solves actually exists in your code.
```

---

## Creational Patterns

- Creational patterns deal with **object creation mechanisms** — they abstract the instantiation process so that the system is independent of how its objects are created, composed, and represented.

### Singleton

- Ensures a class has **only one instance** and provides a global point of access to it.
- The class itself is responsible for keeping track of its sole instance.
- **When to use:** logging, configuration, thread pools, caches — any resource where having multiple instances would cause conflicts or waste.

```cpp
class Singleton {
private:
    static Singleton* instance;
    Singleton() {}  // private constructor
public:
    static Singleton* getInstance() {
        if (!instance)
            instance = new Singleton();
        return instance;
    }
};
```

```ad-warning
Singleton is often overused. It introduces global state, makes testing harder (hard to mock), and hides dependencies. Prefer dependency injection when possible.
```

### Factory Method

- Defines an interface for creating an object, but lets **subclasses decide** which class to instantiate.
- The creator class has a method that returns a product, and subclasses override this method to produce different products.
- **When to use:** when a class cannot anticipate the type of objects it needs to create, or when you want subclasses to specify the objects they create.

```cpp
class Transport {
public:
    virtual void deliver() = 0;
};

class Truck : public Transport {
public:
    void deliver() override { /* road delivery */ }
};

class Ship : public Transport {
public:
    void deliver() override { /* sea delivery */ }
};

class Logistics {
public:
    virtual Transport* createTransport() = 0;

    void planDelivery() {
        Transport* t = createTransport();
        t->deliver();
    }
};

class RoadLogistics : public Logistics {
public:
    Transport* createTransport() override {
        return new Truck();
    }
};
```

### Abstract Factory

- Provides an interface for creating **families of related objects** without specifying their concrete classes.
- Think of it as a factory of factories.
- **When to use:** when your system needs to work with multiple families of related products (e.g., a UI toolkit that supports Windows and macOS widgets — buttons, checkboxes, menus — all need to match the same family).

```cpp
// Abstract products
class Button { public: virtual void render() = 0; };
class Checkbox { public: virtual void render() = 0; };

// Concrete families
class WinButton : public Button { void render() override { /* Windows look */ } };
class WinCheckbox : public Checkbox { void render() override { /* Windows look */ } };

class MacButton : public Button { void render() override { /* macOS look */ } };
class MacCheckbox : public Checkbox { void render() override { /* macOS look */ } };

// Abstract factory
class GUIFactory {
public:
    virtual Button* createButton() = 0;
    virtual Checkbox* createCheckbox() = 0;
};

class WinFactory : public GUIFactory {
public:
    Button* createButton() override { return new WinButton(); }
    Checkbox* createCheckbox() override { return new WinCheckbox(); }
};
```

### Builder

- Separates the **construction** of a complex object from its **representation**, so the same construction process can create different representations.
- Useful when an object requires many steps to create, or has many optional parameters.
- **When to use:** constructing objects that would require a telescoping constructor (constructor with many parameters), or when construction involves several steps.

```cpp
class Pizza {
public:
    std::string dough, sauce, topping;
};

class PizzaBuilder {
protected:
    Pizza* pizza;
public:
    PizzaBuilder() { pizza = new Pizza(); }
    Pizza* getPizza() { return pizza; }

    virtual void buildDough() = 0;
    virtual void buildSauce() = 0;
    virtual void buildTopping() = 0;
};

class MargheritaBuilder : public PizzaBuilder {
public:
    void buildDough() override { pizza->dough = "thin crust"; }
    void buildSauce() override { pizza->sauce = "tomato"; }
    void buildTopping() override { pizza->topping = "mozzarella + basil"; }
};

class Cook {
    PizzaBuilder* builder;
public:
    void setBuilder(PizzaBuilder* b) { builder = b; }
    Pizza* makePizza() {
        builder->buildDough();
        builder->buildSauce();
        builder->buildTopping();
        return builder->getPizza();
    }
};
```

### Prototype

- Creates new objects by **cloning an existing object** (the prototype) instead of building from scratch.
- **When to use:** when object creation is expensive (e.g., requires database calls or complex computation), and a similar object already exists. Also useful when you need copies that are independent of their concrete classes.

```cpp
class Shape {
public:
    virtual Shape* clone() = 0;
    virtual void draw() = 0;
};

class Circle : public Shape {
    int radius;
public:
    Circle(int r) : radius(r) {}
    Shape* clone() override { return new Circle(*this); }
    void draw() override { /* draw circle */ }
};
```

---

## Structural Patterns

- Structural patterns deal with **object composition** — how classes and objects are combined to form larger structures while keeping them flexible and efficient.

### Adapter

- Converts the interface of a class into **another interface that clients expect**. Lets classes work together that otherwise couldn't because of incompatible interfaces.
- Think of it like a power plug adapter: the wall socket (existing interface) does not change, but the adapter lets your device (client) plug in.
- **When to use:** when you want to use an existing class but its interface does not match what you need, or when integrating third-party/legacy code.

```cpp
// Existing interface your code expects
class MediaPlayer {
public:
    virtual void play(std::string filename) = 0;
};

// Incompatible third-party class
class VLCLibrary {
public:
    void playVLC(std::string filename) { /* plays .vlc files */ }
};

// Adapter bridges the gap
class VLCAdapter : public MediaPlayer {
    VLCLibrary* vlc;
public:
    VLCAdapter() { vlc = new VLCLibrary(); }
    void play(std::string filename) override {
        vlc->playVLC(filename);
    }
};
```

### Decorator

- Attaches **additional responsibilities** to an object dynamically. Provides a flexible alternative to subclassing for extending functionality.
- Decorators wrap the original object and add behavior before or after delegating to it.
- **When to use:** when you need to add behavior to individual objects without affecting other objects of the same class, or when extension by subclassing is impractical (combinatorial explosion of subclasses).

```cpp
class Coffee {
public:
    virtual double cost() = 0;
    virtual std::string description() = 0;
};

class SimpleCoffee : public Coffee {
public:
    double cost() override { return 1.0; }
    std::string description() override { return "Simple coffee"; }
};

class MilkDecorator : public Coffee {
    Coffee* wrapped;
public:
    MilkDecorator(Coffee* c) : wrapped(c) {}
    double cost() override { return wrapped->cost() + 0.5; }
    std::string description() override {
        return wrapped->description() + ", milk";
    }
};

// Usage: Coffee* c = new MilkDecorator(new SimpleCoffee());
// c->cost() == 1.5, c->description() == "Simple coffee, milk"
```

### Facade

- Provides a **simplified interface** to a complex subsystem. It does not hide the subsystem — it just provides a convenient entry point.
- **When to use:** when a subsystem has many classes and the client only needs a simple high-level interface, or when you want to layer your system.

```cpp
// Complex subsystem classes
class CPU { public: void start() { } };
class Memory { public: void load() { } };
class HardDrive { public: void read() { } };

// Facade
class Computer {
    CPU cpu;
    Memory memory;
    HardDrive hd;
public:
    void startComputer() {
        cpu.start();
        memory.load();
        hd.read();
        // client doesn't need to know the boot sequence
    }
};
```

### Proxy

- Provides a **surrogate or placeholder** for another object to control access to it.
- Types of proxies: **virtual** (lazy initialization), **protection** (access control), **remote** (represents an object in a different address space), **logging/caching** proxy.
- **When to use:** lazy loading of expensive objects, access control, logging, caching, or working with remote services.

```cpp
class Image {
public:
    virtual void display() = 0;
};

class RealImage : public Image {
    std::string filename;
public:
    RealImage(std::string f) : filename(f) {
        // expensive: loads image from disk
    }
    void display() override { /* render image */ }
};

class ProxyImage : public Image {
    RealImage* realImage = nullptr;
    std::string filename;
public:
    ProxyImage(std::string f) : filename(f) {}
    void display() override {
        if (!realImage)
            realImage = new RealImage(filename); // lazy load
        realImage->display();
    }
};
```

### Composite

- Composes objects into **tree structures** to represent part-whole hierarchies. Lets clients treat individual objects and compositions of objects uniformly.
- **When to use:** when you have a tree-like structure (file system, UI components, organization charts) and want to treat leaves and containers the same way.

```cpp
class FileComponent {
public:
    virtual void showDetails() = 0;
};

class File : public FileComponent {
    std::string name;
public:
    File(std::string n) : name(n) {}
    void showDetails() override { /* print file name */ }
};

class Folder : public FileComponent {
    std::string name;
    std::vector<FileComponent*> children;
public:
    Folder(std::string n) : name(n) {}
    void add(FileComponent* c) { children.push_back(c); }
    void showDetails() override {
        for (auto* child : children)
            child->showDetails(); // works on files AND folders
    }
};
```

### Bridge

- Decouples an **abstraction from its implementation** so the two can vary independently.
- Think of it as: shape (abstraction) and color (implementation) can change independently without a `RedCircle`, `BlueCircle`, `RedSquare`, `BlueSquare` explosion.
- **When to use:** when both the abstraction and the implementation may change, and you want to avoid a combinatorial explosion of subclasses.

```cpp
// Implementation hierarchy
class Color {
public:
    virtual std::string fill() = 0;
};
class Red : public Color {
public:
    std::string fill() override { return "red"; }
};
class Blue : public Color {
public:
    std::string fill() override { return "blue"; }
};

// Abstraction hierarchy
class Shape {
protected:
    Color* color;
public:
    Shape(Color* c) : color(c) {}
    virtual void draw() = 0;
};
class Circle : public Shape {
public:
    Circle(Color* c) : Shape(c) {}
    void draw() override { /* draw circle filled with color->fill() */ }
};
```

### Flyweight

- Uses **sharing** to support large numbers of fine-grained objects efficiently.
- Separates intrinsic state (shared, immutable) from extrinsic state (unique, passed in by client).
- **When to use:** when your application creates a huge number of similar objects that eat up memory (e.g., characters in a text editor, trees in a game forest).

```cpp
// Each unique tree type is shared (flyweight)
class TreeType {
    std::string name, color, texture;
public:
    TreeType(std::string n, std::string c, std::string t)
        : name(n), color(c), texture(t) {}
    void draw(int x, int y) { /* draw at position using shared data */ }
};

// Extrinsic state: position is unique per tree
class Tree {
    int x, y;
    TreeType* type; // shared
public:
    Tree(int x, int y, TreeType* t) : x(x), y(y), type(t) {}
    void draw() { type->draw(x, y); }
};
```

---

## Behavioral Patterns

- Behavioral patterns deal with **communication between objects** — how objects interact and distribute responsibility.

### Observer

- Defines a **one-to-many dependency** between objects so that when one object (the subject) changes state, all its dependents (observers) are notified and updated automatically.
- **When to use:** event systems, pub/sub, UI updates when model data changes (MVC), any scenario where multiple objects need to react to state changes.

```cpp
class Observer {
public:
    virtual void update(float temperature) = 0;
};

class WeatherStation {
    std::vector<Observer*> observers;
    float temperature;
public:
    void addObserver(Observer* o) { observers.push_back(o); }
    void setTemperature(float t) {
        temperature = t;
        for (auto* o : observers)
            o->update(temperature); // notify all
    }
};

class PhoneDisplay : public Observer {
public:
    void update(float temperature) override {
        // update phone screen with new temperature
    }
};
```

### Strategy

- Defines a **family of algorithms**, encapsulates each one, and makes them **interchangeable**. Strategy lets the algorithm vary independently from the clients that use it.
- **When to use:** when you have multiple ways of doing the same thing (sorting, compression, routing, payment) and want to switch between them at runtime.

```cpp
class SortStrategy {
public:
    virtual void sort(std::vector<int>& data) = 0;
};

class QuickSort : public SortStrategy {
public:
    void sort(std::vector<int>& data) override { /* quicksort */ }
};

class MergeSort : public SortStrategy {
public:
    void sort(std::vector<int>& data) override { /* mergesort */ }
};

class Sorter {
    SortStrategy* strategy;
public:
    void setStrategy(SortStrategy* s) { strategy = s; }
    void doSort(std::vector<int>& data) { strategy->sort(data); }
};
```

### Command

- Encapsulates a **request as an object**, thereby allowing you to parameterize clients with different requests, queue requests, log them, and support undoable operations.
- **When to use:** undo/redo functionality, task queues, macro recording, callback abstraction.

```cpp
class Command {
public:
    virtual void execute() = 0;
    virtual void undo() = 0;
};

class Light {
public:
    void on() { /* turn on */ }
    void off() { /* turn off */ }
};

class LightOnCommand : public Command {
    Light* light;
public:
    LightOnCommand(Light* l) : light(l) {}
    void execute() override { light->on(); }
    void undo() override { light->off(); }
};

class RemoteControl {
    std::vector<Command*> history;
public:
    void pressButton(Command* cmd) {
        cmd->execute();
        history.push_back(cmd);
    }
    void pressUndo() {
        if (!history.empty()) {
            history.back()->undo();
            history.pop_back();
        }
    }
};
```

### State

- Allows an object to **alter its behavior when its internal state changes**. The object will appear to change its class.
- Instead of massive `if/else` or `switch` blocks checking state, each state becomes its own class.
- **When to use:** when an object's behavior depends on its state and it must change behavior at runtime, or when operations have large conditional statements that depend on the object's state.

```cpp
class State {
public:
    virtual void handle() = 0;
};

class LockedState : public State {
public:
    void handle() override { /* show "enter PIN" screen */ }
};

class UnlockedState : public State {
public:
    void handle() override { /* show home screen */ }
};

class Phone {
    State* state;
public:
    void setState(State* s) { state = s; }
    void pressHome() { state->handle(); }
};
```

### Template Method

- Defines the **skeleton of an algorithm** in a base class, deferring some steps to subclasses. Lets subclasses redefine certain steps without changing the algorithm's structure.
- **When to use:** when multiple classes share the same algorithm structure but differ in specific steps.

```cpp
class DataMiner {
public:
    // Template method - defines the skeleton
    void mine() {
        openFile();
        extractData();
        parseData();
        analyzeData();
        closeFile();
    }
protected:
    virtual void openFile() = 0;
    virtual void extractData() = 0;
    void parseData() { /* common parsing logic */ }
    void analyzeData() { /* common analysis logic */ }
    virtual void closeFile() = 0;
};

class CSVDataMiner : public DataMiner {
protected:
    void openFile() override { /* open CSV */ }
    void extractData() override { /* read CSV rows */ }
    void closeFile() override { /* close CSV */ }
};
```

### Iterator

- Provides a way to **access elements of a collection sequentially** without exposing its underlying representation (array, linked list, tree, etc.).
- **When to use:** when you want to traverse a collection without exposing its internals, or when you need multiple traversal algorithms over the same collection.

```ad-note
In C++, the STL already implements the Iterator pattern extensively via `begin()`, `end()`, and range-based for loops. You rarely need to implement this from scratch in modern C++.
```

### Chain of Responsibility

- Passes a request along a **chain of handlers**. Each handler decides either to process the request or to pass it to the next handler in the chain.
- **When to use:** when multiple objects may handle a request and the handler isn't known in advance, or when you want to decouple sender from receiver (e.g., middleware pipelines, event bubbling, logging levels).

```cpp
class Handler {
protected:
    Handler* next = nullptr;
public:
    void setNext(Handler* h) { next = h; }
    virtual void handle(int request) {
        if (next) next->handle(request);
    }
};

class LowLevelSupport : public Handler {
public:
    void handle(int severity) override {
        if (severity < 5) { /* handle it */ }
        else Handler::handle(severity); // pass up the chain
    }
};

class HighLevelSupport : public Handler {
public:
    void handle(int severity) override {
        if (severity >= 5) { /* handle it */ }
        else Handler::handle(severity);
    }
};
```

### Mediator

- Defines an object that **encapsulates how a set of objects interact**. Promotes loose coupling by keeping objects from referring to each other explicitly.
- **When to use:** when a set of objects communicate in complex but well-defined ways (e.g., a chat room, an air traffic control tower, a dialog box coordinating its widgets).

### Memento

- Captures and externalizes an object's **internal state** so it can be **restored later**, without violating encapsulation.
- **When to use:** undo mechanisms, saving game state, snapshots, transaction rollback.

### Visitor

- Lets you **add new operations** to existing object structures without modifying the structures themselves.
- Uses double dispatch: the object "accepts" a visitor, which then performs the operation.
- **When to use:** when you have a stable class hierarchy but need to frequently add new operations on those classes (e.g., compiler AST nodes with multiple passes: type checking, code generation, optimization).

---

## Quick Reference Table

| Pattern | Category | One-Line Summary |
|---|---|---|
| Singleton | Creational | One instance, global access |
| Factory Method | Creational | Subclass decides which object to create |
| Abstract Factory | Creational | Create families of related objects |
| Builder | Creational | Step-by-step complex object construction |
| Prototype | Creational | Clone existing objects |
| Adapter | Structural | Convert one interface to another |
| Decorator | Structural | Add responsibilities dynamically |
| Facade | Structural | Simplified interface to a subsystem |
| Proxy | Structural | Placeholder that controls access |
| Composite | Structural | Tree structure, uniform leaf/node treatment |
| Bridge | Structural | Separate abstraction from implementation |
| Flyweight | Structural | Share objects to save memory |
| Observer | Behavioral | Notify dependents of state changes |
| Strategy | Behavioral | Swap algorithms at runtime |
| Command | Behavioral | Encapsulate request as object (undo/redo) |
| State | Behavioral | Behavior changes with internal state |
| Template Method | Behavioral | Algorithm skeleton with customizable steps |
| Iterator | Behavioral | Sequential access without exposing internals |
| Chain of Responsibility | Behavioral | Pass request along a handler chain |
| Mediator | Behavioral | Centralize complex communication |
| Memento | Behavioral | Capture and restore state |
| Visitor | Behavioral | Add operations without modifying classes |

---

```ad-tip
**How to pick the right pattern:**
1. Identify the problem (object creation? structure? communication?)
2. That narrows it to one of the three categories
3. Within the category, match the specific constraint (need one instance? need to swap algorithms? need undo?)
4. If two patterns seem to fit, check whether you actually need the pattern at all - simpler code is always better
```
