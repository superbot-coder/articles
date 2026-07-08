
Перегрузка операторов — понятие, которое сложно описать. Должен признаться, статья в Википедии¹⁸ не имела для меня никакого смысла. Поэтому вместо статьи в Википедии вы можете использовать эту главу, чтобы разобраться с перегрузкой операторов.

Во-первых, что такое операторы? Операторы — это символы, используемые языком программирования для выполнения многих стандартных функций. Например, мы используем символ «+» для обозначения сложения.

Мы могли бы просто определить

```pascal
function Add(A, B: integer): integer;
```

для выполнения сложения. Однако это не так удобно, как просто набрать:

```pascal
A + B
```

Таким образом, операторы предоставляют более простой способ выражения многих распространенных функций.

Конечно, список операторов довольно длинный. Он включает все распространенные математические символы

`+, -, *, /, =, <, >, т.д.`

а также логические операторы, такие как `and` и `or`, и побитовые операторы, такие как `shl` и `shr`, и т.д. Кроме того, в Delphi вы можете перегружать операторы `Inc`, `Dec`, `Trunc` и `Round`.

Перегрузка операторов — это добавление функциональности операторам для различных типов данных. Знакомым примером является компилятор Delphi, который перегружает символ «+» для строк, позволяя вам делать такие вещи, как:

```pascal
SomeString := 'Why did the dog lay in the shade? ' + 'Because it didn''t want to be a hot dog';
```

Итак, перегрузка операторов — это способность добавлять функциональность операторам, применяя их к новым типам данных.

Delphi позволяет выполнять перегрузку операторов для типов записей (record). Вы можете определить запись, а затем перегрузить любой из длинного списка операторов, чтобы применить новую функциональность к этим новым типам записей. (Полный список операторов, которые можно перегружать, можно найти в документации Delphi¹⁹).

Поскольку они поддерживают автоматический подсчет ссылок (ARC), новые версии мобильного компилятора позволяют перегружать операторы для классов. Однако, как упоминалось в Предисловии, мы будем придерживаться здесь компилятора для Windows и, таким образом, сосредоточимся на перегрузке операторов для записей.

### Пример: TFraction

Пример должен прояснить ситуацию. Компьютеры работают с целыми числами и числами с плавающей точкой, но может возникнуть ситуация, когда вам потребуется использовать хорошие старые дроби. Так почему бы нам не создать тип `TFraction`, перегрузив общие математические операторы, которые позволят нам складывать, вычитать, умножать, делить и сравнивать дроби? Звучит неплохо, верно?

Хорошо, давайте начнем с простого определения `TFraction`:

```pascal
type
  TFraction = record
  strict private
    FNumerator: Integer;
    FDenominator: Integer;
    class function GCD(a, b: Integer): Integer; static;
    function Reduced: TFraction;
  public
    class function CreateFrom(aNumerator: Integer; aDenominator: Integer): TFraction; static;

    property Numerator: Integer read FNumerator;
    property Denominator: Integer read FDenominator;
  end;
```

Это простая запись со свойствами `Numerator` (числитель) и `Denominator` (знаменатель). У нее также есть своего рода конструктор под названием `CreateFrom`, который позволяет инициализировать `TFraction`. Эта функция, вероятно, немного сложнее, чем вы могли подумать:

```pascal
class function TFraction.CreateFrom(aNumerator, aDenominator: Integer): TFraction;
begin
  if aDenominator = 0 then
  begin
    raise EZeroDivide.CreateFmt('Invalid fraction %d/%d, numerator cannot be zero', [aNumerator, aDenominator]);
  end;
  if (aDenominator < 0) then
  begin
    Result.FNumerator := -aNumerator;
    Result.FDenominator := -aDenominator;
  end else
  begin
    Result.FNumerator := aNumerator;
    Result.FDenominator := aDenominator;
  end;
  Assert(Result.Denominator > 0); // необходимо для сравнений
end;
```

Во-первых, поскольку мы имеем дело только с рациональными числами, у вас не может быть знаменателя, равного нулю, поэтому возникает исключение, если вы попытаетесь создать дробь с нулем внизу. Далее, чтобы обеспечить лучший код при выполнении операторов сравнения немного позже, если знаменатель отрицательный, мы переносим знак в числитель.

Теперь мы можем создать дробь вот так:

```pascal
var
  OneThird: TFraction;
begin
  OneThird := TFraction.CreateFrom(1, 3);
end;
```

Но какой в этом смысл? Не лучше ли было бы производить математические операции с этими дробями? Что ж, мы могли бы сделать это так:

```pascal
function AddFractions(a, b: TFraction): TFraction;
```

но это было бы громоздко и неинтересно. Было бы гораздо лучше иметь возможность сделать:

```pascal
var
  A, B, C: TFraction;
begin
  A := TFraction.Create(1, 2);
  B := TFraction.Create(1, 3);
  C := A + B;
end;
```

Что ж, давайте сделаем это возможным. Для этого нам нужно добавить некоторые методы `class operator` к записи `TFraction`. Мы хотим иметь возможность складывать дроби вместе, поэтому для выполнения сложения нам понадобится этот метод:

```pascal
class operator Add(const Left, Right: TFraction): TFraction;
```

Объявление `class operator` — это специальный метод класса, который указывает, что оператор должен быть перегружен. В данном случае мы перегружаем сложение, на что указывает имя метода `Add`. Компилятор распознает `Add` как специальное имя метода и использует его для переопределения оператора «+». За `class operator` должно следовать конкретное имя оператора, как определено в ссылке на документацию выше. Например, если вы объявите

```pascal
class operator FooBar(const AValue: TFraction): TFraction;
```

вы получите ошибку вроде этой:

`[dcc32 Error] uFractions.pas(36): E2393 Invalid operator declaration`

Другими словами, вам нужно иметь корректное, допустимое имя оператора для объявления `class operator`, чтобы скомпилировать код. Здесь у нас есть `Add`, но через минуту мы добавим еще, например `Subtract` и `Negative`.

Однако нет никаких ограничений на типы параметров и возвращаемые значения. Вы можете предоставлять перегрузки и конвертеры для любого типа Delphi, если это уместно для вашей перегрузки.

Итак, как вы складываете две дроби?

```pascal
class operator TFraction.Add(const Left, Right: TFraction): TFraction;
begin
  Result := CreateFrom(Left.Numerator * Right.Denominator + Left.Denominator * Right.Numerator, Left.Denominator * Right.Denominator).Reduced;
end;
```

Этот код создает новый экземпляр `TFraction` и выполняет формулу для сложения двух дробей вместе. И, наконец, он вызывает `Reduced`, чтобы сократить результирующую дробь до наименьшего общего знаменателя. `Reduced` объявляется следующим образом:

```pascal
function TFraction.Reduced: TFraction;
var
  LGCD: Integer;
begin
  LGCD := GCD(Numerator, Denominator);
  Result := CreateFrom(Numerator div LGCD, Denominator div LGCD);
end;
```

а `GCD` объявляется как:

```pascal
class function TFraction.GCD(a, b: Integer): Integer;
var
  rem: Integer;
begin
  rem := a mod b;
  if rem = 0 then
    Result := b;
  else
    Result := GCD(b, rem)
end;
```

Я не математик, но я понимаю, что функция `GCD` — это рекурсивная реализация алгоритма Евклида для нахождения наибольшего общего делителя. Впечатляет, да?

В любом случае, код гарантирует, что все результаты правильно сокращены, и вы увидите вызов `Reduced` в конце любого метода, возвращающего `TFraction`.

Хорошо, это была реализация для сложения двух дробей вместе. Код для сложения дроби с целым числом очень похож, так как целое число — это просто дробь со знаменателем 1, поэтому я не буду показывать его здесь. (Вы можете проверить его в репозитории BitBucket²⁰ для кода книги).

Как насчет вычитания? Что ж, прежде чем мы посмотрим на вычитание, нам нужно взглянуть на другой оператор:

```pascal
class operator Negative(const AValue: TFraction): TFraction;
```

Это перегрузка знака минус перед дробью, что, конечно, делает ее отрицательной. Она реализована следующим образом:

```pascal
class operator TFraction.Negative(const AValue: TFraction): TFraction;
begin
  Result := CreateFrom(-AValue.Numerator, AValue.Denominator);
end;
```

Ее реализация довольно очевидна — она просто создает отрицательную версию переданного ей `TFraction`, делая числитель отрицательным. Я показал ее вам первой, прежде чем взглянуть на метод `Subtraction`, потому что метод `Subtraction` использует ее. Вот реализация для вычитания одной дроби из другой:

```pascal
class operator TFraction.Subtract(const Left, Right: TFraction): TFraction;
begin
  Result := Left + (-Right);
end;
```

Обратите внимание, что классовые операторы могут строиться друг на друге. Вычитание использует оператор `Add` и оператор `Negative` вместе для создания вычитания, что приводит к более легкому для чтения и поддержки коду.

Методы `Multiply` и `Divide` реализованы точно так, как вы могли бы подумать, поэтому я не буду рассматривать их здесь.

Следующий набор операторов, на который мы посмотрим, — это операторы сравнения, то есть `=`, `<>`, `>`, `<`, `<=` и `>=`. Опять же, они будут строиться друг на друге, поэтому мы начнем с `Equal` и `NotEqual`.

