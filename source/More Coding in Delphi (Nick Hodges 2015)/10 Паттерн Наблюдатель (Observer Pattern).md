### **Введение**

Следующий паттерн, который мы рассмотрим, — это паттерн Наблюдатель. Его следует использовать, когда один объект должен уведомлять любое количество других объектов о возникновении определенных событий. Паттерн Наблюдатель создает отношение «один ко многим» между объектом, генерирующим событие, и объектами, которые должны быть уведомлены об этом событии.

Паттерн Наблюдатель — отличный способ разделить ваши сферы ответственности. Во-первых, у вас есть задача производства данных. Производство данных бесполезно, если их нельзя как-то использовать или отобразить. Однако может существовать любое количество способов сохранения, отображения или иного распространения этих данных. Таким образом, «Субъект» (Subject) — объект, за которым ведется наблюдение, — отслеживает всех «Наблюдателей» (Observers), которым нужны данные. Наблюдатели должны быть отделены от субъекта через интерфейсы.

Хороший способ понять паттерн Наблюдатель — вспомнить о подписке на журналы. Вы можете подписаться на новостной журнал, который доставляется вам еженедельно. Этот журнал является «Субъектом», а вы — «Наблюдателем». Вы читаете (наблюдаете) журнал каждую неделю по мере его доставки. Вы можете прекратить подписку в любое время. Но вы не единственный подписчик — другие люди, стоматологические клиники, предприятия и т. д. могут подписаться или прекратить подписку в любое время. Издатель журнала почти ничего не знает о том, куда уходят его журналы, кроме адреса, а вы мало знаете о том, что требуется для создания журнала — вы просто знаете, что он появляется каждую неделю.

Паттерн Наблюдатель работает именно так: наблюдатели могут подписываться на субъекта, а субъект может уведомлять подписчиков о «новостях».

### **Бейсбольные данные**

Возьмем спортивный пример — бейсбольный матч. В бейсбольных матчах традиционно отслеживаются пробежки (runs) в каждой половине каждого иннинга, а также счет, хиты (hits) и ошибки (errors) для каждой команды. Эти статистические данные обновляются по ходу игры, отображаются и передаются повсюду в режиме реального времени: на веб-сайты, на табло стадиона, в спортивные телепередачи и т. д..

Идея здесь заключается в том, что бейсбольный матч является Субъектом — это объект, за которым наблюдают и который выполняет уведомление. Различные дисплеи для статистики игры являются наблюдателями — они ждут новостей о том, что происходит во время бейсбольного матча, и обновляются на основе уведомлений, которые они получают от игры. Другой взгляд на это состоит в том, что бейсбольный матч является издателем информации, а дисплеи — подписчиками.

Формальное определение паттерна Наблюдатель из книги «Банды четырех» гласит: «Паттерн Наблюдатель определяет зависимость "один ко многим" между объектами таким образом, что при изменении состояния одного объекта все его зависимые объекты уведомляются и обновляются автоматически».

Однако, когда приходит время для реализации, велик соблазн просто встроить дисплеи внутрь метода самой бейсбольной игры. Вы могли бы создать классы для каждого представления, инициализировать их в конструкторе бейсбольного матча, а затем просто обновлять представления в методе с названием `GameChanged`.

Конечно, если вы сделаете это, вы нарушите некоторые базовые правила проектирования. Во-первых, вы будете программировать для реализации, а не для интерфейса. Во-вторых, вам будет очень сложно добавить новый дисплей данных, если это станет необходимым. Дисплеи жестко закодированы в бейсбольном матче и не могут быть добавлены или удалены во время выполнения. (Обратите внимание, что такой способ действий нарушил бы принцип открытости/закрытости, потому что класс бейсбольного матча пришлось бы менять для добавления дисплея). Если мы захотим добавить больше дисплеев, нам придется модифицировать саму игру. Или, говоря более кратко, бейсбольный матч и его дисплеи будут тесно связаны друг с другом. А если вы знаете обо мне хоть что-то, я презираю тесную связанность. Вы тоже должны.

