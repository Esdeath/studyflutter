### Bridge 设计模式概述

![标题图片](./img/header.png)

想要查看所有设计模式的实际应用，请访问 [Flutter 设计模式应用程序](https://flutterdesignpatterns.com/)。

## 什么是 Bridge（桥接）设计模式？

![一只狗和它的橡胶鸡跳过桥的图片](./img/jump_over_bridge.jpeg)

**Bridge（桥接）**，也被称为 **Handle/Body（处理/体）**，属于结构型设计模式的一种。在 [GoF 的书籍](https://en.wikipedia.org/wiki/Design_Patterns)中，该设计模式的目的被描述为：

> _将抽象与其实现分离，使两者可以独立变化。_

通常，抽象要有几种可能的实现方式，使用的方法是继承 — 抽象定义接口，而具体的子类以不同的方式实现它。然而，这种方法并不灵活，因为它在编译时就将实现与抽象绑定，使得在运行时无法更改实现。如果我们想在运行时选择并交换实现呢？

Bridge 设计模式将抽象与其实现分离，使两者可以独立变化。在这种情况下，抽象使用另一个抽象作为其实现，而不是直接使用实现。这种抽象与其实现（更具体地说，是另一个抽象）之间的关系被称为桥接 — 它连接了抽象与其实现，让它们可以独立变化。

如果 _Abstraction（抽象）_ 和 _Implementation（实现）_ 这些术语听起来太学术化，那么可以这样想象：抽象（或接口）只是某个特定实体的高层层次。这一层只是一个接口，本身并不做任何实际工作 — 它应该将工作委托给实现层。这方面的一个很好的例子是 GUI（图形用户界面）和 OS（操作系统）。GUI 只是用户与操作系统交流的顶层层次，但它本身并不进行任何实际工作 — 它只是将用户命令（事件）传递给平台。重要的是，无论是 GUI 还是 OS 都可以彼此独立扩展，例如，桌面应用程序可能有不同的视图/面板/仪表板，同时支持多个 API（可以在 Windows、Linux 和 macOS 上运行） — 这两部分可以独立变化。听起来像是 Bridge 设计模式，对吗？

## 分析

Bridge 设计模式的总体结构如下所示：

![Bridge 设计模式的结构](./img/bridge.png)

- *Abstraction（抽象）* - 定义了抽象的接口，并维护一个类型为 _Implementation（实现）_ 的对象的引用；
- *Refined Abstraction（精化抽象）* - 实现了 _Abstraction_ 接口，并提供了控制逻辑的不同变体；
- *Implementation（实现）* - 为实现类定义了一个接口。_Abstraction_ 只能通过声明在这里的方法与 _Implementation_ 对象进行通信；
- *Concrete Implementations（具体实现）* - 实现了 _Implementation_ 接口，并包含了特定于平台的代码。

### 适用性

当你想要分割一个包含了功能多个变体的单一类时，应该使用 Bridge 设计模式。这种情况下，该模式允许将类拆分为几个类层次结构，它们可以独立变化 — 这简化了代码维护，并且较小的类减少了破坏现有代码的风险。这种方法的一个好例子是，当你想在持久化层中使用多种不同的方法，例如同时使用数据库和文件系统持久化。

此外，当抽象及其实现都需要通过子类化进行扩展时，也应该使用桥接设计模式 — 该模式允许结合不同的抽象和实现，并独立地扩展它们。

最后，当你需要在运行时切换实现时，桥接设计模式是救星。该模式允许你在抽象内部替换实现对象 — 你可以通过构造器注入它，或者仅将其作为字段/属性的新值赋值。

## 实现

![我们开始工作吧](./img/lets_get_to_work.gif)

在实现部分，我们将使用 Bridge 设计模式实现我们示例的持久化层。

假设你的应用程序使用外部 SQL 数据库（不是设备中的本地 SQLite 选项，而是云端的）。一切都很好，直到突然出现连接问题。这种情况下，有两个选择：不允许用户使用应用程序并提供一个有趣的“连接丢失”屏幕，或者你可以将数据存储在某种本地存储中，并在连接恢复后同步数据。显然，第二种方法更加用户友好，但如何实现呢？

在持久化层中，有多个针对每种实体类型的仓库。这些仓库共享一个公共接口 — 这就是我们的抽象。如果你想在运行时更改存储类型（使用本地或云端存储），这些仓库不能引用特定的存储实现，它们应该使用不同类型存储之间共享的某种抽象。好吧，我们可以在此基础上构建另一个抽象（接口），然后由特定存储实现。现在我们将我们的仓库抽象与存储的接口连接起来 — 就是这样，Bridge 设计模式就被引入到我们的应用程序中了！让我们先看一下类图，然后再探究一些实现细节。


### Class diagram

The class diagram below shows the implementation of the Bridge design pattern:

![Class Diagram - Implementation of the Bridge design pattern](./img/bridge_implementation.png)

The `EntityBase` is an abstract class that is used as a base class for all the entity classes. The class contains an `id` property and a named constructor `EntityBase.fromJson()` to map the JSON object to the class field.

`Customer` and `Order` are concrete entities that extend the abstract class `EntityBase`. `Customer` class contains `name` and `email` properties, `Customer.fromJson()` named constructor to map the JSON object to class fields and a `toJson()` method to map class fields to the corresponding JSON map object. `Order` class contain `dishes` (a list of dishes of that order) and `total` fields, a named constructor `Order.fromJson()` and a `toJson()` method respectively.

`IRepository` defines a common interface for the repositories:

- `getAll()` - returns all records from the repository;
- `save()` - saves an entity of type `EntityBase` in the repository.

`CustomersRepository` and `OrdersRepository` are concrete implementations of the `IRepository` interface. Also, these classes contain a storage property of type `IStorage` which is injected into the repository via the constructor.

`IStorage` defines a common interface for the storages:

- `getTitle()` - returns the title of the storage. The method is used in UI;
- `fetchAll<T>()` - returns all the records of type `T` from the storage;
- `store<T>()` - stores a record of type `T` in the storage.

`FileStorage` and `SqlStorage` are concrete implementations of the `IStorage` interface. Additionally, the `FileStorage` class uses the `JsonHelper` class and its static methods to serialise/deserialise JSON objects.

`BridgeExample` initialises and contains both - customer and order - repositories which are used to retrieve the corresponding data. Additionally, the storage type of these repositories could be changed between the `FileStorage` and `SqlStorage` separately and at the run-time.

### EntityBase

An abstract class that stores the `id` field and is extended by all of the entity classes.

```dart title="entity_base.dart"
abstract class EntityBase {
  EntityBase() : id = faker.guid.guid();

  final String id;

  EntityBase.fromJson(Map<String, dynamic> json) : id = json['id'] as String;
}
```

### Customer

A simple class to store information about the customer: its `name` and `email`. Also, the constructor generates random values when initialising the `Customer` object.

```dart title="customer.dart"
class Customer extends EntityBase {
  Customer()
      : name = faker.person.name(),
        email = faker.internet.email();

  final String name;
  final String email;

  Customer.fromJson(super.json)
      : name = json['name'] as String,
        email = json['email'] as String,
        super.fromJson();

  Map<String, dynamic> toJson() => {
        'id': id,
        'name': name,
        'email': email,
      };
}
```

### Order

A simple class to store information about the order: a list of `dishes` it contains and the `total` price of the order. Also, the constructor generates random values when initialising the `Order` object.

```dart title="order.dart"
class Order extends EntityBase {
  Order()
      : dishes = List.generate(
          random.integer(3, min: 1),
          (_) => faker.food.dish(),
        ),
        total = random.decimal(scale: 20, min: 5);

  final List<String> dishes;
  final double total;

  Order.fromJson(super.json)
      : dishes = List.from(json['dishes'] as List),
        total = json['total'] as double,
        super.fromJson();

  Map<String, dynamic> toJson() => {
        'id': id,
        'dishes': dishes,
        'total': total,
      };
}
```

### JsonHelper

A helper class is used by the `FileStorage` to serialise objects of type `EntityBase` to JSON map objects and deserialise them from the JSON string.

```dart title="json_helper.dart"
class JsonHelper {
  const JsonHelper._();

  static String serialiseObject(EntityBase entityBase) {
    return jsonEncode(entityBase);
  }

  static T deserialiseObject<T extends EntityBase>(String jsonString) {
    final json = jsonDecode(jsonString)! as Map<String, dynamic>;

    return switch (T) {
      Customer => Customer.fromJson(json) as T,
      Order => Order.fromJson(json) as T,
      _ => throw Exception("Type of '$T' is not supported."),
    };
  }
}
```

### IRepository

An interface that defines methods to be implemented by the derived repository classes.

```dart title="irepository.dart"
abstract interface class IRepository {
  List<EntityBase> getAll();
  void save(EntityBase entityBase);
}
```

### Concrete repositories

`CustomersRepository` - a specific implementation of the `IRepository` interface to store customers' data.

```dart title="customers_repository.dart"
class CustomersRepository implements IRepository {
  const CustomersRepository(this.storage);

  final IStorage storage;

  @override
  List<EntityBase> getAll() => storage.fetchAll<Customer>();

  @override
  void save(EntityBase entityBase) {
    storage.store<Customer>(entityBase as Customer);
  }
}
```

`OrdersRepository` - a specific implementation of the `IRepository` interface to store orders' data.

```dart title="orders_repository.dart"
class OrdersRepository implements IRepository {
  const OrdersRepository(this.storage);

  final IStorage storage;

  @override
  List<EntityBase> getAll() => storage.fetchAll<Order>();

  @override
  void save(EntityBase entityBase) {
    storage.store<Order>(entityBase as Order);
  }
}
```

### IStorage

An interface that defines methods to be implemented by the derived storage classes.

```dart title="istorage.dart"
abstract interface class IStorage {
  String getTitle();
  List<T> fetchAll<T extends EntityBase>();
  void store<T extends EntityBase>(T entityBase);
}
```

### Concrete storages

`FileStorage` - a specific implementation of the `IStorage` interface to store an object in the storage as a file - this behaviour is mocked by storing an object as a JSON string.

```dart title="file_storage.dart"
class FileStorage implements IStorage {
  final Map<Type, List<String>> fileStorage = {};

  @override
  String getTitle() => 'File Storage';

  @override
  List<T> fetchAll<T extends EntityBase>() {
    if (!fileStorage.containsKey(T)) return [];

    final files = fileStorage[T]!;

    return files.map<T>((f) => JsonHelper.deserialiseObject<T>(f)).toList();
  }

  @override
  void store<T extends EntityBase>(T entityBase) {
    if (!fileStorage.containsKey(T)) fileStorage[T] = [];

    fileStorage[T]!.add(JsonHelper.serialiseObject(entityBase));
  }
}
```

`SqlStorage` - a specific implementation of the `IStorage` interface to store an object in the storage as an entity - this behaviour is mocked by using the Map data structure and appending entities of the same type to the list.

```dart title="sql_storage.dart"
class SqlStorage implements IStorage {
  final Map<Type, List<EntityBase>> sqlStorage = {};

  @override
  String getTitle() => 'SQL Storage';

  @override
  List<T> fetchAll<T extends EntityBase>() =>
      sqlStorage.containsKey(T) ? sqlStorage[T]! as List<T> : [];

  @override
  void store<T extends EntityBase>(T entityBase) {
    if (!sqlStorage.containsKey(T)) sqlStorage[T] = <T>[];

    sqlStorage[T]!.add(entityBase);
  }
}
```

## Example

First of all, a markdown file is prepared and provided as a pattern's description:

![Example markdown](./img/example_markdown.gif)

`BridgeExample` contains a list of storage - instances of `SqlStorage` and `FileStorage` classes. Also, it initialises `Customer` and `Order` repositories. In the repositories the concrete type of storage could be interchanged by triggering the `onSelectedCustomerStorageIndexChanged()` for the `CustomersRepository` and `onSelectedOrderStorageIndexChanged()` for the `OrdersRepository` respectively.

```dart title="bridge_example.dart"
class BridgeExample extends StatefulWidget {
  const BridgeExample();

  @override
  _BridgeExampleState createState() => _BridgeExampleState();
}

class _BridgeExampleState extends State<BridgeExample> {
  final _storages = [SqlStorage(), FileStorage()];

  late IRepository _customersRepository;
  late IRepository _ordersRepository;

  late List<Customer> _customers;
  late List<Order> _orders;

  var _selectedCustomerStorageIndex = 0;
  var _selectedOrderStorageIndex = 0;

  void _onSelectedCustomerStorageIndexChanged(int? index) {
    if (index == null) return;

    setState(() {
      _selectedCustomerStorageIndex = index;
      _customersRepository = CustomersRepository(_storages[index]);
      _customers = _customersRepository.getAll() as List<Customer>;
    });
  }

  void _onSelectedOrderStorageIndexChanged(int? index) {
    if (index == null) return;

    setState(() {
      _selectedOrderStorageIndex = index;
      _ordersRepository = OrdersRepository(_storages[index]);
      _orders = _ordersRepository.getAll() as List<Order>;
    });
  }

  void _addCustomer() {
    _customersRepository.save(Customer());

    setState(
      () => _customers = _customersRepository.getAll() as List<Customer>,
    );
  }

  void _addOrder() {
    _ordersRepository.save(Order());

    setState(() => _orders = _ordersRepository.getAll() as List<Order>);
  }

  @override
  void initState() {
    super.initState();

    _customersRepository =
        CustomersRepository(_storages[_selectedCustomerStorageIndex]);
    _customers = _customersRepository.getAll() as List<Customer>;

    _ordersRepository = OrdersRepository(_storages[_selectedOrderStorageIndex]);
    _orders = _ordersRepository.getAll() as List<Order>;
  }

  @override
  Widget build(BuildContext context) {
    return ScrollConfiguration(
      behavior: const ScrollBehavior(),
      child: SingleChildScrollView(
        padding: const EdgeInsets.symmetric(
          horizontal: LayoutConstants.paddingL,
        ),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: <Widget>[
            Row(
              children: <Widget>[
                Text(
                  'Select customers storage:',
                  style: Theme.of(context).textTheme.titleLarge,
                ),
              ],
            ),
            StorageSelection(
              storages: _storages,
              selectedIndex: _selectedCustomerStorageIndex,
              onChanged: _onSelectedCustomerStorageIndexChanged,
            ),
            PlatformButton(
              materialColor: Colors.black,
              materialTextColor: Colors.white,
              onPressed: _addCustomer,
              text: 'Add',
            ),
            if (_customers.isNotEmpty)
              CustomersDatatable(customers: _customers)
            else
              Text(
                '0 customers found',
                style: Theme.of(context).textTheme.titleSmall,
              ),
            const Divider(),
            Row(
              children: <Widget>[
                Text(
                  'Select orders storage:',
                  style: Theme.of(context).textTheme.titleLarge,
                ),
              ],
            ),
            StorageSelection(
              storages: _storages,
              selectedIndex: _selectedOrderStorageIndex,
              onChanged: _onSelectedOrderStorageIndexChanged,
            ),
            PlatformButton(
              materialColor: Colors.black,
              materialTextColor: Colors.white,
              onPressed: _addOrder,
              text: 'Add',
            ),
            if (_orders.isNotEmpty)
              OrdersDatatable(orders: _orders)
            else
              Text(
                '0 orders found',
                style: Theme.of(context).textTheme.titleSmall,
              ),
          ],
        ),
      ),
    );
  }
}
```

The concrete repository does not care about the specific type of storage it uses as long as the storage implements the `IStorage` interface and all of its methods. As a result, the abstraction (repository) is separated from the implementor (storage) - the concrete implementation of the storage could be changed for the repository at run-time, and the repository does not depend on its implementation details.

![Bridge example](./img/example.gif)

As you can see in the example, the storage type could be changed for each repository separately and at run-time - it would not be possible by using the simple class inheritance approach.

All of the code changes for the Bridge design pattern and its example implementation could be found [here](https://github.com/mkobuolys/flutter-design-patterns/pull/18).

:::tip
To see the pattern in action, check the [interactive Bridge example](https://flutterdesignpatterns.com/pattern/bridge).
:::
