
### **Введение**

Кто не хочет писать качественный (solid) код? Конечно, есть просто надежный код, а есть SOLID код. SOLID — это акроним, описывающий пять принципов, которые помогут вам писать чистый и надежный код.

#### **Пять принципов SOLID:**
*   **Single Responsibility Principle** (Принцип единой ответственности — SRP)
*   **Open/Closed Principle** (Принцип открытости/закрытости — OCP)
*   **Liskov’s Substitution Principle** (Принцип подстановки Барбары Лисков — LSP)
*   **Interface Segregation Principle** (Принцип разделения интерфейса — ISP)
*   **Dependency Inversion Principle** (Принцип инверсии зависимостей — DIP)

Мы разберем каждый из них по порядку. Эти принципы были впервые собраны Робертом С. Мартином, более известным как «Дядя Боб». [11](http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod)
**Принципы** — это всего лишь принципы. Это скорее рекомендации, чем жесткие правила. Они не могут быть заключены в какой-то один фреймворк или библиотеку кода. Часто они используются вместе. Например, применение принципа подстановки Лисков почти неизбежно приводит к написанию кода, следующего принципам единой ответственности и инверсии зависимостей.

#### **Что такое SOLID**

Несмотря на обилие языков на базе ООП, многие разработчики склонны писать процедурный код. Классический пример — программирование логики в событии «OnClick». Слишком часто мы думаем, что создаем иерархию классов, но на самом деле просто оборачиваем наш процедурный код в классы. Типичный пример — «Божественный класс» (God Class), который начинается с одного класса, в который добавляется всё подряд, пока он не начинает делать абсолютно всё. Вы наверняка видели такие классы в качестве потомков `TForm`.

Принципы SOLID призваны гарантировать, что вы пишете настоящий объектно-ориентированный код. Если вы следуете им, будет очень сложно просто писать процедурный код в объектах. Они заставят вас думать о том, как спроектированы ваши объекты и как они должны быть правильно написаны. Их цель — помочь вам писать более качественный ООП код — код, который правильно декомпозирован, слабо связан и, следовательно, легко поддерживаем и тестируем.

Если вы начнете писать SOLID код, произойдет следующее: у вас будет больше классов, больше модулей и больше юнитов, чем вы привыкли. И это нормально. По какой-то причине разработчики опасаются «большого количества классов», предпочитая меньшее количество классов, которые делают больше вещей. Но разве есть определение хуже для «сильно связанного кода», чем «меньше классов, которые делают больше вещей»? Большее количество классов, каждый из которых делает одну вещь (Принцип единой ответственности), — это путь к чистому коду.

Я говорил это раньше и скажу снова: нет конца вреду, который возникает, когда разработчик пытается заставить одну вещь делать больше, чем одну работу. Поэтому не бойтесь декомпозированного, слабо связанного кода, даже если он приводит к обилию классов, интерфейсов и модулей.

SOLID код также решит проблему многих «запахов проектирования» (Design Smells), которые отравляют код:
*   Код, который трудно изменять из-за сильной связанности и процедурного стиля, называют **жестким** (rigid). Принципы SOLID сделают ваш код менее жестким.
*   Код, который легко ломается из-за плохой организации связей, пересекающих все границы, называют **хрупким** (fragile). SOLID сделает ваш код менее хрупким.
*   Иногда мы пишем код, который **неподвижен** (immobile) — то есть его трудно использовать повторно в новых местах. Принципы SOLID сделают ваш код мобильным и пригодным для повторного использования.
*   **Вязкость** (viscosity) — это показатель способности вашего кода позволять вам поступать правильно. Принципы SOLID сделают ваш код более «вязким» в хорошем смысле — позволяя вам делать правильные вещи, когда вы знаете, что должны их делать, вместо того чтобы поступать неправильно.
*   Наконец, наш код может стать **перегруженным** (over-designed), когда мы наделяем его ненужной сложностью. SOLID код прост и чист, он поможет вам предотвратить избыточное проектирование.

