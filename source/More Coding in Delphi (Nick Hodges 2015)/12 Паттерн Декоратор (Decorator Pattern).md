
Одной из тем этой книги является стремление сделать ваш код как можно более гибким во время выполнения и проектировать его так, чтобы отдавать предпочтение гибкости во время выполнения, а не во время компиляции. Каждый раз, когда вы определяете что-то во время компиляции, вы жестко закрепляете эту функцию в своем коде. Ее нельзя изменить во время выполнения.

Наследование является столпом ООП, но это конструкция времени компиляции. Как мы видели ранее в главе «Шесть мыслей», наследование мощно, но негибко во время выполнения. У вас есть те классы, которые есть, и их нельзя переделать или изменить после того, как вы закрепили их в камне скомпилированного кода. Если вы пытаетесь придумать различные комбинации для определения каждого унаследованного класса, вы можете прийти к огромному количеству производных классов, которыми станет трудно управлять и поддерживать. Изменения в одной из функций означают, что любое количество потомков также может потребовать изменений. Мы говорили о том, что вам следует предпочитать композицию наследованию.

Паттерн декоратор — это способ обойти эти ограничения и обеспечить мощь наследования, обеспечивая при этом гибкость во время выполнения. Это может быть иной способ добавления новых поведений во время выполнения, отличный от того, что мы видели в главе «Шесть мыслей». На самом деле он использует простое наследование, чтобы «обернуть» данный класс и предоставить дополнительную функциональность без необходимости изменения исходного класса.

Вот формальное определение из Википедии: _Паттерн Декоратор динамически наделяет объект дополнительными обязанностями. Декораторы предоставляют гибкую альтернативу субклассированию для расширения функциональности_.

На самом деле все довольно просто. Вот что нужно сделать:

Во-первых, вы начинаете с концепции класса, который необходимо декорировать. Он становится «базовым» классом. Мы воспользуемся примером пиццы. Мы создадим всего две: одну с сыром Пармезан и одну с сыром Моцарелла (предположим, что во всех пиццах есть томатный соус). Каждая будет украшена различными топпингами, такими как колбаса, пепперони, черные оливки и лук.

Затем вы создаете одного потомка, который будет базовым классом декоратора. Этот класс будет принимать в качестве свойства экземпляр исходного базового класса. Базовый декоратор должен иметь конструктор, принимающий исходный базовый класс в качестве параметра.

Затем вы создаете конкретные экземпляры декораторов, которые переопределяют необходимые методы и свойства базового класса декоратора.

Отсюда вы можете начать с базового класса, а затем объединить в цепочку столько декораторов, сколько захотите.

Итак, давайте посмотрим на это в коде. Вот базовый класс, `TPizza`:

``` pascal
type
  TPizza = class
  private
    FDescription: string;
  protected
    function GetDescription: string; virtual;
    procedure SetDescription(const Value: string);
  public
    function Cost: Currency; virtual; abstract;
    property Description: string read GetDescription write SetDescription;
  end;
```

Базовый класс является абстрактным. У него есть два свойства — ну, фактически одно — при этом `Cost` является виртуальной абстрактной функцией.

`TPizza` — это базовый класс для начала процесса обертывания. Мы создаем потомков, которые сформируют базовый класс, описанный выше. Эти классы переопределяют метод `Cost`, предоставляя свои собственные цены. Они будут отправной точкой для декорирования. В нашем случае мы создадим конкретные базовые типы пиццы, которые можно оборачивать:

```pascal
TParmesanCheesePizza = class(TPizza)
  constructor Create;
  function Cost: Currency; override;
end;

TMozarellaCheesePizza = class(TPizza)
  constructor Create;
  function Cost: Currency; override;
end;
```

Интересная часть находится в базовом классе декоратора:

```pascal
TPizzaDecorator = class(TPizza)
private
  FPizza: TPizza;
public
  constructor Create(aPizza: TPizza);
  destructor Destroy; override;
end;
```

`TPizzaDecorator` делает три вещи. Во-первых, он наследуется от `TPizza`. Во-вторых, он получает другой экземпляр `TPizza` в своем конструкторе, чтобы «обернуть» себя вокруг начального базового класса. И в-третьих, деструктор гарантирует, что внутренне сохраненный экземпляр будет правильно освобожден, когда придет время.