```pascal
class operator TFraction.Equal(const Left, Right: TFraction): Boolean;
begin
  Result := Left.Numerator * Right.Denominator = Right.Numerator * Left.Denominator;
end;

class operator TFraction.NotEqual(const Left, Right: TFraction): Boolean;
begin
  Result := not (Left = Right);
end;
```

Обратите внимание, что метод `Equal` использует классический алгоритм, который вы изучили в четвертом классе, чтобы определить, равны ли две дроби, но метод `NotEqual` просто использует преимущества оператора `Equal`, используя `not` для получения правильного результата.

Естественно, следуя из этого, идут операторы больше/меньше, включая равенство. И угадайте что, они тоже строятся друг на друге:

```pascal
class operator TFraction.LessThan(const Left, Right: TFraction): Boolean;
begin
  Result := Left.Numerator * Right.Denominator < Right.Numerator * Left.Denominator;
end;

class operator TFraction.GreaterThan(const Left, Right: TFraction): Boolean;
begin
  Result := Left.Numerator * Right.Denominator > Right.Numerator * Left.Denominator;
end;

class operator TFraction.LessThanOrEqual(const Left, Right: TFraction): Boolean;
begin
  Result := (Left < Right) or (Left = Right);
end;

class operator TFraction.GreaterThanOrEqual(const Left, Right: TFraction): Boolean;
begin
  Result := (Left = Right) or (Left > Right);
end;
```

Опять же, код довольно прост и практически такой, каким вы его ожидаете. Обратите внимание, что `GreaterThanOrEqual` и `LessThanOrEqual` используют существующую функциональность для своей работы.

## Присваивания

Итак, теперь у нас есть `TFraction`, который может выполнять базовые математические операции и сравнения. Но что, если вы хотите присвоить `TFraction` другому типу, скажем, `Double`, или если вы хотите присвоить целое число переменной типа `TFraction`?

Что ж, вы можете сделать это, используя вызов class operator `Implicit`. `Implicit` — это оператор, который позволяет писать код, присваивающий один тип другому. Для `TFraction` мы объявим два таких оператора:

```pascal
class operator Implicit(const aValue: Integer): TFraction;
class operator Implicit(const aValue: TFraction): Double;
```

Один позволит преобразовать целое число в `TFraction`, а другой преобразует `TFraction` в double. Реализации просты:

```pascal
class operator TFraction.Implicit(const AValue: Integer): TFraction;
begin
  Result := TFraction.CreateFrom(aValue, 1);
end;

class operator TFraction.Implicit(const aValue: TFraction): Double;
begin
  Result := aValue.Numerator / aValue.Denominator;
end;
```

Строгая типизация Delphi не позволяет мне хотеть разрешить присваивание `TFraction` строке, но я предоставил метод `ToString`, который превращает дробь в читаемый вид:

```pascal
function TFraction.ToString: string;
begin
  Result := Format('%d/%d', [Numerator, Denominator]);
end;
```

Кроме того, я использовал метод class operator `Explicit` для разрешения жёсткого приведения `TFraction` к строке. Этот оператор позволяет писать такой код:

```pascal
SomeString := string(TFraction.CreateFrom(1, 3));
```

## Implicit против Explicit

Неявное приведение происходит, когда, например, вы передаёте строку функции, принимающей `Variant`, или напрямую присваиваете целое число переменной типа `Double`. В нашем примере мы можем использовать неявное преобразование для сложения `TFraction` с целым числом. Это избавляет нас от необходимости реализовывать несколько операторов `Add`:

```pascal
// Не нужны из-за неявных преобразований
class operator Add(const Left: TFraction; const Right: Integer): TFraction;
class operator Add(const Left: Integer; const Right: TFraction): TFraction;
```

Таким образом, мы можем неявно складывать `TFraction` с целым числом. А как насчёт сложения `TFraction` и double с результатом в double? Что ж, у нас есть следующее:

```pascal
class operator Implicit(const aValue: TFraction): Double;
```

поэтому вы могли бы подумать, что этот тест пройдёт:

```pascal
procedure TFractionTests.TestAddDouble;
var
  a: TFraction;
  b: Double;
begin
  a := TFraction.CreateFrom(1, 3);
  b := a + 0.5;
  Assert.AreEqual(b, 5/6);
end;
```

но на самом деле он даже не скомпилируется, выдавая ошибку:

`[dcc32 Error] uFractionsTests.pas(111): E2015 Operator not applicable to this operand type`

Это связано с ограничениями компилятора Delphi. Он не «видит», что, преобразовав `TFraction` в `Double` через неявное приведение, он фактически может выполнить сложение.

Для того чтобы вышеуказанный тест скомпилировался и прошёл, вы можете реализовать непосредственно оператор `Add` для `TFraction` и `TDouble` следующим образом:

```pascal
class operator TFraction.Add(const Left: TFraction; const Right: double): double;
var
  LDouble: Double;
begin
  LDouble := Left;
  Result := LDouble + Right;
end;
```

и тогда наш тест пройдёт.

Обратите внимание, что у меня нет неявного конвертера для преобразования `TFraction` в `integer`. Почему? Ну, потому что такое преобразование было бы «с потерями». То есть вы потеряли бы информацию при этом. Подумайте сами — как преобразовать 3/4 в целое число? На самом деле это невозможно, и поэтому такая функциональность не предусмотрена.

Я принял это решение за пользователя. Но если бы я захотел позволить пользователю самому выбрать такое преобразование с потерями, я мог бы предоставить метод явного преобразования (`Explicit`), который позволил бы пользователю выполнить «жёсткое» приведение `TFraction` к `integer`. Жёсткие приведения — ответственность разработчика, поэтому явное преобразование может быть предоставлено по желанию, понимая, что оно перекладывает ответственность за потерю данных на пользователя. Такое преобразование могло бы усечь или округлить дробь до целого числа.

## Использование TFraction

Итак, давайте немного потренируемся. Вот некоторые вещи, которые можно делать с `TFraction`. Сначала сложим несколько дробей:

```pascal
var
  A, B, C, D: TFraction;
  S: string;
  Dub: Double;
begin
  A := TFraction.CreateFrom(2, 6);
  B := TFraction.CreateFrom(4, 18);
  C := A + B;
  S := C.ToString;
  WriteLn(S);
end;
```

Это выведет 5/9. Обратите внимание, что дробь была сокращена с 10/18.

А как насчёт методов `Negative` и `Explicit`? Позволяют ли они сделать дробь отрицательной и жёстко привести дробь к строке? Конечно, да.

```pascal
C := TFraction.CreateFrom(7, 12);
D := -C;
S := string(D);
WriteLn(S);
```

в результате получим -7/12.

А как насчёт неравенства?

```pascal
A := TFraction.CreateFrom(1, 2);
B := TFraction.CreateFrom(2, 5);
if A <> B then
begin
  WriteLn('Not Equal');
end else
begin
  WriteLn('Inequality check failed');
end;
```

Да, это выводит Not Equal, как и ожидалось.

А как насчёт GreaterThan:

```pascal
A := TFraction.CreateFrom(1, 3);
B := TFraction.CreateFrom(1, 100);
if A > B then
begin
  WriteLn('Greater Than');
end else
begin
  WriteLn('Greater than check failed');
end;
```

Да, это работает. На самом деле всё работает, и я не буду повторять демонстрации для всех возможных вызовов методов. Вы можете посмотреть тестовый код в репозитории BitBucket²¹. Суть в том, что теперь у нас есть возможность делать практически всё, что нам нужно, с `TFraction`.

> Демо-код, являющийся частью этой главы, включает полный набор модульных тестов, показывающих, что `TFraction` работает именно так, как задумано.

## Заключение

И вот он — новый record `TFraction`, который можно использовать со всеми основными математическими операторами. Признайтесь, код совсем не сложный.

> Delphi поставляется с более сложным примером операторной перегрузки, работающим с комплексными числами. Его можно найти на вашем жёстком диске, скорее всего по пути:
> C:\Users\Public\Documents\Embarcadero\Studio\16.0\Samples\Object Pascal\RTL\ComplexNumbers
> Это демо написано великим Халльвардом Вассботном и является очень полным.

Тем не менее, это мощная техника, которая может привести к гораздо более чистому коду. Никто не хочет видеть

```pascal
Sum := ThisFraction.Add(ThatFraction);
```

когда можно написать

```pascal
Sum := ThisFraction + ThatFraction;
```

Выглядит довольно очевидно, правда?

Операторная перегрузка подходит не для каждой ситуации — не каждый record будет требовать или нуждаться в ней, но это может быть очень полезной техникой для тех типов данных, над которыми можно выполнять «операции». Как правило, только те records, которые могут использоваться в математических выражениях, должны иметь перегруженные математические операторы. Не увлекайтесь чрезмерно — вы легко можете получить очень запутанный код, перегружая operators для records, которые этого не заслуживают.

---
¹⁸ http://en.wikipedia.org/wiki/Operator_overloading
¹⁹ http://docwiki.embarcadero.com/RADStudio/XE7/en/Operator_Overloading_%28Delphi%29
²⁰ https://bitbucket.org/NickHodges/nickdemocode/src/11f331ec18dc0f2f4f6ade17153bff0b31ab13cf/MoreCodingInDelphi/?at=default
*²¹ https://bitbucket.org/NickHodges/nickdemocode/src/11f331ec18dc0f2f4f6ade17153bff0b31ab13cf/MoreCodingInDelphi/?at=default*