Ладно, давайте сразу перейдем к делу.

#### **Принцип единой ответственности (Single Responsibility Principle)**

Принцип единой ответственности гласит, что класс должен делать одну и только одну вещь. Дядя Боб говорит, что это означает: «у класса должна быть только одна причина для изменения». Как бы вы на это ни смотрели, класс не должен делать несколько вещей.

**Рассмотрим следующее объявление класса:**

```pascal
type
  TBook = class
  private
    FCurrentPage: integer;
    FTitle: string;
    FAuthor: string;
    procedure SetTitle(const Value: string);
    procedure SetAuthor(const Value: string);
  public
    procedure DisplayPage;     // Отображение
    function TurnPage: integer; // Листание
    procedure PrintCurrentPage; // Печать
    procedure SaveCurrentPage;  // Сохранение
    property Title: string read FTitle write SetTitle;
    property Author: string read FAuthor write SetAuthor;
  end;
```

Этот класс `TBook`, вероятно, похож на многие классы, которые вы писали или видели. И всё кажется разумным, верно? Книга делает кучу вещей: отображает страницы, листает их, печатает и т.д. Что тут может не нравиться?

Что тут не нравится, так это то, что если вы захотите изменить способ печати страниц, вам придется изменять сам класс. А возможно, кто-то другой использует этот класс и ему нравится, как работает печать. Класс завязан на печать, отображение и сохранение.

Другими словами, этот класс является грубым нарушением принципа единой ответственности. Каково же решение? Как насчет такого:

```pascal
type
  IPrintable = interface
    procedure Print(aString: string);
  end;

  TConsolePrinter = class(TInterfacedObject, IPrintable)
    procedure Print(aString: string);
  end;

  IPageNumberSaver = interface
    procedure Save(aPageNumber: integer);
  end;

type
  TBookManager = class
  private
    FCurrentPage: integer;
    FTitle: string;
    FAuthor: string;
    procedure SetTitle(const Value: string);
    procedure SetAuthor(const Value: string);
  public
    function TurnPage: integer;
    procedure PrintCurrentPage(aPage: IPrintable);
    procedure SavePageNumber(aPageNumberSaver: IPageNumberSaver);
    property Title: string read FTitle write SetTitle;
    property Author: string read FAuthor write SetAuthor;
  end;
```

Этому классу `TBook` (здесь названному `TBookManager`) теперь не нужно меняться. Мы использовали инверсию зависимостей (DIP), чтобы вынести функциональность печати и сохранения. Если вы не хотите печатать на консольный принтер, просто передайте другую реализацию `IPrintable`. Хотите сохранить номер страницы в INI-файл или базу данных? Передайте соответствующую реализацию `IPageNumberSaver`. `TBook` ничего не знает о внутреннем устройстве этих реализаций и не имеет причин меняться из-за них.

`TBook` теперь делает одну вещь: он является книгой. Он больше не «штука для печати» или «штука для сохранения». Вы измените его только в том случае, если захотите изменить поведение самой книги. Ответственность за печать и сохранение передана внешним интерфейсам.

Вот что подразумевается под принципом единой ответственности.

#### **Принцип открытости/закрытости (Open/Closed Principle)**

Принцип открытости/закрытости гласит, что класс должен быть открыт для расширения, но закрыт для изменения. То есть правильно спроектированный класс после развертывания никогда не должен нуждаться в изменениях (за исключением исправления ошибок), и если появляется новая функциональность, она должна добавляться через расширение или наследование.

Мы все виновны в нарушении этого принципа. Если вы использовали оператор `case` в своем коде, весьма вероятно, что вы нарушаете принцип открытости/закрытости. Посмотрите на этот код:

```pascal
type
  TItemType = (Normal, TenPercentOff, BOGO, TwentyPercentOff);
  TItem = record
    SKU: string;
    Quantity: integer;
    Price: double;
    ItemType: TItemType;
  end;

  TOrder = class
  private
    FListOfItems: IList<TItem>;
    function GetItems: IEnumerable<TItem>;
  public
    procedure Add(aItem: TItem);
    function TotalAmount: Double;
    property Items: IEnumerable<TItem> read GetItems;
  end;

implementation

procedure TOrder.Add(aItem: TItem);
begin
  FListOfItems.Add(aItem);
end;

function TOrder.TotalAmount: Double;
var
  Item: TItem;
  LQuantity: integer;
begin
  for Item in FListOfItems do
  begin
    case Item.ItemType of
      Normal:
        begin
          Result := (Item.Price * Item.Quantity);
        end;
        
      TenPercentOff: 
        begin
          Result := (Item.Price * 0.90 * Item.Quantity);
        end;
        
      BOGO: 
        begin // Buy One Get One
          LQuantity := Item.Quantity * 2;
          Result := Item.Price * LQuantity;
        end;
      // Будут новые типы!
    end;
  end;
end;
```

Этот код широко открыт для изменений. Если вы захотите добавить новый тип распродажи, вам придется залезть внутрь оператора `case` функции `TotalAmount`, чтобы завершить это изменение. Класс определенно не закрыт для изменений.

Например, если вы захотите добавить новый тип `TwentyPercentOff`, вам придется добавить элемент в перечислимый тип (не такая уж большая проблема), но затем добавить еще один пункт в оператор `case`. Это не тот путь, которым вы хотите следовать, так как изменение самого класса упрощает внесение ошибок и заставляет обновлять тесты. Почему бы не спроектировать вещи так, чтобы вы могли развернуть класс один раз, а затем расширять его, вместо того чтобы «вскрывать» и менять его?

Вот пример использования интерфейса (ISP) и инверсии зависимостей (DIP) для достижения того же результата способом, который легко расширять:

```pascal
type
  IItemPricer = interface;

  TItem = record
    ID: string;
    Price: double;
    Quantity: integer;
    Rule: IItemPricer;
    constructor Create(aID: string; aPrice: Double; aQuantity: integer; aRule: IItemPricer);
  end;

  IItemPricer = interface
    function CalculatePrice(aItem: TItem): Double;
  end;

  TNormalPricer = class(TInterfacedObject, IItemPricer)
    function CalculatePrice(aItem: TItem): Double;
  end;

  TTenPercentOffPricer = class(TInterfacedObject, IItemPricer)
    function CalculatePrice(aItem: TItem): Double;
  end;

  TBOGOFreePricer = class(TInterfacedObject, IItemPricer)
    function CalculatePrice(aItem: TItem): Double;
    
  TTwentyPercentOffPricer = class(TInterfacedObject, IItemPricer)
    function CalculatePrice(aItem: TItem): Double;
  end;

  TOrder = class
  private
    FListOfItems: IList<TItem>;
    function GetItems: IEnumerable<TItem>;
  public
    procedure Add(aItem: TItem);
    function TotalAmount: Double;
    property Items: IEnumerable<TItem> read GetItems;
    constructor Create;
  end;
```

Сначала мы объявляем интерфейс `IItemPricer`, который будет обрабатывать задачу расчета цены для конкретного товара в нашем инвентаре. Затем `TItem` принимает `IItemPricer` в качестве параметра своего конструктора. Далее мы реализуем `IItemPricer` несколько раз в отдельных классах с различными схемами ценообразования. После этого мы можем просто добавить их к товару в конструкторе.

Мы также можем создавать новые правила, не меняя класс `TOrder`. Он готов работать с любым количеством товаров, и эти товары готовы принять любые правила расчета цены. Таким образом, `TOrder` закрыт для изменений — нет причин менять его — и открыт для расширения: вы можете легко создавать новые правила расчета и добавлять их в список товаров `TOrder`.