Поскольку `TPizzaDecorator` сам является `TPizza`, вы можете продолжать оборачивать предыдущий результат в новый, декорируя декоратор, так сказать. Кроме того, он будет иметь точно такой же интерфейс, как и базовый класс пиццы, поэтому он может вести себя как пицца, потому что, собственно, ею и является.

Что касается классов, которые выполняют декорирование, они будут наследоваться от `TPizzaDecorator`:

```pascal
TPepperoni = class(TPizzaDecorator)
protected
  function GetDescription: string; override;
public
  function Cost: Currency; override;
end;

TSausage = class(TPizzaDecorator)
protected
  function GetDescription: string; override;
public
  function Cost: Currency; override;
end;

TBlackOlives = class(TPizzaDecorator)
protected
  function GetDescription: string; override;
public
  function Cost: Currency; override;
end;

TOnions = class(TPizzaDecorator)
protected
  function GetDescription: string; override;
public
  function Cost: Currency; override;
end;
```

Каждый из этих классов делает две вещи.

Во-первых, они переопределяют геттер для свойства `Description`. Они делают это интересным способом. Поскольку они декорируют базовую пиццу, они предполагают, что они являются «аддитивными», то есть они будут добавляться к базовому описанию. Метод `TPepperoni.GetDescription` выглядит следующим образом:

```pascal
function TPepperoni.GetDescription: string;
begin
  Result := FPizza.GetDescription + ', Pepperoni';
end;
```

Этот метод берет описание из класса, который он оборачивает, и добавляет запятую и свое собственное описание.

Во-вторых, метод `Cost` переопределяется для добавления стоимости к цене класса, который он декорирует.

```pascal
function TPepperoni.Cost: Currency;
begin
  Result := FPizza.Cost + 1.20;
end;
```

Итак, теперь у нас есть куча типов пиццы — базовые классы — и куча начинок, чтобы обернуть эти классы и их самих. Давайте соберем все это вместе:

Сначала мы создадим процедуру для вывода типа пиццы в консоль:

```pascal
procedure OutputPizza(aPizza: TPizza);
begin
  WriteLn('A ', aPizza.Description, ' costs ', '$', Format('%2f', [aPizza.Cost]));
  WriteLn;
end;
```

Затем мы создадим пиццу Моцарелла и обернем ее Колбасой и Пепперони:

```pascal
Pizza := TMozarellaCheesePizza.Create;
try
  Pizza := TPepperoni.Create(Pizza);
  Pizza := TSausage.Create(Pizza);
  OutputPizza(Pizza);
finally
  Pizza.Free;
end;
```

Это сформирует описание и рассчитает правильную цену, и приведет к:

![ScrinShot](image_page069.png)
*(Изображение со страницы 69 / image_page069.png)*

Еще один способ создания пиццы — просто вкладывать каждый декоратор в конструктор предыдущего. Этот метод фактически обеспечивает программное представление того, как каждый декоратор оборачивает предыдущий:

```pascal
Pizza := TBlackOlives.Create(
                  TPepperoni.Create(
                  TPepperoni.Create(TParmesanCheesePizza.Create)));
try
  OutputPizza(Pizza);
finally
  Pizza.Free;
end;
```

> Вы должны заметить, что использование паттерна Декоратор выше (и того, который мы рассмотрим ниже) иллюстрирует Принцип открытости/закрытости (Open/Closed Principle) — `TPizza` остается закрытым, однако мы использовали паттерн декоратор, чтобы сделать его открытым для расширения. `TPizza` даже не знает, что его декорируют. Это также помогает соблюдать Принцип единственной ответственности (Single Responsibility Principle), гарантируя, что `TPizza` остается просто Пиццей и не делает ничего сверх этого.

### **Декоратор и интерфейсы (Decorator and Interfaces) **

Предыдущий пример использовал чистое наследование, чтобы проиллюстрировать один из способов реализации паттерна Декоратор. Но вы знаете меня, и знаете, что я одержим программированием под интерфейс (кстати, вам тоже следовало бы...). Так как насчет реализации паттерна Декоратор с использованием интерфейсов вместо простого наследования классов? Я думал, вам понравится эта идея.