В любом случае, есть лучший способ — паттерн Наблюдатель. Как отмечалось выше, паттерн Наблюдатель состоит из Субъекта, за которым следят Наблюдатели. Мы реализуем очень простую версию этого паттерна. Первое, что мы сделаем, конечно, это объявим два интерфейса:

```pascal
type
  IObserver = interface
    ['{69B40B25-B2C8-4F11-B442-39B7DC26FE80}']
    procedure Update(aGameInfo: TGameInfo);
  end;

  ISubject = interface
    ['{A9240295-B0C2-441D-BD43-932AF735832A}']
    procedure RegisterObserver(aObserver: IObserver);
    procedure RemoveObserver(aObserver: IObserver);
    procedure NotifyObservers;
  end;
```


Первый интерфейс — `IObserver`, который будут реализовывать классы-наблюдатели. В нем они будут получать обновления со всей бейсбольной статистикой, которую предоставит матч. Второй — `ISubject`, который будет реализован бейсбольным матчем. Он определяет три метода: два для обработки подключения и отключения `IObservers` и третий для фактического уведомления наблюдателей.

Но сначала нам нужно объявить несколько структур данных, которые мы будем использовать для отслеживания игры.

```pascal
type
  TInningHalf = (Top, Bottom);
  TTeam = (Home, Away);

  TInning = record
    Top: integer; // будет содержать счет для этой половины иннинга
    Bottom: integer;
    constructor Create(aTop, aBottom: integer);
  end;

  TRuns = record
    Home: integer;
    Away: integer;
    constructor Create(aHome, aAway: integer);
  end;

  THits = record
    Home: integer;
    Away: integer;
    constructor Create(aHome, aAway: integer);
  end;
  
  TErrors = record
    Home: integer;
    Away: integer;
    constructor Create(aHome, aAway: integer);
  end;

  TInningNumber = 1..9; // В бейсбольном матче обычно девять иннингов

  TGameInfo = record
    Innings: array[TInningNumber] of TInning;
    Runs: TRuns;
    Hits: THits;
    Errors: TErrors;
    procedure SetInning(aInningNumber: integer; aInningHalf: TInningHalf; aValue: integer);
  end;
```


Основные данные для бейсбольного матча включают счет для каждой половины иннинга, а также общее количество пробежек, хитов и ошибок, произошедших во время игры. Они агрегированы в запись под названием `TGameInfo` для более легкого доступа.

Далее мы рассмотрим Субъекта — объект, за которым наблюдают. Обратите внимание, что он реализует `ISubject` и, таким образом, содержит код для отслеживания любого количества наблюдателей во внутреннем списке.

```pascal
type
  TBaseballGame = class(TInterfacedObject, ISubject)
  private
    FHomeTeam: string;
    FAwayTeam: string;
    FGameInfo: TGameInfo;
    FObserverList: TList<IObserver>;
    procedure NotifyObservers;
  public
    constructor Create(aHomeTeam: string; aAwayTeam: string);
    destructor Destroy; override;
    procedure RegisterObserver(aObserver: IObserver);
    procedure RemoveObserver(aObserver: IObserver);
    procedure SetInning(aInningNumber: TInningNumber; aInning: TInning);
    procedure SetRuns(aRuns: TRuns);
    procedure SetHits(aHits: THits);
    procedure SetErrors(aErrors: TErrors);
    procedure GameChanged;
    property GameInfo: TGameInfo read FGameInfo write FGameInfo;
    property HomeTeam: string read FHomeTeam;
    property AwayTeam: string read FAwayTeam;
  end;
```


Механика управления реализацией `TBaseballGame` вполне ожидаема. Интересная часть — это реализация `ISubject`: внутри она использует `TList<IObserver>` для управления всеми зарегистрированными наблюдателями. Методы `RegisterObserver` и `RemoveObserver` просто вставляют и удаляют соответственно экземпляры классов, реализующих интерфейс `IObserver`.