Конечно, у нас стало больше классов, но каждый из них очень легко тестировать, расширение системы совсем не сложно, а базовый класс `TOrder` не нужно менять, даже если мы захотим добавить новый способ предоставления скидок.

#### **Принцип подстановки Барбары Лисков (Liskov’s Substitution Principle)**

Принцип подстановки Лисков назван в честь Барбары Лисков, которая его сформулировала. Она заявила это довольно формально:

> *«Пусть $q(x)$ является свойством, верным для объектов $x$ типа $T$. Тогда $q(y)$ должно быть верным для объектов $y$ типа $S$, где $S$ является подтипом $T$».*

Признаюсь, мне потребовалось время, чтобы это расшифровать. Дядя Боб выразил это проще: «Подтипы должны быть заменяемы своими базовыми типами». Марк Симан (Mark Seemann) сформулировал это так: «Клиент должен иметь возможность использовать любую реализацию интерфейса без нарушения корректности системы». По сути, это означает, что вы должны проектировать систему так, чтобы либо дочерний класс, либо любая реализация интерфейса могли быть подставлены вместо базового типа или данного интерфейса без каких-либо поломок.

Думаю, здесь необходим пример. Как насчет классического примера с `TVehicle` (Транспортное средство)? Это всегда благодатная почва для проблем:

```pascal
TVehicle = class abstract
  procedure Go; virtual; abstract;
  procedure FillWithGas; virtual; abstract;
end;

TCar = class(TVehicle)
  procedure Go; override;
  procedure FillWithGas; override;
end;
```

Это работает отлично. Реализации методов будут такими, как вы ожидаете, и `TCar` может полиморфно использоваться как `TVehicle`. Всё хорошо. Но, конечно, что произойдет, когда вы попытаетесь определить `TBicycle` (Велосипед)?

```pascal
TBicycle = class(TVehicle)
  procedure Go; override;
  procedure FillWithGas; override;
end;

procedure TBicycle.Go;
begin
  WriteLn('Крути педали как сумасшедший!');
end;

procedure TBicycle.FillWithGas;
begin
  raise EVehicleException.Create('Глупышка, нельзя заправить велосипед бензином.');
end;
```

Итак, вот ваша первая улика того, что вы нарушаете принцип подстановки Лисков: чтобы реализовать класс, вам приходится генерировать исключение или иным образом не реализовывать данный метод потомка. Если ваш производный класс не может должным образом реализовать то, что он *обязан* реализовать, ваш базовый класс нарушает принцип подстановки Лисков.

Как же поступить правильно? Возможно, объявить вещи следующим образом:

```pascal
type
  TVehicle = class abstract
    procedure Go; virtual; abstract;
  end;

  TGasVehicle = class abstract(TVehicle)
    procedure FillWithGas; virtual; abstract;
  end;

  TCar = class(TGasVehicle)
    procedure Go; override;
    procedure FillWithGas; override;
  end;

  TTruck = class(TGasVehicle)
    procedure Go; override;
    procedure FillWithGas; override;
  end;

  TBicycle = class(TVehicle)
    procedure Go; override;
  end;
```

Здесь мы убрали требование наличия бензина из `TVehicle` и вынесли его в собственный класс `TGasVehicle`. Это оставляет `TBicycle` чистым, позволяя ему заботиться только о методе `Go` и не беспокоиться о заправке.

Принцип подстановки Лисков также работает с интерфейсами. Давайте взглянем. Вот интерфейс для птицы:

```pascal
IBird = interface
  procedure Fly;
  procedure Eat;
end;
```

Птицы летают и едят, верно? Идеальный интерфейс. (Хе-хе...) Вот реализация:

```pascal
TCrow = class(TInterfacedObject, IBird)
  procedure Fly;
  procedure Eat;
end;
```

Теперь давайте реализуем `TPenguin` (Пингвин):