##### **Что-то, что можно декорировать (Something to Decorate) **

Сначала мы объявим простой интерфейс, который позже сможем декорировать другой функциональностью. Мы начнем с простого интерфейса процессора заказов:

```pascal
IOrderProcessor = interface 
['{193966C6-BD40-487F-8A2F-E36BC707CA7D}']
  procedure ProcessOrder(aOrder: TOrder);
end;
```

Затем не менее простая реализация:

```pascal
TOrderProcessor = class(TInterfacedObject, IOrderProcessor)
  procedure ProcessOrder(aOrder: TOrder);
end;

procedure TOrderProcessor.ProcessOrder(aOrder: TOrder);
begin
  WriteLn('Processed order for Order #', aOrder.ID);
end;
```

Вот объявление для `TOrder`. Оно очень простое и на самом деле лишь косвенно относится к тому, что мы делаем:

```pascal
type
  TOrder = class
  private
    FID: integer;
  public
    constructor Create(aID: integer);
    property ID: integer read FID;
  end;

constructor TOrder.Create(aID: integer);
begin
  inherited Create;
  FID := aID;
end;
```

Теперь у нас есть базовый класс, который мы можем декорировать с помощью интерфейсов.

Хорошо, теперь мы хотим декорировать `IOrderProcessor` — или, точнее, декорировать его реализацию. Начнем с объявления нового класса, который также реализует `IOrderProcessor`:

```pascal
TOrderProcessorDecorator = class abstract(TInterfacedObject, IOrderProcessor)
private
  FInnerOrderProcessor: IOrderProcessor;
public
  constructor Create(aOrderProcessor: IOrderProcessor);
  procedure ProcessOrder(aOrder: TOrder); virtual; abstract;
end;
```

Конструктор реализован следующим образом:

```pascal
constructor TOrderProcessorDecorator.Create(aOrderProcessor: IOrderProcessor);
begin
  inherited Create;
  FInnerOrderProcessor := aOrderProcessor;
end;
```

Класс `TOrderProcessorDecorator` будет базовым классом для всего декорирования `IOrderProcessor`. Он также является абстрактным классом, требующим реализации метода `ProcessOrder` в производных классах. `ProcessOrder` также является единственным методом интерфейса `IOrderProcessor`. Совпадение? Я так не думаю. Его цель — быть своего рода «шлюзом» для вставки других интерфейсов в реализации `IOrderProcessor` и расширения — если хотите, «декорирования» — функциональности интерфейса.

Заметьте, что конструктор сохраняет то, что называется `FInnerProcessor`. Это будет наш исходный `IOrderProcessor`, который будет декорирован. Мы увидим его использование через секунду.

Давайте обернем `IOrderProcessor` какой-нибудь функциональностью — как насчет логирования. Мы естественно захотим логировать все, что происходит в процессе заказа, и поэтому мы унаследуемся от `TOrderProcessorDecorator`, вставив в него новый интерфейс и обернув функциональность `IOrderProcessor` новым интерфейсом логирования.

Вот этот интерфейс с простой реализацией:

```pascal
ILogger = interface 
['{955784CD-D867-4B41-A8F2-3D31B4593F39}']
  procedure Log(const aString: string);
end;

TLogger = class(TInterfacedObject, ILogger)
  procedure Log(const aString: string);
end;

procedure TLogger.Log(const aString: string);
begin
  WriteLn(aString);
end;
```

Здесь мы логируем в консоль, но, конечно, ваша реализация логирования может быть любой, какой вы пожелаете. Поскольку мы программируем под интерфейс, на самом деле не имеет значения, как реализовано логирование.

Итак, теперь мы объявляем `TLoggingOrderProcessor` следующим образом:

```pascal
TLoggingOrderProcessor = class(TOrderProcessorDecorator)
private
  FLogger: ILogger;
public
  constructor Create(aOrderProcessor: IOrderProcessor; aLogger: ILogger);
  procedure ProcessOrder(aOrder: TOrder); override;
end;

constructor TLoggingOrderProcessor.Create(aOrderProcessor: IOrderProcessor; aLogger: ILogger);
begin
  inherited Create(aOrderProcessor);
  FLogger := aLogger;
end;

procedure TLoggingOrderProcessor.ProcessOrder(aOrder: TOrder);
begin
  FLogger.Log('Logging: About to process Order #' + aOrder.ID.ToString);
  FInnerOrderProcessor.ProcessOrder(aOrder);
  FLogger.Log('Logging: Finished processing Order #' + aOrder.ID.ToString);
end;
```

Вот несколько вещей, на которые стоит обратить внимание в этом коде:

- Конструктор принимает два параметра. Первый — это `IOrderProcessor`, который мы собираемся декорировать. Второй параметр — это экземпляр реализации `ILogger`, который представляет новую функциональность, которую мы собираемся обернуть вокруг первого параметра. Он сохраняет новую функциональность, `FLogger`, для последующего использования.
- Если вы помните мою предыдущую книгу, это Внедрение зависимостей (Dependency Injection) в действии — конкретно Внедрение через конструктор (Constructor Injection). Видите? Эти вещи действительно пригождаются.
- Класс переопределяет абстрактный метод `ProcessOrder` и оборачивает — декорирует — вызов `FInnerOrderProcessor.ProcessOrder` кодом логирования.
- Декорирующая функциональность обеспечивается полем `FLogger`, которое, конечно же, является интерфейсом. Оно было внедрено в конструкторе, и мы используем его в методе `ProcessOrder` для декорирования внутренней функциональности исходного класса обработки заказов.

Конечно, было бы неинтересно на этом останавливаться. Нам нужно показать, как теперь мы можем обернуть этот обернутый класс еще большей функциональностью. Как насчет того, чтобы добавить в микс немного авторизации? Часто вы захотите защитить определенный метод паролем, давая разрешение на выполнение определенного фрагмента кода только конкретным людям.

Мы объявим интерфейс и простую реализацию:

```pascal
type
  IAuthentication = interface 
  ['{10793CD1-B1C4-4A59-9C5B-A48B64B4AC2A}']
    function Authenticate(const aPassword: string): Boolean;
  end;

TAuthentication = class(TInterfacedObject, IAuthentication)
private
  function Authenticate(const aPassword: string): Boolean;
end;

function TAuthentication.Authenticate(const aPassword: string): Boolean;
var
  S: string;
begin
  Write('Enter the Password: ');
  ReadLn(S);
  Result := S = aPassword;
end;
```

Это очень простая реализация, которая просто запрашивает пароль и возвращает `True`, если он совпадает с переданным. Я бы не стал защищать этим свой банковский счет, но это иллюстрирует суть, верно?

Затем давайте объявим следующий код, который снова оборачивает `IOrderProcessor` новым интерфейсом и применяет к нему функциональность аутентификации.

```pascal
TAuthenticationOrderProcessor = class(TOrderProcessorDecorator)
private
  FAuthentication: IAuthentication;
public
  constructor Create(aOrderProcessor: IOrderProcessor; aAuthentication: IAuthentication);
  procedure ProcessOrder(aOrder: TOrder); override;
end;

constructor TAuthenticationOrderProcessor.Create(aOrderProcessor: IOrderProcessor; aAuthentication: IAuthentication);
begin
  inherited Create(aOrderProcessor);
  FAuthentication := aAuthentication;
end;

procedure TAuthenticationOrderProcessor.ProcessOrder(aOrder: TOrder);
begin
  if FAuthentication.Authenticate('password') then
  begin
    FInnerOrderProcessor.ProcessOrder(aOrder);
  end else begin
    WriteLn('You failed authentication!');
  end;
end;
```

Опять же, вот несколько моментов, на которые стоит обратить внимание в этом коде:

- Конструктор снова принимает два параметра: исходный `IOrderProcessor` и интерфейс к новой функциональности — в данном случае `IAuthentication`.
- Метод `ProcessOrder` делает вызов метода `IAuthentication.Authenticate`, передавая действительно надежный пароль. Если функция возвращает `True`, то внутреннему процессору заказов разрешается делать свое дело. Если нет, вы получаете неприятное сообщение.
- Теперь заметьте, что этот класс, как и предыдущий класс, оба реализуют (через наследование) интерфейс `IOrderProcessor`. Поэтому будет возможно, как мы увидим ниже, оборачивать любой из этих классов вокруг любых других.

