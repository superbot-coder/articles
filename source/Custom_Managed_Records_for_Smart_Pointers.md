# Настраиваемые управляемые записи для интеллектуальных указателей

[12.08.2020 Автор: Erik van Bilsen](https://blog.grijjy.com/author/erikvanbilsen)
[Ссылка на оригинал: "Custom Managed Records for Smart Pointers"](https://blog.grijjy.com/2020/08/12/custom-managed-records-for-smart-pointers)

## О смарт-указках

Но сначала давайте разберёмся с терминологией. «Умные указатели» — это общее название различных технологий, упрощающих управление жизненным циклом объекта. В [предыдущей статье](Automate_Restorable_Operations_with_Custom_Managed_Records.md) я показал простую `TAutoFree`CMR (Custom Managed Record), которую можно использовать для автоматического освобождения объекта, когда CMR выходит из области видимости. Это несколько похоже на `std::unique_ptr`тип в C++.

Однако это не очень «умная» CMR: нельзя создать несколько `TAutoFree`записей, ссылающихся на один и тот же объект, поскольку каждая из них уничтожит объект при выходе из области действия, что приведёт к двойному уничтожению. Мы попытались предотвратить это, запретив `TAutoFree`копирование CMR.

В этой статье мы рассмотрим другой тип умных указателей: тот, который можно копировать (или использовать совместно) и который использует автоматический подсчёт ссылок (ARC) для отслеживания количества умных указателей, ссылающихся на объект. Когда счётчик ссылок достигает нуля, ссылки на объект прекращаются, и его можно уничтожить. Это больше похоже на тип `std::shared_ptr`в C++.

Этот тип интеллектуальных указателей можно использовать для автоматизации управления жизненным циклом объектов. До Delphi 10.4 мобильные компиляторы использовали ARC для управления жизненным циклом всех объектов. Хотя мне лично эта функция нравилась, она создавала проблему несовместимости с моделями управления памятью на настольных платформах. В результате вы никогда не могли воспользоваться этой функцией для разработки кроссплатформенного кода. Теперь, когда ARC для объектов больше не существует, вы можете использовать интеллектуальные указатели, чтобы вернуть себе некоторые из её преимуществ. Но на этот раз вы можете использовать её выборочно и только там, где это упрощает жизнь и снижает вероятность утечек памяти.

В этой статье я покажу вам четыре возможных способа создания умного указателя: один с использованием объектных интерфейсов и три с использованием CMR. Код из этой статьи можно найти в нашем [репозитории JustAddCode на GitHub](https://github.com/grijjy/JustAddCode/tree/master/CustomManagedRecords). Но сначала давайте вспомним, что такое Custom Managed Record.

## Индивидуально управляемый обзор записей

В Delphi 10.4 в язык были добавлены пользовательские управляемые записи (Custom Managed Records). В [первой части](Wrapping%20C(%2B%2B)%20APIs%20with%20Custom%20Managed%20Records.md) этой серии я объяснил эту новую функцию и то, как её можно использовать для обёртки сторонних API C(++). Во [второй части](Automate_Restorable_Operations_with_Custom_Managed_Records.md) я показал, как её можно использовать для автоматизации «восстанавливаемых операций», таких как автоматическая установка и снятие блокировки. Вам не нужно читать эти две части, чтобы понять эту публикацию. Поэтому я взял на себя смелость скопировать введение в CMR из второй части. Можете смело пропустить этот раздел, если вы уже знакомы с CMR.

В качестве краткого обзора рассмотрим следующий CMR:

``` pascal
type
  TFoo = record
  public
    class operator Initialize(out ADest: TFoo);
    class operator Finalize(var ADest: TFoo);
    class operator Assign(var ADest: TFoo;
      const [ref] ASrc: TFoo);
  end;
```

Тогда Delphi будет автоматически вызывать  `Initialize` оператор при объявлении такой записи и  `Finalize` при выходе записи из области действия.  `Assign` Оператор вызывается для копирования одной записи в другую. Необязательно использовать все три оператора, но для создания CMR необходим хотя бы один.

Теперь, когда вы пишете следующий код:
``` pascal
begin
  var Foo1: TFoo;
  var Foo2 := Foo1;
end; 
```

Затем Delphi автоматически преобразует это в следующий код (обратите внимание, что этот код не компилируется):

``` pascal
begin
  var Foo1, Foo2: TFoo;
  TFoo.Initialize(Foo1);
  try
    TFoo.Initialize(Foo2);
    try
      TFoo.Assign(Foo2, Foo1);
    finally
      TFoo.Finalize(Foo2);
    end;
  finally
    TFoo.Finalize(Foo1);
  end;
end; 
```

Delphi автоматически вставляет  `try..finally` операторы, чтобы гарантировать, что  `Finalize` оператор всегда будет вызываться. Это важная функция для работы умных указателей.

## 1. Использование объектных интерфейсов

Поскольку интерфейсы уже обеспечивают автоматический подсчёт ссылок, они — очевидный кандидат на использование умных указателей. Простая реализация может выглядеть так:

```pascal
type
  ISmartPtr<T: class> = interface
    function GetRef: T;

    property Ref: T read GetRef;
  end;

type
  TSmartPtr<T: class> = class(TInterfacedObject, ISmartPtr<T>)
  private
    FRef: T;
  protected
    { ISmartPtr<T> }
    function GetRef: T;
  public
    constructor Create(const ARef: T);
    destructor Destroy; override;
  end;

{ TSmartPtr<T> }

constructor TSmartPtr<T>.Create(const ARef: T);
begin
  inherited Create;
  FRef := ARef;
end;

destructor TSmartPtr<T>.Destroy;
begin
  FRef.Free;
  inherited;
end;

function TSmartPtr<T>.GetRef: T;
begin
  Result := FRef;
end;
```

Здесь мы используем универсальный интерфейс для обеспечения типобезопасности и предотвращения приведения типа объекта, на который ссылается объект. Параметр типа `T`имеет ограничение класса, поэтому интеллектуальный указатель можно использовать только с объектами. Свойство `Ref`просто возвращает объект, на который ссылается объект (типа `T`). Интеллектуальный указатель становится владельцем объекта и освобождает его в деструкторе. Копирование интеллектуального указателя безопасно, поскольку подсчёт ссылок гарантирует, что объект не будет уничтожен до тех пор, пока последний ссылающийся на него интеллектуальный указатель не выйдет из области действия.

В следующем примере показано, как этот умный указатель можно использовать для обертывания `TStringList`:
```pascal
var List1: ISmartPtr<TStringList> :=
  TSmartPtr<TStringList>.Create(TStringList.Create);

{ The smart pointer has a reference count of 1.
  Add some strings }
List1.Ref.Add('Foo');
List1.Ref.Add('Bar');

begin
  { Copy the smart pointer.
    It will have a reference count of 2 now. }
  var List2 := List1;

  { Check contents of List2 }
  Assert(List2.Ref[0] = 'Foo');
  Assert(List2.Ref[1] = 'Bar');

  { List2 will go out of scope here, so only List1
    will keep a reference to the TStringList.
    The reference count will be reduced to 1. }
end;

{ Check contents of List1 again }
Assert(List1.Ref.Count = 2);

{ Now List1 will go out of scope, reducing the
  reference count to 0 and destroying the TStringList. }
  ```
  
Важно отметить, что для создания экземпляра умного указателя _нельзя использовать вывод типа. То есть этот код неверен:_
```pascal
var List1 := TSmartPtr<TStringList>.Create(TStringList.Create);
```
Это создаст `List1`тип of `TSmartPtr<>`вместо `ISmartPtr<>`, что приведёт к двум утечкам памяти (для `TSmartPtr<>`самого объекта и для `TStringList`). Вам необходимо явно указать тип:
```pascal
var List1: ISmartPtr<TStringList> :=
  TSmartPtr<TStringList>.Create(TStringList.Create);
```
Это одна из причин, по которой интерфейсы не являются идеальным решением для умных указателей: довольно легко непреднамеренно создать утечку памяти. Вот другие причины, по которым интерфейсы не являются лучшим инструментом для этой задачи:

- Для отслеживания другого объекта требуется выделение отдельного объекта. Это увеличивает использование памяти и повышает вероятность фрагментации памяти.
- Каждый доступ к `Ref`свойству осуществляется через этот `GetRef`метод. Поскольку каждый метод в интерфейсе виртуальный, это означает дополнительный уровень косвенного обращения. Этот дополнительный поиск засоряет кэш, и в сочетании с накладными расходами на вызов метода это может негативно сказаться на производительности.

Объектные интерфейсы — отличный способ отделить спецификацию от реализации и обеспечивают преимущество автоматического управления жизненным циклом. Поэтому их следует использовать для решения многих повседневных задач программирования.

Однако в RTL или сторонних библиотеках есть множество классов, которые не используют интерфейсы и не могут использовать преимущества ARC. Поэтому для автоматизации управления жизненным циклом вам понадобится какой-то умный указатель. Однако я не думаю, что их обёртывание в интерфейс, как показано выше, — хорошее решение. Пользовательские управляемые записи предлагают лучшую альтернативу…

## 2. Использование CMR с базовым классом

Итак, как же здесь может помочь Custom Managed Record? Нам нужен способ ссылаться на объекты счётчика. Возможно, первым порывом будет добавить и объект, и счётчик ссылок в CMR:
```pascal
type
  TSmartPtr<T: class> = record
  private
    FRef: T;
    FRefCount: Integer;
  public
    constructor Create(const ARef: T);
    class operator Initialize(out ADest: TSmartPtr<T>);
    class operator Finalize(var ADest: TSmartPtr<T>);
    class operator Assign(var ADest: TSmartPtr<T>;
      const [ref] ASrc: TSmartPtr<T>);

    property Ref: T read FRef;
  end;
  ```

Затем в конструкторе и операторах `Assign`and `Finalize`вы управляете счётчиком ссылок и освобождаете объект, если он достигает 0. Однако это не сработает, если вы присваиваете одну запись другой. В этом случае у обеих копий будут свои счётчики ссылок. При этом счётчик ссылок одной записи может достичь 0 (и уничтожить ссылаемый объект), в то время как другая запись всё ещё будет иметь ссылку на него.

Поэтому счётчик ссылок должен быть частью объекта. Один из способов сделать это — создать базовый класс, содержащий только счётчик ссылок, чтобы его можно было использовать с умным указателем:

```pascal
type
  TRefCountable = class abstract
  protected
    FRefCount: Integer;
  end;
```  

> Поле `FRefCount`, вероятно, должно иметь `[volatile]`атрибут, но это выходит за рамки данной статьи.

Затем мы можем добавить ограничение универсального типа к умному указателю, чтобы его можно было использовать только с подклассами `TRefCountable`:

```pascal
type
  TSmartPtr<T: TRefCountable> = record
  private
    FRef: T;
    procedure Retain; inline;
    procedure Release; inline;
  public
    constructor Create(const ARef: T);
    class operator Initialize(out ADest: TSmartPtr<T>);
    class operator Finalize(var ADest: TSmartPtr<T>);
    class operator Assign(var ADest: TSmartPtr<T>;
      const [ref] ASrc: TSmartPtr<T>);

    property Ref: T read FRef;
  end;
```

А реализация может выглядеть так:

```pascal
constructor TSmartPtr<T>.Create(const ARef: T);
begin
  Assert( (ARef = nil) or (ARef.FRefCount = 0) );
  FRef := ARef;
  Retain;
end;

class operator TSmartPtr<T>.Initialize(out ADest: TSmartPtr<T>);
begin
  ADest.FRef := nil;
end;

class operator TSmartPtr<T>.Finalize(var ADest: TSmartPtr<T>);
begin
  ADest.Release;
end;

procedure TSmartPtr<T>.Retain;
begin
  if (FRef <> nil) then
    AtomicIncrement(FRef.FRefCount);
end;

procedure TSmartPtr<T>.Release;
begin
  if (FRef <> nil) then
  begin
    if (AtomicDecrement(FRef.FRefCount) = 0) then
      FRef.Free;

    FRef := nil;
  end;
end;

class operator TSmartPtr<T>.Assign(var ADest: TSmartPtr<T>;
  const [ref] ASrc: TSmartPtr<T>);
begin
  if (ADest.FRef <> ASrc.FRef) then
  begin
    ADest.Release;
    ADest.FRef := ASrc.FRef;
    ADest.Retain;
  end;
end;
```

Вот как это работает:

- Оператор `Initialize`просто устанавливает ссылку на `nil`. Это очень распространённый шаблон для CMR.
- Конструктор устанавливает ссылку на объект (полученный от `TRefCountable`) и сохраняет её (подробнее об этом ниже). Мы выполняем проверку корректности, чтобы убедиться, что один и тот же объект не может быть передан конструктору нескольких умных указателей (то есть, счётчик его ссылок должен быть равен 0).
- Оператор `Finalize`просто освобождает ссылку, что может привести к освобождению объекта.
- Вспомогательные методы `Retain`отвечают `Release`за подсчет ссылок:
    - `Retain`Увеличивает счётчик ссылок (если объект не является `nil`). Это происходит атомарно, поэтому умные указатели могут совместно использоваться несколькими потоками.
    - Аналогично, `Release`уменьшает счётчик ссылок. Когда он достигает 0, объект, на который он ссылается, уничтожается. В любом случае, ссылка устанавливается в значение , `nil`поскольку она больше не должна использоваться.
- Оператор `Assign`— самый сложный (относительно). Сначала он проверяет, ссылаются ли уже два умных указателя на один и тот же объект. Если да, то ничего делать не нужно. Если нет, он сначала освобождает целевой умный указатель, так как ему будет назначен новый. Это может привести к уничтожению объекта, если на него нет ссылок у других умных указателей. Затем он копирует ссылку и сохраняет её. Это распространённый шаблон, который также используется RTL при назначении интерфейсов объектов.

Этот умный указатель можно использовать следующим образом:

```pascal
type
  TFoo = class(TRefCountable)
  private
    FIntVal: Integer;
    FStrVal: String;
  public
    property IntVal: Integer read FIntVal write FIntVal;
    property StrVal: String read FStrVal write FStrVal;
  end;

procedure TestSmartPointer;
begin
  var Foo1 := TSmartPtr<TFoo>.Create(TFoo.Create);
  { The smart pointer has a reference count of 1. }

  { Set some properties }
  Foo1.Ref.IntVal := 42;
  Foo1.Ref.StrVal := 'Foo';

  begin
    { Copy the smart pointer.
      It has a reference count of 2 now. }
    var Foo2 := Foo1;

    { Check properties }
    Assert(Foo2.Ref.IntVal = 42);
    Assert(Foo2.Ref.StrVal = 'Foo');

    { Foo2 will go out of scope here, so only Foo1
      will keep a reference to the TFoo object.
      The reference count will be reduced to 1. }
  end;

  { Check properties again }
  Assert(Foo1.Ref.IntVal = 42);

  { Now Foo1 will go out of scope, reducing the
    reference count to 0 and destroying the TFoo object. }
end;
```

Теперь вы можете использовать вывод типа, например:

```pascal
var Foo1 := TSmartPtr<TFoo>.Create(TFoo.Create);
```

Этого нельзя было сделать при использовании объектных интерфейсов для умных указателей.

Конечно, большим недостатком этого метода является то, что его можно использовать только с классами, производными от `TRefCountable`. Этот умный указатель нельзя использовать с . `TStringList`или любым другим классом RTL.

## 3. Использование CMR с общим счетчиком ссылок

Одним из способов решения этой проблемы является динамическое выделение счетчика ссылок, чтобы его можно было использовать совместно с копиями интеллектуального указателя:

```pascal
type
  TSmartPtr<T: class> = record
  private
    FRef: T;
    FRefCount: PInteger;
    procedure Retain; inline;
    procedure Release; inline;
  public
    constructor Create(const ARef: T);
    class operator Initialize(out ADest: TSmartPtr<T>);
    class operator Finalize(var ADest: TSmartPtr<T>);
    class operator Assign(var ADest: TSmartPtr<T>;
      const [ref] ASrc: TSmartPtr<T>);

    property Ref: T read FRef;
  end;
  ```

Поле _не_`FRefCount` является текущим значением , а представляет собой указатель на целое число. Реализация немного сложнее, поскольку нам нужно выделять и освобождать это поле при необходимости:`Integer`

```pascal
constructor TSmartPtr<T>.Create(const ARef: T);
begin
  FRef := ARef;
  if (ARef <> nil) then
  begin
    GetMem(FRefCount, SizeOf(Integer));
    FRefCount^ := 0;
  end;
  Retain;
end;

class operator TSmartPtr<T>.Initialize(out ADest: TSmartPtr<T>);
begin
  ADest.FRef := nil;
  ADest.FRefCount := nil;
end;

class operator TSmartPtr<T>.Finalize(var ADest: TSmartPtr<T>);
begin
  ADest.Release;
end;

procedure TSmartPtr<T>.Retain;
begin
  if (FRefCount <> nil) then
    AtomicIncrement(FRefCount^);
end;

procedure TSmartPtr<T>.Release;
begin
  if (FRefCount <> nil) then
  begin
    if (AtomicDecrement(FRefCount^) = 0) then
    begin
      FRef.Free;
      FreeMem(FRefCount);
    end;

    FRef := nil;
    FRefCount := nil;
  end;
end;

class operator TSmartPtr<T>.Assign(var ADest: TSmartPtr<T>;
  const [ref] ASrc: TSmartPtr<T>);
begin
  if (ADest.FRef <> ASrc.FRef) then
  begin
    ADest.Release;
    ADest.FRef := ASrc.FRef;
    ADest.FRefCount := ASrc.FRefCount;
    ADest.Retain;
  end;
end;
```

Отличия по сравнению с версией 2:

- Конструктор выделяет память для `FRefCount`поля. Поскольку динамически выделяемая память содержит случайные данные, он должен установить это поле в 0.
- Оператор `Initialize`устанавливает как ссылку, так и `FRefCount`значение `nil`.
- Операторы `Retain`и `Release`теперь работают с `FRefCount`полем (если оно назначено). Когда счётчик ссылок достигает 0, освобождается не только объект, но и память, занимаемая самим счётчиком ссылок.
- Наконец, `Assign`оператор практически не меняется. Вам нужно лишь скопировать и ссылку, и указатель на счётчик ссылок.

Вы будете использовать этот умный указатель точно так же, как и в версии 2. Преимущество теперь состоит в том, что вы можете использовать этот умный указатель с любым классом, а не только с классами, производными от `TRefCountable`.

Однако есть и недостаток: для хранения счётчика ссылок приходится динамически выделять очень небольшой фрагмент памяти (4 байта). Это не только приводит к увеличению потребления памяти (поскольку менеджер памяти выделяет больше, чем 4 байта), но и увеличивает фрагментацию памяти.

Последняя версия пытается этого избежать.

## 4. Использование CMR с хаком монитора

Итак, основная проблема заключается в том, что нам нужно связать счётчик ссылок с объектом. В версии 2 это было реализовано путём определения базового класса со счётчиком ссылок, а в версии 3 использовался динамически выделяемый счётчик ссылок. Есть ли способ создать умный указатель, не требующий базового класса или динамически выделяемого поля? То есть, есть ли другой способ связать счётчик ссылок с объектом?

На самом деле, есть, но это своего рода хак, так что он может вас немного смутить. Вы можете знать, а можете и не знать, что у каждого объекта есть скрытое поле размером с указатель, которое используется, когда объект используется как монитор. Это происходит, когда вы передаёте объект `TMonitor.Enter`или `TMonitor.Leave`используете его как блокировку в многопоточных сценариях. Лично я всегда считал это скрытое поле пустой тратой памяти, поскольку лишь очень малая часть объектов используется как мониторы. Более того, хотя любой объект может использоваться как монитор, рекомендуется создавать отдельные объекты специально для этой цели (что, по моему скромному мнению, ещё одна причина, почему добавление этой функциональности на `TObject`уровне — пустая трата времени).

Но мы можем воспользоваться этим скрытым полем и хранить в нём счётчик ссылок. Если вы не используете объект в качестве монитора, это совершенно нормально.

Итак, как же нам добраться до этого скрытого поля? Delphi добавляет это скрытое поле в самом конце любого класса. Даже при создании подкласса и добавлении в него новых полей Delphi позаботится о том, чтобы поле монитора было добавлено _после_ всех добавленных вами пользовательских полей. Таким образом, чтобы добраться до поля, нужно найти ячейку памяти, соответствующую концу объекта, и вычесть значение размером с указатель. Модуль System определяет константы `hfFieldSize`и `hfMonitorOffset`помогает в этом. Таким образом, для любого объекта указатель на монитор можно найти следующим образом:

```pascal
function GetMonitorPtr(const AObj: TObject): Pointer;
begin
  Result := Pointer(IntPtr(AObj) + AObj.InstanceSize 
    - hfFieldSize + hfMonitorOffset);
end;
```

Компонент `IntPtr(AObj) + AObj.InstanceSize`вычисляет адрес, по которому только что пройден конец объекта. Затем он вычитает размер указателя ( `hfFieldSize`) и добавляет смещение к полю монитора ( `hfMonitorOffset`, которое в текущей версии Delphi равно 0).

Этот прием можно использовать для создания следующего умного указателя:
```pascal
type
  TSmartPtr<T: class> = record
  private
    FRef: T;
    function GetRefCountPtr: PInteger; inline;
    procedure Retain; inline;
    procedure Release; inline;
  public
    constructor Create(const ARef: T);
    class operator Initialize(out ADest: TSmartPtr<T>);
    class operator Finalize(var ADest: TSmartPtr<T>);
    class operator Assign(var ADest: TSmartPtr<T>;
      const [ref] ASrc: TSmartPtr<T>);

    property Ref: T read FRef;
  end;

Это очень похоже на версии 2 и 3, но добавляет метод `GetRefCountPtr`, который возвращает адрес поля счетчика ссылок (монитора):

constructor TSmartPtr<T>.Create(const ARef: T);
begin
  FRef := ARef;
  Assert((FRef = nil) or (GetRefCountPtr^ = 0));
  Retain;
end;

class operator TSmartPtr<T>.Initialize(out ADest: TSmartPtr<T>);
begin
  ADest.FRef := nil;
end;

class operator TSmartPtr<T>.Finalize(var ADest: TSmartPtr<T>);
begin
  ADest.Release;
end;

function TSmartPtr<T>.GetRefCountPtr: PInteger;
begin
  if (FRef = nil) then
    Result := nil
  else
    Result := PInteger(IntPtr(FRef) + FRef.InstanceSize
      - hfFieldSize + hfMonitorOffset);
end;

procedure TSmartPtr<T>.Retain;
begin
  var RefCountPtr := GetRefCountPtr;
  if (RefCountPtr <> nil) then
    AtomicIncrement(RefCountPtr^);
end;

procedure TSmartPtr<T>.Release;
begin
  var RefCountPtr := GetRefCountPtr;
  if (RefCountPtr <> nil) then
  begin
    if (AtomicDecrement(RefCountPtr^) = 0) then
      FRef.Free;

    FRef := nil;
  end;
end;

class operator TSmartPtr<T>.Assign(var ADest: TSmartPtr<T>;
  const [ref] ASrc: TSmartPtr<T>);
begin
  if (ADest.FRef <> ASrc.FRef) then
  begin
    ADest.Release;
    ADest.FRef := ASrc.FRef;
    ADest.Retain;
  end;
end;
```

Реализация лишь незначительно отличается от версий 2 и 3:

- Методы `Retain`и `Release`используют `GetRefCountPtr`вспомогательный метод для получения указателя на скрытое поле монитора. Вместо этого он использовал этот адрес для хранения счётчика ссылок.
- Прежде чем уничтожить объект, необходимо убедиться, что скрытое поле монитора имеет значение `nil`. В противном случае деструктор объекта попытается освободить монитор, который на самом деле не монитор, а счётчик ссылок. Это, скорее всего, приведёт к нарушению прав доступа. К счастью, объект освобождается только тогда, когда счётчик ссылок достигает 0, что равносильно установке поля монитора в значение `nil`. Поэтому нам не нужно делать ничего дополнительного.

Опять же, вы можете использовать этот умный указатель точно так же, как в версиях 2 и 3. Преимущество заключается в том, что его можно использовать с любым объектом, и он не требует какой-либо дополнительной динамически выделяемой памяти.

Хоть это и своего рода хак, он работает очень хорошо. Единственное, что нужно помнить, — это то, что объект нельзя использовать в качестве монитора (чего вы, скорее всего, и не делаете).

## Недостатки

Как и всё, умные указатели имеют и некоторые недостатки:

1. Во-первых, умные указатели приводят к небольшому снижению производительности из-за подсчета ссылок и скрытых вызовов операторов `Initialize`, `Finalize`и `Assign`.

2. Также, как и в случае с объектными интерфейсами, вы можете создать цикл ссылок:
   
```pascal
type
  TBar = class;

  TFoo = class
  private
    FBar: TSmartPtr<TBar>;
  public
    property Bar: TSmartPtr<TBar> read FBar write FBar;
  end;

  TBar = class
  private
    FFoo: TSmartPtr<TFoo>;
  public
    property Foo: TSmartPtr<TFoo> read FFoo write FFoo;
  end;

procedure MakeReferenceCycle;
begin
  { Create smart pointers for TFoo and TBar }
  var Foo := TSmartPtr<TFoo>.Create(TFoo.Create);
  var Bar := TSmartPtr<TBar>.Create(TBar.Create);

  { Make a reference cycle }
  Foo.Ref.Bar := Bar;
  Bar.Ref.Foo := Foo;
end;
```

Это приводит к утечке памяти, поскольку оба указателя `Foo`— и `Bar`— имеют сильную ссылку друг на друга и никогда не будут уничтожены, пока цикл не будет разорван. Атрибуты `[weak]`или нельзя использовать `[unsafe]`с умными указателями. Решением может быть ручной разрыв цикла в какой-то момент или _отказ_ от использования умного указателя на одной из сторон. Или можно создать что-то вроде `std::weak_ptr`типа в C++.

3. Другим недостатком является то, что вам придется вводить текст `.Ref`каждый раз, когда вы захотите получить доступ к указанному объекту:
```pascal
var List := TSmartPtr<TStringList>.Create(TStringList.Create);
List.Ref.Add('Foo');
List.Ref.Add('Bar');
```
Было бы неплохо, если бы Delphi предоставлял перегрузку операторов для оператора « `^`» (или даже « `.`»), чтобы этот код можно было переписать так:
```pascal
var List := TSmartPtr<TStringList>.Create(TStringList.Create);
List^.Add('Foo');
List^.Add('Bar');
```
Возможно, в будущей версии Delphi…

4. Аналогично, написание кода `TSmartPtr<TStringList>`каждый раз может стать немного громоздким. Хотя это можно смягчить, определив вспомогательные типы для регулярно используемых умных указателей:

```pascal
type
  TSPStringList = TSmartPtr<TStringList>;

begin
  var List := TSPStringList.Create(TStringList.Create);
end;
```

Вы можете использовать своего рода соглашение об именовании (например, `TSP`префикс *), чтобы было ясно, что тип представляет собой интеллектуальный указатель.

5. Вам всё ещё может показаться странным, что приходится `TStringList`сначала создавать объект, а затем передавать его конструктору умного указателя. Вы можете поручить умному указателю создать объект автоматически, добавив `constructor`ограничение к параметру generic-типа:
   
```pascal
type
  TSmartPtr<T: class, constructor> = record
  private
    FRef: T;
  public
    class function Create: TSmartPtr<T>; static;
    ...
  end;

{ TSmartPtr<T> }

class function TSmartPtr<T>.Create: TSmartPtr<T>;
begin
  Result.FRef := T.Create;
end;
```

Тогда вы можете использовать его вот так:

```pascal
type
  TSPStringList = TSmartPtr<TStringList>;

begin
  var List := TSPStringList.Create;
end;
```

Конечно, это работает только с классами, имеющими конструктор без параметров.

Вы также можете не использовать имя типа « `Create`» для создания умного указателя, поскольку многие ассоциируют его с конструированием объектов. Вместо этого можно использовать имя типа « `Make`» или « `New`».

6. И наконец, возможно, вы придерживаетесь старой школы и предпочитаете всегда вручную отслеживать создаваемые объекты. Таким образом, вы полностью контролируете момент уничтожения объекта. Конечно, это совершенно нормально: вы можете использовать то, что вам удобно.

Я совершенно не хочу снова поднимать дискуссию о ручном и автоматическом управлении памятью. У обоих подходов есть свои плюсы и минусы.

Но иногда между объектами возникают сложные связи, и становится сложно отследить, кто владеет объектом и отвечает за его освобождение. В таких случаях на помощь приходят умные указатели (или интерфейсы объектов).

## Заключение

Список недостатков немаленький. Но, к счастью, большинство из них незначительны и не окажут существенного влияния или их можно легко обойти.

Кроме того, это не единственные способы реализации умных указателей. В прошлом было предпринято множество более или менее успешных попыток создания умных указателей с помощью Delphi. Например, Руди Велтхейс представил подход, использующий запись [с интерфейсом, защищающим объект](https://rvelthuis.blogspot.com/2017/09/smart-pointers.html) . Барри Келли демонстрирует довольно оригинальный подход с использованием [анонимных методов](http://blog.barrkel.com/2008/11/somewhat-more-efficient-smart-pointers.html) , а Джаррод Холлингворт развивает его, предлагая собственную [реализацию умного указателя](https://adugmembers.wordpress.com/2011/12/05/smart-pointers) .

Теперь вы можете добавить Custom Managed Records в свой набор инструментов для создания умных указателей. Думаю, они работают довольно хорошо. Особенно версия 4, представленная выше, которая весьма эффективна, не использует дополнительную память и работает с любыми типами объектов.

Если хотите попробовать, загляните в наш репозиторий [JustAddCode](https://github.com/grijjy/JustAddCode/tree/master/CustomManagedRecords) за примерами реализаций. Вы можете модифицировать их по своему усмотрению.