```pascal
TPenguin = class(TInterfacedObject, IBird)
  procedure Fly;
  procedure Eat;
end;
```

Ой-ой. Что нам делать с реализацией метода `Fly`? Пингвины — отличные пловцы, но они совсем не умеют летать, и теперь мы застряли так же, как и раньше. Что делать? Как насчет этого:

```pascal
IFlyable = interface
  procedure Fly;
end;

IEater = interface
  procedure Eat;
end;

TCrow = class(TInterfacedObject, IFlyable, IEater)
  procedure Fly;
  procedure Eat;
end;

TPenguin = class(TInterfacedObject, IEater)
  procedure Eat;
end;
```

Наш первый `TPenguin` нарушал принцип подстановки Лисков, но когда мы разделили интерфейсы, мы можем применять их по мере необходимости, и всё в порядке. И вот забавная часть — помните, я говорил, что эти принципы часто работают вместе? Как мы увидим ниже, решение выше использовало принцип разделения интерфейса (ISP) для решения дилеммы, созданной `IBird` и `TPenguin`.

Вот список вещей, которые могут заставить вас подумать, что вы нарушаете принцип подстановки Лисков:
*   Ваш код выбрасывает `NotSupportedException` или что-то подобное при реализации метода.
*   Вам приходится делать много приведений типов (typecasting), чтобы получить правильный класс во всех случаях.
*   Выделенные интерфейсы, которые не разделены должным образом, часто приводят к нарушению LSP.
*   Часто вы обнаруживаете, что вам приходится удалять функции при реализации интерфейса. Это может быть признаком того, что вы нарушаете LSP. Пример — коллекция только для чтения, где вам пришлось бы удалять такие вещи, как `Add` и `Insert`, чтобы всё это работало.

#### **Принцип разделения интерфейса (Interface Segregation Principle)**

Принцип разделения интерфейса гласит: «Клиенты не должны зависеть от методов, которые они не используют». То есть клиент должен сам определять, что ему нужно, и его не должны заставлять зависеть от вещей, которые ему не нужны или в которых он не нуждается. Интерфейс определяется клиентом, а не наоборот.

> Считаете ли вы, что совершаете грубое нарушение ISP? Возьмите один из ваших классов, скопируйте его секцию `public` и превратите её в интерфейс. Я почти на 100% гарантирую, что это нарушит ISP. Подумайте об этом — поступая так, вы заставляете реализацию, а не клиента, определять, что представляет собой интерфейс.

Чтобы следовать ISP, вы должны отдавать предпочтение написанию «ролевых» интерфейсов (role-based) вместо «заголовочных» (header-based). Мы видели это в разделе про принцип подстановки Лисков: вместо одного `IBird` с методами `Fly` и `Eat`, вы должны позволить клиенту диктовать условия — в данном случае `TCrow` и `TPenguin` — и иметь два отдельных интерфейса вместо одного — `IFlyable` и `IEater`.

Не бойтесь иметь много маленьких интерфейсов, даже если в них всего по одному методу. Чем меньше интерфейс, тем больше вероятность, что он будет использован, и чем он «толще», тем меньше вероятность его повторного использования. Повторно используемый код — это хорошо, верно?

Давайте рассмотрим более практичный пример из реального мира. Представьте, что вы решили бросить всё это программирование на Delphi и заняться Netflix, открыв видеопрокат. Вы решаете создать небольшое приложение для управления вашим инвентарем. В результате вы объявляете следующий интерфейс:

```pascal
type
  IProduct = interface
    procedure SetPrice(aValue: Double);
    function GetPrice: Double;
    procedure SetStock(aValue: integer);
    function GetStock: integer;
    procedure SetAgeLimit(aValue: integer);
    function GetAgeLimit: integer;
    procedure SetRunningTime(aValue: integer);
    function GetRunningTime: integer;

    property Price: Double read GetPrice write SetPrice;
    property Stock: integer read GetStock write SetStock;
    property AgeLimit: integer read GetAgeLimit write SetAgeLimit;
    property RunningTime: integer read GetRunningTime write SetRunningTime;
  end;
```