Хорошо, давайте попрактикуемся в этом. Вот простая процедура, которую мы вызовем в нашем консольном приложении, показывающая, как все это работает:

```pascal
procedure DoIt;
var
  OrderProcessor: IOrderProcessor;
  Order: TOrder;
begin
  Order := TOrder.Create(42);
  try
    OrderProcessor := TOrderProcessor.Create;
    OrderProcessor := TLoggingOrderProcessor.Create(OrderProcessor, TLogger.Create);
    OrderProcessor.ProcessOrder(Order);

    WriteLn;

    OrderProcessor := TAuthenticationOrderProcessor.Create(OrderProcessor, TAuthentication.Create);
    OrderProcessor.ProcessOrder(Order);
  finally
    Order.Free;
  end;
end;
```

Сначала мы создаем заказ для обработки. Затем мы создаем базовый класс `TOrderProcessor`. После этого мы создаем `TLoggingOrderProcessor`, передавая ему наш существующий экземпляр `TOrderProcessor`, а также экземпляр интерфейса `ILogger`. Затем мы вызываем метод `ProcessOrder`. Наш вывод покажет логирование (см. ниже).

После этого мы передаем обернутую логированием версию `OrderProcessor` реализации аутентификации `IOrderProcessor`, таким образом «дважды оборачивая» исходный процессор заказов. Затем, когда мы вызываем событие `ProcessOrder`, мы сначала получим аутентификацию, а затем функциональность логирования, а также исходную обработку заказа. 

![ScrinShot](image_page075.png)
*(Изображение со страницы 75 / image_page075.png)*

Довольно круто, не так ли?

И снова интерфейсы обеспечивают мощный, но чистый способ реализации разделения ответственности. И снова мы поддержали Принцип единственной ответственности и Принцип открытости/закрытости. Можно даже сказать, что здесь вступил в действие Принцип подстановки Лисков (Liskov Substitution Principle), поскольку мы можем передать любую реализацию логирования и аутентификации декораторам.

Те, кто продвинулся немного дальше, могут заметить, что этот пример паттерна Декоратор фактически выполняет простую форму аспектно-ориентированного программирования (Aspect Oriented Programming), которое мы будем более подробно рассматривать в одной из следующих глав. В начале той главы мы фактически будем делать то же самое, что и здесь.

### **Резюме** 

Итак, подведем итоги:

- Все начинается с базового класса или интерфейса, который будет декорирован. Затем мы можем добавить функциональность к этому базовому классу или интерфейсу без необходимости его изменения. Заметьте, это соответствует Принципу открытости/закрытости.
- При использовании простого наследования классы-декораторы одновременно наследуются от этого базового класса и сохраняют ссылку на него. Это означает, что они будут иметь тот же интерфейс, но ссылка позволит использовать композицию вместо наследования. Помните, как мы говорили об этом в начале книги?
- При использовании интерфейсов мы можем легко передать интерфейс в класс декоратора для расширения функциональности исходного интерфейса.
- Один или несколько декораторов могут быть обернуты вокруг базового класса и друг друга для создания единой сущности. Один или несколько интерфейсов могут быть объединены в цепочку для добавления нескольких декораторов к исходному интерфейсу.
- Вы можете динамически декорировать объекты в любое время, что позволяет создавать любые комбинации декораторов без необходимости создания производного класса для каждого.
- Декораторы позволяют расширять поведение существующих классов без необходимости изменения существующего кода. В нашем примере с наследованием добавление нового ингредиента для пиццы так же просто, как создание нового простого класса, переопределяющего `TPizzaDecorator`. В примере с интерфейсом все, что нам нужно сделать, — это объявить новый интерфейс и передать его в конструктор класса-декоратора.
- Основная идея заключается в том, чтобы упростить и добавить гибкости вашему коду вместо сложной модели наследования. Кроме того, это облегчает создание во время выполнения одного экземпляра сложных комбинаций функций или элементов, о которых вы, возможно, никогда раньше и не задумывались.