# О настраиваемых записях
[Автор: Erik van Bilsen](https://blog.grijjy.com/author/erikvanbilsen)
[Ссылка на оригинал:](https://blog.grijjy.com/2020/08/03/automate-restorable-operations-with-custom-managed-records)

Если вы еще не знакомы с пользовательскими управляемыми записями (далее именуемыми CMR), то взгляните на мою [предыдущую публикацию об использовании CMR для обертывания API C(++)](https://blog.grijjy.com/2020/07/20/wrapping-c-apis-with-custom-managed-records) .

В качестве краткого обзора рассмотрим следующий CMR:

```pascal
type
  TFoo = record
  public
    class operator Initialize(out ADest: TFoo);
    class operator Finalize(var ADest: TFoo);
    class operator Assign(var ADest: TFoo;
      const [ref] ASrc: TFoo);
  end;
```

Тогда Delphi будет автоматически вызывать `Initialize`оператор при объявлении такой записи и `Finalize`при выходе записи из области действия. `Assign`Оператор вызывается для копирования одной записи в другую. Необязательно использовать все три оператора, но для создания CMR необходим хотя бы один.

Теперь, когда вы пишете следующий код:

```pascal
begin
  var Foo1: TFoo;
  var Foo2 := Foo1;
end;
```

Затем Delphi автоматически преобразует это в следующий код (обратите внимание, что этот код не компилируется):

```pascal
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

Delphi автоматически вставляет `try..finally`операторы, чтобы гарантировать, что `Finalize`оператор всегда вызывается. Это ключ к автоматизации «восстанавливаемых» операций.

Более подробную информацию о Custom Managed Records можно найти в [официальной документации](https://docwiki.embarcadero.com/RADStudio/Sydney/en/Custom_Managed_Records) , моем [предыдущем посте](https://blog.grijjy.com/2020/07/20/wrapping-c-apis-with-custom-managed-records) или на других ресурсах в Интернете.

В оставшейся части статьи показаны три способа использования CMR для автоматизации восстанавливаемых операций.

## 1. Автоматическое начало обновления и завершение обновления

Начнём с простого примера: у всех элементов управления FireMonkey есть методы `BeginUpdate`и `EndUpdate`методы, которые следует вызывать для эффективного массового обновления элемента управления. Когда такие методы встречаются парами, вы инстинктивно используете `try..finally`блок, чтобы гарантировать его `EndUpdate`постоянный вызов:

```pascal
ListView.BeginUpdate;
try
  ListView.Items.Add.Text := 'Item 1';
  ListView.Items.Add.Text := 'Item 2';
finally
  ListView.EndUpdate;
end;
```
Всякий раз, когда вы видите подобные закономерности, существует возможность автоматизации этого процесса с помощью CMR:
```pascal
type
  TAutoUpdate = record
  private
    FControl: TControl; // Reference
  public
    constructor Create(const AControl: TControl);
  public
    class operator Initialize(out ADest: TAutoUpdate);
    class operator Finalize(var ADest: TAutoUpdate);
    class operator Assign(var ADest: TAutoUpdate;
      const [ref] ASrc: TAutoUpdate);
  end;

constructor TAutoUpdate.Create(const AControl: TControl);
begin
  Assert(Assigned(AControl));
  FControl := AControl;
  FControl.BeginUpdate;
end;

class operator TAutoUpdate.Initialize(out ADest: TAutoUpdate);
begin
  ADest.FControl := nil;
end;

class operator TAutoUpdate.Finalize(var ADest: TAutoUpdate);
begin
  if (ADest.FControl <> nil) then
    ADest.FControl.EndUpdate;
end;

class operator TAutoUpdate.Assign(var ADest: TAutoUpdate;
  const [ref] ASrc: TAutoUpdate);
begin
  raise EInvalidOperation.Create(
    'TAutoUpdate records cannot be copied')
end;
```

Некоторые заметки:

- Оператор `Initialize`просто очищает ссылку на элемент управления. Это очень распространённый шаблон, который часто встречается в CMR. Обратите внимание, что добавлять этот оператор не обязательно _,_ но ниже мы объясним, почему это стоит сделать.
- Вы передаёте элемент управления, который хотите автоматически обновить, конструктору. Конструктор немедленно вызывает `BeginUpdate`его.
- Оператор `Finalize`вызывается в `finally`разделе скрытого `try..finally`блока, поэтому именно здесь вы вызываете `EndUpdate`. Однако сначала необходимо проверить, назначен ли элемент управления, поскольку это возможно, `nil`даже если конструктор не был вызван.
- И наконец, такие CMR-записи не должны копироваться (что объясняется ниже). Поэтому мы создаём исключение в `Assign`операторе, если вы всё равно попытаетесь их скопировать.

Теперь мы можем переписать фрагмент примера следующим образом:

```pascal
var AutoUpdate := TAutoUpdate.Create(ListView);
ListView.Items.Add.Text := 'Item 1';
ListView.Items.Add.Text := 'Item 2';
```

Вы просто создаёте CMR-метод автообновления, который немедленно вызывает `BeginUpdate`. Затем, когда CMR выходит из области действия (в конце метода), `EndUpdate`метод будет автоматически вызван благодаря скрытому `try..finally`блоку и `Finalize`оператору.

Этот CMR экономит 4 строки кода каждый раз, когда вы им пользуетесь, что, по-моему, довольно здорово (конечно, если вам не платят за каждую строку).

CMR отлично работают со встроенными переменными, как в этом примере. Если вы объявляете `AutoUpdate`переменную как «обычную» локальную переменную, `Initialize`оператор будет вызван сразу после входа в метод. При использовании встроенной переменной оператор `Initialize`будет вызван в том месте, где вы объявляете встроенную переменную, что даёт вам больше контроля.

> Подобные CMR очень распространены в C++ (хотя в C++ они не называются CMR). В C++ конструкторы и деструкторы вызываются автоматически при объявлении объекта/структуры в стеке. Кроме того, в C++ есть конструкторы копирования, которые аналогичны `Assign`операторам в Delphi.

### Зачем нужен оператор инициализации?

Что же произойдёт, если не написать `Initialize`оператор? В этом случае CMR вообще не будет инициализирован (в том числе и компилятором). Это означает, что `FControl`поле в образце CMR содержит случайные данные. Это не должно быть проблемой, если вы всегда вызываете конструктор, но что произойдёт, если вы этого не сделаете?
```
begin
  var AutoUpdate: TAutoUpdate;
end;
```
В этом примере вы просто объявляете CMR, не создавая его. Если бы оператора не было `Initialize`, его `FControl`поле содержало бы случайные данные. Затем, когда CMR выходит за пределы области действия, `Finalize`вызывается оператор, который вызывает `EndUpdate`метод для недопустимого объекта, что, скорее всего, приводит к нарушению прав доступа.

### Почему именно оператор Assign?

Как я уже упоминал выше, подобные CMR-записи не должны копироваться. Но почему? Предположим, у вас есть следующий код:

```pascal
var AutoUpdate := TAutoUpdate.Create(ListView);
var AutoUpdateCopy := AutoUpdate;
ListView.Items.Add.Text := 'Item 1';
ListView.Items.Add.Text := 'Item 2';
```

Выделенная строка в этом примере вызовет `Assign`оператор (если он доступен) или выполнит обычное копирование записи. Теперь, когда копия выходит за пределы области действия, `Finalize`вызывается её оператор, который вызывает `EndUpdate`. Затем, когда оригинал выходит за пределы области действия, `EndUpdate`снова вызывается , что приводит к несбалансированной паре `BeginUpdate`/ .`EndUpdate`

К счастью, копировать такие CMR нет смысла. Чтобы избежать копирования CMR по ошибке, следует написать `Assign`оператор и обработать эту ситуацию. В этом примере я вызываю исключение при попытке копирования. Но эту ситуацию можно обработать и другими способами. Например, вместо этого можно установить поле `FControl`целевого CMR равным `nil`, что предотвратит вызов `EndUpdate`.

## 2. Автоматическая блокировка

Распространённый шаблон проектирования многопоточной обработки — это блокировка ресурса для его защиты, последующее использование этого ресурса и последующее снятие блокировки. Конечно, это помещается в `try..finally`блок, чтобы гарантировать, что блокировка всегда будет снята. Рассмотрим следующий простой заблокированный список:

```pascal
type
  TLockedList<T> = class
  private
    FList: TList<T>;
    FLock: TSynchroObject;
  public
    constructor Create;
    destructor Destroy; override;
    procedure Add(const AItem: T);
  end;

constructor TLockedList<T>.Create;
begin
  inherited;
  FList := TList<T>.Create;
  FLock := TCriticalSection.Create;
end;

destructor TLockedList<T>.Destroy;
begin
  FLock.Free;
  FList.Free;
  inherited;
end;

procedure TLockedList<T>.Add(const AItem: T);
begin
  FLock.Acquire;
  try
    FList.Add(AItem);
  finally
    FLock.Release;
  end;
end;

Метод Add может использовать преимущества CMR:

procedure TLockedList<T>.Add(const AItem: T);
begin
  var AutoLock := TAutoLock.Create(FLock);
  FList.Add(AItem);
end;
```

А CMR может выглядеть так:
```pascal
type
  TAutoLock = record
  private
    FSyncObj: TSynchroObject; // Reference
  public
    constructor Create(const ASyncObj: TSynchroObject);
  public
    class operator Initialize(out ADest: TAutoLock);
    class operator Finalize(var ADest: TAutoLock);
    class operator Assign(var ADest: TAutoLock;
      const [ref] ASrc: TAutoLock);
  end;

constructor TAutoLock.Create(const ASyncObj: TSynchroObject);
begin
  Assert(Assigned(ASyncObj));
  FSyncObj := ASyncObj;
  FSyncObj.Acquire;
end;

class operator TAutoLock.Initialize(out ADest: TAutoLock);
begin
  ADest.FSyncObj := nil;
end;

class operator TAutoLock.Finalize(var ADest: TAutoLock);
begin
  if (ADest.FSyncObj <> nil) then
    ADest.FSyncObj.Release;
end;

class operator TAutoLock.Assign(var ADest: TAutoLock;
  const [ref] ASrc: TAutoLock);
begin
  raise EInvalidOperation.Create(
    'TAutoLock records cannot be copied');
end;
```

Эта CMR очень похожа на `TAutoUpdate`CMR, представленную ранее, поэтому я не буду вдаваться в подробности. Обратите внимание, что её можно использовать с любым типом объекта синхронизации (критическая секция, мьютекс и т. д.).

> Эта CMR настолько распространена в C++, что является частью стандартной библиотеки под именем `std::lock_guard`.

## 3. Автоматическое уничтожение объектов

А как насчёт очевидного варианта использования: автоматического уничтожения объектов? Конечно, можно использовать для этого похожую CMR:

```pascal
type
  TAutoFree = record
  private
    FInstance: TObject; // Reference
  public
    constructor Create(const AInstance: TObject);
    class operator Initialize(out ADest: TAutoFree);
    class operator Finalize(var ADest: TAutoFree);
    class operator Assign(var ADest: TAutoFree;
      const [ref] ASrc: TAutoFree);
  end;

constructor TAutoFree.Create(const AInstance: TObject);
begin
  Assert(Assigned(AInstance));
  FInstance := AInstance;
end;

class operator TAutoFree.Initialize(out ADest: TAutoFree);
begin
  ADest.FInstance := nil;
end;

class operator TAutoFree.Finalize(var ADest: TAutoFree);
begin
  ADest.FInstance.Free;
end;

class operator TAutoFree.Assign(var ADest: TAutoFree;
  const [ref] ASrc: TAutoFree);
begin
  raise EInvalidOperation.Create(
    'TAutoFree records cannot be copied')
end;
```

> Это похоже на `std::unique_ptr`C++

Вы можете использовать это так:

```pascal
begin
  var Writer := TStreamWriter.Create('foo.txt');
  var AutoFree := TAutoFree.Create(Writer);
  Writer.WriteLine('Some Text');
  OpenFile('foo.txt');
end;
```

Однако в этом примере есть проблема: `TStreamWriter`объект будет автоматически уничтожен, когда `AutoFree`CMR выйдет из области действия. Это происходит в конце метода, _после_ вызова `OpenFile`. Это означает, что `OpenFile`вызов может не открыть файл, поскольку у потокового редактора всё ещё есть исключительная блокировка. Однако с помощью встроенных переменных мы можем ограничить область действия `AutoFree`CMR и гарантировать, что потоковый редактор будет уничтожен до открытия файла:

```pascal
begin
  var Writer := TStreamWriter.Create('foo.txt');
  begin
    var AutoFree := TAutoFree.Create(Writer);
    Writer.WriteLine('Some Text');
  end;
  OpenFile('foo.txt');
end;
```

Теперь `AutoFree`CMR выходит из области видимости _до_ `OpenFile` вызова. Обратите внимание, что вы также можете поместить конструкцию записи потока во внутреннюю область видимости.

Хотя эта CMR может быть полезна, её нельзя использовать для автоматического управления жизненным циклом объекта в ситуациях, когда он используется совместно. В таких ситуациях традиционно используются объектные интерфейсы, позволяющие воспользоваться преимуществами автоматического подсчёта ссылок.

Но с появлением CMR появилась альтернатива: CMR можно использовать для добавления автоматического подсчета ссылок к обычным объектам.

> Что было бы похоже на `std::shared_ptr`в C++

И вот тут мы подходим к теме «умных указателей», которая выходит за рамки этой статьи, но может стать темой будущей статьи…