Затем вы реализуете этот интерфейс для DVD и Blu-ray дисков. (Вот объявление для DVD...)

```pascal
TDVD = class(TInterfacedObject, IProduct)
private
  FPrice: Double;
  FStock: integer;
  FAgeLimit: integer;
  FRunningTime: integer;
  // ... реализация процедур и функций ...
public
  property Price: Double read GetPrice write SetPrice;
  property Stock: integer read GetStock write SetStock;
  property AgeLimit: integer read GetAgeLimit write SetAgeLimit;
  property RunningTime: integer read GetRunningTime write SetRunningTime;
end;
```

Это работает отлично — вы можете отслеживать продажи, уровень запасов, быстро определять, можно ли 13-летнему подростку смотреть фильм и т.д. `IProduct` работает замечательно.

На самом деле дела идут настолько хорошо, что вы решаете продавать футболки (T-Shirts) с фанатскими комментариями. Вы добавляете их в инвентарь, реализуя `TShirt` с помощью `IProduct`... и тут начинаются проблемы. Футболка — это, конечно, товар, но она не является видео. Хотя на ней нет никакой грубости, свойства `AgeLimit` (возрастной ценз) и `RunningTime` (длительность) не имеют смысла для продажи футболки.

И тут вас осеняет — Принцип разделения интерфейса нарушен! Какой же вы были глупец! Вы позволили реализации — DVD и Blu-ray — определять интерфейс, а не наоборот. Итак, вы делаете умный ход и разделяете интерфейсы:

```pascal
IProduct = interface
  procedure SetPrice(aValue: Double);
  function GetPrice: Double;
  procedure SetStock(aValue: integer);
  function GetStock: integer;

  property Price: Double read GetPrice write SetPrice;
  property Stock: integer read GetStock write SetStock;
end;

IVideo = interface(IProduct)
  procedure SetAgeLimit(aValue: integer);
  function GetAgeLimit: integer;
  procedure SetRunningTime(aValue: integer);
  function GetRunningTime: integer;

  property AgeLimit: integer read GetAgeLimit write SetAgeLimit;
  property RunningTime: integer read GetRunningTime write SetRunningTime;
end;
```

Теперь вы можете объявить свой DVD так (Blu-ray будут выглядеть точно так же):

```pascal
TDVD = class(TInterfacedObject, IProduct, IVideo)
  ...
end;
```

А вашу футболку вот так:

```pascal
type
  TShirt = class(TInterfacedObject, IProduct)
  private
    FPrice: Double;
    FStock: integer;
    // ... реализация методов ...
  public
    property Price: Double read GetPrice write SetPrice;
    property Stock: integer read GetStock write SetStock;
  end;
```

Ваша футболка — это не видео, поэтому она не должна определять или нести в себе багаж того, что в итоге стало интерфейсом `IVideo`. Футболка всё еще может быть продуктом (`IProduct`), но она определенно не видео. Следовательно, их не следует заставлять делить один и тот же интерфейс. Эти два интерфейса должны быть разделены — или сегрегированы — друг от друга. В этом и заключается суть принципа разделения интерфейса.

#### **Принцип инверсии зависимостей (Dependency Inversion Principle)**

Мы уже видели принцип инверсии зависимостей в действии, когда рассматривали принцип единой ответственности. Мы видели, как функциональность печати и сохранения была «инвертирована» с помощью интерфейсов, чтобы класс, о котором шла речь, перестал заниматься печатью и сохранением.