Настоящее действие происходит в методе `NotifyObservers`. Мы вернемся к этому через секунду. Сначала давайте взглянем на наблюдателей; они будут реализовывать различные способы отображения бейсбольного счета. Поскольку все дисплеи очень похожи, это хорошее время использовать старое доброе наследование для объявления базового класса, реализующего необходимую функциональность, от которого будут наследоваться классы, выполняющие работу по предоставлению конкретных дисплеев. Вот интерфейс для `TBaseballGameDisplay`:

```pascal
  TBaseballGameDisplay = class(TInterfacedObject, IObserver, IDisplay)
  private
    FSubject: TBaseballGame;
  public
    constructor Create(aBaseballGame: TBaseballGame);
    procedure Update(aGameInfo: TGameInfo); virtual;
    procedure Display; virtual; abstract;
  end;
```

Реализуемый им интерфейс `IDisplay` находится здесь:

```pascal
  IDisplay = interface
    ['{1517E56B-DDB3-4E04-AF1A-C70CF16293B2}']
    procedure Display;
  end;
```


`TBaseballGameDisplay` имеет внутреннее поле для отслеживания бейсбольного матча, за которым он наблюдает. Его потомки могут делать с содержащимися в нем данными все, что им угодно. Класс также реализует интерфейс `IDisplay`, чтобы он мог сообщать то, что ему нужно. Метод `Update` позволит Субъекту — в данном случае бейсбольному матчу — обновлять все дисплеи, наблюдающие за игрой. `Update`, кстати, является виртуальным методом, чтобы потомки могли по-разному обрабатывать поступающую информацию.

Кроме того, вот конструктор и деструктор для `TBaseballGameDisplay`:

```pascal
constructor TBaseballGameDisplay.Create(aBaseballGame: TBaseballGame);
begin
  inherited Create;
  if aBaseballGame = nil then
  begin
    raise Exception.Create('You cannot pass a nil baseball game');
  end;
  FSubject := aBaseballGame;
  FSubject.RegisterObserver(Self);
end;

destructor TBaseballGameDisplay.Destroy;
begin
  FSubject.RemoveObserver(Self);
  inherited;
end;
```


Обратите внимание, что он принимает `TBaseballGame` в качестве параметра и регистрирует себя в этом объекте через вызов `RegisterObserver`.

Итак, заглянем «под капот». Бейсбольный матч может принимать и удалять реализации `IObserver` во время выполнения. Он реализует метод `NotifyObservers` как часть интерфейса `ISubject`. Он вызывается всякий раз, когда информация о бейсбольном матче меняется. Он реализован следующим образом:

```pascal
procedure TBaseballGame.NotifyObservers;
var
  Observer: IObserver;
begin
  for Observer in FObserverList do
  begin
    Observer.Update(GameInfo);
  end;
end;
```

Это довольно просто — он просто перебирает каждый элемент в списке наблюдателей и вызывает метод `Update` на каждом дисплее, передавая новые данные `GameInfo`. Базовый класс сохраняет информацию, а его потомки ее обрабатывают.

Настоящая «работа» выполняется, когда бейсбол обновляет данные иннинга и вызывает `NotifyObservers`:

```pascal
procedure TBaseballGame.SetInning(aInningNumber: TInningNumber; aInning: TInning);
begin
  FGameInfo.Innings[aInningNumber].Top := aInning.Top;
  FGameInfo.Innings[aInningNumber].Bottom := aInning.Bottom;
  GameChanged; // просто вызывает NotifyObservers
end;
```

А что насчет дисплеев? Я определил два и намекнул на третий. Первый выводит традиционный бейсбольный счет по иннингам:

```pascal
procedure TFullGameDisplay.Display;
var
  i: Integer;
begin
  WriteLn('1 2 3 4 5 6 7 8 9 R H E');
  WriteLn('------------------------');
  for i := Low(TInningNumber) to High(TInningNumber) do
  begin
    Write(FSubject.GameInfo.Innings[i].Top, ' ');
  end;
  WriteLn(' ', FSubject.GameInfo.Runs.Away, ' ', FSubject.GameInfo.Hits.Away, ' ',
    FSubject.GameInfo.Errors.Away);
  for i := Low(TInningNumber) to High(TInningNumber) do
  begin
    Write(FSubject.GameInfo.Innings[i].Bottom, ' ');
  end;
  WriteLn(' ', FSubject.GameInfo.Runs.Home, ' ', FSubject.GameInfo.Hits.Home, ' ',
    FSubject.GameInfo.Errors.Home);
end;
```

Следующий дисплей просто выводит счет игры:

```pascal
procedure TGameScore.Display;
begin
  WriteLn;
  WriteLn(FSubject.AwayTeam, ': ', FSubject.GameInfo.Runs.Away);
  WriteLn(FSubject.HomeTeam, ': ', FSubject.GameInfo.Runs.Home);
  WriteLn;
end;
```

Третий, `THTMLOutput`, если бы я был по-настоящему амбициозен, предоставлял бы HTML-вывод бейсбольных данных. Но я не настолько амбициозен и привожу его здесь лишь в качестве иллюстрации того, что можно сделать с бейсбольными данными.

Все это объединяется в методе для создания бейсбольного матча и добавления наблюдателей:

```pascal
procedure DoBaseballGame;
var
  BaseballGame: TBaseballGame;
  FullDisplay: IDisplay;
  ScoreDisplay: IDisplay;
  HTMLDisplay: IDisplay;
begin
  BaseballGame := TBaseballGame.Create('Bisons', 'Gazelles');
  try
    FullDisplay := TFullGameDisplay.Create(BaseballGame);
    ScoreDisplay := TGameScore.Create(BaseballGame);
    HTMLDisplay := THTMLOutput.Create(BaseballGame);
    // ... обновление данных
  finally
    BaseballGame.Free;
  end;
end;
```

Вот несколько интересных моментов, на которые стоит обратить внимание:

* Этот код ничего не пишет в консоль. Все это делается объектами отображения, которые регистрируются как наблюдатели. Для бейсбольного матча они являются просто интерфейсами `IObserver`, и единственное, что вы можете сделать с `IObserver` — это вызвать его метод `Update`. С точки зрения бейсбольного матча, больше в этом ничего нет.
* Экран обновляется каждый раз, когда обновляется информация об иннинге. В демонстрационных целях я просто обновляю игру при изменении информации об иннинге.
* `TBaseballGame` ничего не знает о дисплеях игры — он знает только об `IObserver`. Это означает, что у вас могут быть наблюдатели, выполняющие другие задачи, такие как потоковая передача результатов по сети или сохранение в базу данных.
* Мы можем менять наблюдателей в любое время без изменения Субъекта. Они очень слабо связаны.

Наш интерфейс `IObserver` очень специфичен для бейсбольных данных. Это подходящее место для использования дженериков, и на самом деле фреймворк Delphi Spring реализует универсальный интерфейс `IObservable<T>` и соответствующий класс реализации.

#### **Обобщенный Наблюдатель с Spring4D (Generic Observer with Spring4D)**

Создание приложения с использованием `IObservable<T>` из Spring4D довольно просто. Сначала мы создадим новый `TBaseballGame`. Этот вариант чуть проще:

```pascal
type
  TBaseballGame = class(TObservable<IDisplay>)
  private
    FGameInfo: TGameInfo;
  protected
    procedure DoNotify(const observer: IDisplay); override;
  public
    procedure SetInning(aInningNumber: TInningNumber; aInning: TInning);
    procedure SetRuns(aRuns: TRuns);
    procedure SetHits(aHits: THits);
    procedure SetErrors(aErrors: TErrors);
    property GameInfo: TGameInfo read FGameInfo;
  end;
```

