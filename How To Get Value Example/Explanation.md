Создать `SingleInstance` `Codeunit 50002`. У него установить свойство `EventSubscriberInstance=Manual;`.
В нём сделать глобальную переменную и 1 глобальный метод - `GetMagicValue` этой переменной.
```cs
[EventSubscriber(Codeunit,50000,OnAfterSetMagicValue)]
LOCAL PROCEDURE OnCodeunit50000OnAfterSetMagicValue@1000000001(MagicValueP@1000000000 : Integer);
BEGIN
  MagicValueG := MagicValueP;
END;

PROCEDURE GetMagicValue@1000000003() : Integer;
VAR
  Value@1000000000 : Integer;
BEGIN
  Value := MagicValueG;
  CLEAR(MagicValueG);
  EXIT(Value);
END;
```

В юните `Codeunit 50000` (где функция, сигнатуру которой нельзя менять) создать функцию-событие (публикатор) и в эту функцию передаём нужную переменную. В функции, из которой нужно получить значение, после присвоения нужной переменной добавить вызов события.
```cs
PROCEDURE GeneralFunction@1000000000(ValueP@1000000000 : Integer);
VAR
  MagicValue@1000000001 : Integer;
BEGIN
  MagicValue := ValueP + 100;
  OnAfterSetMagicValue(MagicValue);
END;

[Integration]
LOCAL PROCEDURE OnAfterSetMagicValue@1000000002(MagicValueP@1000000000 : Integer);
BEGIN
END;

BEGIN
END.
```

А в месте, где нужно сделать получение переменной добавить - в `Codeunit 50001` подписываемся вручную на событие.
`SingleInstance` работает на сессию только одного пользователя и пересечений значений не будет.
```cs
LOCAL PROCEDURE TestGetMagicValueWithSubscription@1000000000();
VAR
  Codeunit50000@1000000000 : Codeunit 50000;
  Codeunit50002@1000000001 : Codeunit 50002;
  SeekValueBinded@1000000002 : Integer;
  SeekValueUnBinded@1000000003 : Integer;
BEGIN
  BINDSUBSCRIPTION(Codeunit50002);
  Codeunit50000.GeneralFunction(50);
  UNBINDSUBSCRIPTION(Codeunit50002);
  SeekValueBinded := Codeunit50002.GetMagicValue();

  Codeunit50000.GeneralFunction(50);
  SeekValueUnBinded := Codeunit50002.GetMagicValue();

  MESSAGE('Binded: %1; UnBinded: %2', SeekValueBinded, SeekValueUnBinded);
END;
```