Самое простое определение DIP, которое я могу придумать, — это известная фраза: **«Программируйте, используя абстракции, а не реализации»**. Другой способ выразить это: «Всегда зависьте от интерфейса, а не от реализации». Более формально это звучит так: «Модули верхних уровней не должны зависеть от модулей нижних уровней. Оба должны зависеть от абстракций. Абстракции не должны зависеть от деталей. Детали должны зависеть от абстракций».

> Здесь следует отметить, что инверсия зависимостей похожа на внедрение зависимостей (Dependency Injection — DI), но это не совсем одно и то же. Для отличного описания этого тонкого различия я предлагаю прочитать пост в блоге Дерика Бэйли (Derick Bailey). [12](http://lostechies.com/derickbailey/2011/09/22/dependency-injection-is-not-the-same-as-the-dependency-inversion-principle/)

Фактически, принцип инверсии зависимостей и принцип подстановки Лисков тесно связаны: точно так же, как вы всегда должны иметь возможность подключить фен, лампу или любой другой электроприбор в розетку вашего дома, вы должны иметь возможность взять одну реализацию интерфейса и заменить её другой реализацией этого интерфейса. Если ваши приборы жестко вмонтированы в стену, вы не сможете заменить бетонный класс другим без проведения «хирургической операции» над вашим классом (и, следовательно, нарушения принципа открытости/закрытости). Таким образом, единственное, что ваш код должен знать о системе, — это интерфейс. Реализация должна быть не важна для вашего кода.

Вы, вероятно, много слышали о связи между ослаблением связей (decoupling) и написанием чистого кода. Именно в этом и заключается суть DIP — в том, чтобы сделать код максимально слабо связанным. Ваши классы никогда не должны работать вместе как единое целое. Но, как мы обсуждали выше, вы хотите, чтобы ваш код был связан максимально тонко — через интерфейсы.

Давайте рассмотрим пример. Вот код, который должен быть вам очень знаком — он зависит от конкретных экземпляров для выполнения основной работы. Я вижу много кода на Delphi, который выглядит именно так.

```pascal
unit uCoupled;

interface

type
  TCompressionService = class
  private
    function DoCompression(aByteArray: TArray<Byte>): TArray<Byte>;
  public
    procedure Compress(aSourceFilename: string; aTargetFilename: string);
  end;

implementation

uses
  System.Classes, System.SysUtils;

procedure TCompressionService.Compress(aSourceFilename: string; aTargetFilename: string);
var
  LContent, CompressedContent: TArray<Byte>;
  InFS, OutFS: TFileStream;
begin
  // Чтение содержимого
  InFS := TFileStream.Create(aSourceFilename, fmOpenRead);
  try
    SetLength(LContent, InFS.Size);
    InFS.Read(LContent, InFS.Size);
  finally
    InFS.Free;
  end;

  // Сжатие
  CompressedContent := DoCompression(LContent);

  // Запись сжатого содержимого
  OutFS := TFileStream.Create(aTargetFilename, fmCreate);
  try
    OutFS.Write(CompressedContent, Length(CompressedContent));
  finally
    OutFS.Free;
  end;
end;

function TCompressionService.DoCompression(aByteArray: TArray<Byte>): TArray<Byte>;
begin
  // Здесь происходит сжатие файла...
  Result := aByteArray;
end;
```

Перед нами код, где модуль высокого уровня, `TCompressionService`, зависит от модуля низкого уровня, `TFileStream`. Это не то, как всё должно работать. Здесь мы привязаны к `TFileStream` и вынуждены проводить радикальную «хирургию» класса, если захотим изменить способ обработки файлов. А что если кому-то понадобится сжать данные в памяти или строку? Мы нарушаем DIP, потому что не зависим от абстракции, и заставляем модуль высокого уровня зависеть от модуля низкого уровня. Метод `Compress` фактически создает два жестко закодированных экземпляра `TFileStream`. `TFileStream` — это деталь, а мы не должны зависеть от деталей; вместо этого мы должны абстрагироваться от них.

**Так давайте инвертируем вещи:**

```pascal
unit uDecoupled;

interface

type
  IReader = interface
    function ReadAll: TArray<Byte>;
  end;

  IWriter = interface
    procedure WriteAll(aBytes: TArray<Byte>);
  end;

  TCompressionService = class
  private
    function DoCompression(aByteArray: TArray<Byte>): TArray<Byte>;
  public
    procedure Compress(aReader: IReader; aWriter: IWriter);
  end;

implementation

procedure TCompressionService.Compress(aReader: IReader; aWriter: IWriter);
var
  LContent: TArray<Byte>;
  LCompressedContent: TArray<Byte>;
begin
  // Чтение контента
  LContent := aReader.ReadAll;
  // Сжатие
  LCompressedContent := DoCompression(LContent);
  // Запись контента
  aWriter.WriteAll(LCompressedContent);
end;

function TCompressionService.DoCompression(aByteArray: TArray<Byte>): TArray<Byte>;
begin
  // Алгоритм сжатия будет здесь, желательно через абстракцию!
  Result := aByteArray;
end;
```

Здесь мы полностью инвертировали зависимости. Теперь класс `TCompressionService` не зависит от деталей, а зависит от двух абстракций: `IReader` и `IWriter`. Больше нет никаких жестко закодированных зависимостей. Вместо этого мы вынесли зависимость — или детали, если хотите — в низкоуровневые модули, скрытые за интерфейсами.

Для полноты картины приведем пример того, как могут быть реализованы эти детали. Важно отметить, что вы можете реализовать `IReader` и `IWriter` любым удобным для вас способом — это детали, которые не касаются модуля высокого уровня `TCompressionService`.

```pascal
unit uFileReaderWriter;

interface

uses
  uDecoupled;

type
  TFileReader = class(TInterfacedObject, IReader)
  private
    FFileName: string;
  public
    constructor Create(aFilename: string);
    function ReadAll: TArray<Byte>;
  end;

  TFileWriter = class(TInterfacedObject, IWriter)
  private
    FFileName: string;
  public
    constructor Create(aFilename: string);
    procedure WriteAll(aBytes: TArray<Byte>);
  end;

implementation

uses
  System.Classes, System.SysUtils;

constructor TFileReader.Create(aFilename: string);
begin
  inherited Create;
  FFileName := aFilename;
end;

function TFileReader.ReadAll: TArray<Byte>;
var
  FS: TFileStream;
begin
  FS := TFileStream.Create(FFileName, fmOpenRead);
  try
    SetLength(Result, FS.Size);
    FS.Read(Result, FS.Size);
  finally
    FS.Free;
  end;
end;

constructor TFileWriter.Create(aFilename: string);
begin
  inherited Create;
  FFileName := aFilename;
end;

procedure TFileWriter.WriteAll(aBytes: TArray<Byte>);
var
  FS: TFileStream;
begin
  FS := TFileStream.Create(FFileName, fmCreate);
  try
    FS.Write(aBytes, Length(aBytes));
  finally
    FS.Free;
  end;
end;
```

Таким образом, DIP становится основой для любого ослабления связей. В приведенном выше коде вы можете изменить реализацию `IReader` и `IWriter`, не затрагивая `TCompressionService` (и тем самым следуя принципу открытости/закрытости!). Вы успешно заставили все модули — особенно детали — зависеть от абстракций (т.е. интерфейсов). В этом и заключается суть принципа инверсии зависимостей.

**Заключение**

Написание SOLID кода не так уж и сложно. Оно просто требует, чтобы вы посмотрели на свой код под другим углом. Следуйте принципам SOLID, и вы в конечном итоге получите чистый, хорошо написанный код, который легко поддерживать. То, что у вас появится больше классов и интерфейсов, — это побочный эффект без реальных минусов. Маленькие дискретные классы легко тестировать, легко использовать повторно и легко объединять в мощные системы. Кто этого не хочет?