`TBaseballGame` наследуется от `TObservable<IDisplay>`. Этот класс имеет абстрактный метод `DoNotify`, который необходимо переопределить. `TBaseballGame` делает это и, таким образом, обеспечивает способ уведомления наблюдателей класса о том, что что-то изменилось:

```pascal
procedure TBaseballGame.DoNotify(const observer: IDisplay);
begin
  observer.Update(Self.GameInfo);
end;
```

Мы сообщим нашим наблюдателям о том, что в игре произошло обновление, в методе `SetInning`:

```pascal
procedure TBaseballGame.SetInning(aInningNumber: TInningNumber; aInning: TInning);
begin
  FGameInfo.Innings[aInningNumber].Top := aInning.Top;
  FGameInfo.Innings[aInningNumber].Bottom := aInning.Bottom;
  Notify;
end;
```

Этот класс бейсбольного матча во многом похож на класс из предыдущего примера, за исключением того, что в нем нет ни одного из методов `ISubject`. Вместо этого он наследуется от `TObservable<T>`, который выглядит следующим образом:

```pascal
IObservable<T> = interface
  procedure Attach(const observer: T);
  procedure Detach(const observer: T);
  procedure Notify;
end;

TObservable<T> = class(TInterfacedObject, IObservable<T>)
  private
    fLock: TMREWSync;
    fObservers: IList<T>;
  protected
    procedure DoNotify(const observer: T); virtual; abstract;
    property Observers: IList<T> read fObservers;
  public
    constructor Create;
    destructor Destroy; override;
    procedure Attach(const observer: T);
    procedure Detach(const observer: T);
    procedure Notify;
end;
```

Вы должны видеть сходство между `IObservable<T>`, `TObservable<T>` и первым демонстрационным приложением. Оба они отслеживают наблюдателей (слушателей) и оба имеют Субъектов (`TObservable`).

Мы просто объявим одного наблюдателя, `TConsoleDisplay`, который реализует `IDisplay` — интерфейс, который мы передадим в качестве параметризованного типа.

```pascal
type
  IDisplay = interface
    ['{E118BD99-37BD-461C-AF69-770FD8E18702}']
    procedure Update(aGameInfo: TGameInfo);
  end;

  TConsoleDisplay = class(TInterfacedObject, IDisplay)
    procedure Update(aGameInfo: TGameInfo);
  end;
```

Основная часть приложения практически такая же, как и в предыдущем примере:

```pascal
var
  BaseballGame: TBaseballGame;
begin
  BaseballGame := TBaseballGame.Create;
  BaseballGame.Attach(TConsoleDisplay.Create);

  BaseballGame.SetRuns(TRuns.Create(1, 0));
  BaseballGame.SetHits(THits.Create(2, 0));
  BaseballGame.SetInning(1, TInning.Create(0, 1));
  ReadLn;
  // ... больше данных
end.
```

Вот пара моментов, на которые стоит обратить внимание:

* Поскольку используются преимущества обобщенной инфраструктуры Spring4D, объем кода для включения паттерна Наблюдатель значительно сокращается.
* `TBaseballGame` является субъектом. Он принимает слушателей и уведомляет их при изменении счета.
* Метод `Notify` итерирует всех наблюдателей и вызывает их метод `DoNotify`.

#### **Заключение**

Подводя итог, можно сказать, что паттерн Наблюдатель гарантирует, что классы-издатели (субъекты) могут передавать обновления своим классам-подписчикам (наблюдателям) с использованием очень гибкой и слабой связанности. Наблюдатели могут обновляться и удаляться во время выполнения. Добавление наблюдателей не требует изменений в субъектах, и наблюдатели могут быть созданы и добавлены в любое время. Субъекты мало знают о том, чем заняты наблюдатели. Все это очень просто, элегантно и слабо связано — именно так, как вам нужно.