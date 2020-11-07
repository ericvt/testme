---
title: Использование инструментария WMI
description: PowerShell поставляется с командлетами для работы с инструментарием WMI с момента выпуска.
ms.date: 06/02/2020
ms.topic: guide
ms.custom: Contributor-mikefrobbins
ms.reviewer: mirobb
ms.openlocfilehash: 243685efa1f976ddb46a0d0efc4ed0635844606d
ms.sourcegitcommit: 0d958eac5bde5ccf5ee2c1bac4f009a63bf71368
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/05/2020
ms.locfileid: "84438295"
---
# <a name="chapter-7---working-with-wmi"></a>Глава 7. Использование инструментария WMI

## <a name="wmi-and-cim"></a>Инструментарий WMI и CIM

По умолчанию PowerShell поставляется с командлетами для использования вместе с другими технологиями, такими как инструментарий управления Windows (WMI). Существует несколько собственных командлетов WMI, которые используются в PowerShell без необходимости установки дополнительного программного обеспечения или модулей.

PowerShell поставляется с командлетами для работы с инструментарием WMI с момента выпуска. `Get-Command` можно использовать для определения командлетов WMI, существующих в PowerShell. Приведенные ниже результаты получены на моем компьютере с Windows 10 в лабораторной среде под управлением PowerShell версии 5.1. Ваши результаты могут отличаться в зависимости от используемой версии PowerShell.

```powershell
Get-Command -Noun WMI*
```

```Output
CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          Get-WmiObject                                      3.1.0.0    Microsof...
Cmdlet          Invoke-WmiMethod                                   3.1.0.0    Microsof...
Cmdlet          Register-WmiEvent                                  3.1.0.0    Microsof...
Cmdlet          Remove-WmiObject                                   3.1.0.0    Microsof...
Cmdlet          Set-WmiInstance                                    3.1.0.0    Microsof...
```

Командлеты модели CIM появились в PowerShell версии 3.0. Командлеты CIM разработаны так, чтобы их можно было использовать на компьютерах под управлением Windows и других ОС. Командлеты WMI являются устаревшими, поэтому вместо них рекомендуется использовать командлеты CIM.

Все командлеты CIM содержатся в модуле. Чтобы получить список командлетов CIM, используйте `Get-Command` вместе с параметром **Module**, как показано в следующем примере.

```powershell
Get-Command -Module CimCmdlets
```

```Output
CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          Export-BinaryMiLog                                 1.0.0.0    CimCmdlets
Cmdlet          Get-CimAssociatedInstance                          1.0.0.0    CimCmdlets
Cmdlet          Get-CimClass                                       1.0.0.0    CimCmdlets
Cmdlet          Get-CimInstance                                    1.0.0.0    CimCmdlets
Cmdlet          Get-CimSession                                     1.0.0.0    CimCmdlets
Cmdlet          Import-BinaryMiLog                                 1.0.0.0    CimCmdlets
Cmdlet          Invoke-CimMethod                                   1.0.0.0    CimCmdlets
Cmdlet          New-CimInstance                                    1.0.0.0    CimCmdlets
Cmdlet          New-CimSession                                     1.0.0.0    CimCmdlets
Cmdlet          New-CimSessionOption                               1.0.0.0    CimCmdlets
Cmdlet          Register-CimIndicationEvent                        1.0.0.0    CimCmdlets
Cmdlet          Remove-CimInstance                                 1.0.0.0    CimCmdlets
Cmdlet          Remove-CimSession                                  1.0.0.0    CimCmdlets
Cmdlet          Set-CimInstance                                    1.0.0.0    CimCmdlets
```

Как и раньше, командлеты CIM позволяют вам работать с WMI, поэтому не путайтесь, если кто-то пытается выполнить запрос WMI с помощью командлетов CIM PowerShell.

Как я уже говорил, WMI — это отдельная технология в PowerShell, поэтому вы просто используете командлеты CIM для доступа к WMI. Вы можете найти прежний сценарий VBScript, в котором используется язык запросов WMI (WQL), позволяющий выполнить запрос WMI, как показано в следующем примере.

```vb
strComputer = "."
Set objWMIService = GetObject("winmgmts:" _
    & "{impersonationLevel=impersonate}!\\" & strComputer & "\root\cimv2")

Set colBIOS = objWMIService.ExecQuery _
    ("Select * from Win32_BIOS")

For each objBIOS in colBIOS
    Wscript.Echo "Manufacturer: " & objBIOS.Manufacturer
    Wscript.Echo "Name: " & objBIOS.Name
    Wscript.Echo "Serial Number: " & objBIOS.SerialNumber
    Wscript.Echo "SMBIOS Version: " & objBIOS.SMBIOSBIOSVersion
    Wscript.Echo "Version: " & objBIOS.Version
Next
```

Вы можете взять WQL-запрос из сценария VBScript и использовать его с командлетом `Get-CimInstance` без изменений.

```powershell
Get-CimInstance -Query 'Select * from Win32_BIOS'
```

```Output
SMBIOSBIOSVersion : 090006
Manufacturer      : American Megatrends Inc.
Name              : Intel(R) Xeon(R) CPU E3-1505M v5 @ 2.80GHz
SerialNumber      : 3810-1995-1654-4615-2295-2755-89
Version           : VRTUAL - 4001628
```

Это не тот способ, который я стандартно использую для запроса WMI с помощью PowerShell. Но он действительно работает, позволяя легко перенести существующие сценарии VBScript в PowerShell. При внесении скрипта из одной строки для запроса WMI я использую следующий синтаксис.

```powershell
Get-CimInstance -ClassName Win32_BIOS
```

```Output
SMBIOSBIOSVersion : 090006
Manufacturer      : American Megatrends Inc.
Name              : Intel(R) Xeon(R) CPU E3-1505M v5 @ 2.80GHz
SerialNumber      : 3810-1995-1654-4615-2295-2755-89
Version           : VRTUAL - 4001628
```

Если нужно указать только серийный номер, я могу передать выходные данные в `Select-Object` и указать только свойство **SerialNumber**.

```powershell
Get-CimInstance -ClassName Win32_BIOS | Select-Object -Property SerialNumber
```

```Output
SerialNumber
------------
3810-1995-1654-4615-2295-2755-89
```

По умолчанию существуют несколько свойств, извлекающихся в фоновом режиме, который никогда не используется.
Это действие не важно, если запрос инструментария WMI выполняется на локальном компьютере. Но когда вы приступаете к выполнению запросов с удаленных компьютеров, оно позволяет не только увеличить время обработки для возврата информации, но и получить по сети дополнительные ненужные данные. `Get-CimInstance` содержит параметр **Property**, который ограничивает извлекаемые данные. Это действие позволяет выполнять запрос к инструментарию WMI более эффективно.

```powershell
Get-CimInstance -ClassName Win32_BIOS -Property SerialNumber |
Select-Object -Property SerialNumber
```

```Output
SerialNumber
------------
3810-1995-1654-4615-2295-2755-89
```

Предыдущие результаты вернули объект. Для возврата простой строки используйте параметр **ExpandProperty**.

```powershell
Get-CimInstance -ClassName Win32_BIOS -Property SerialNumber |
Select-Object -ExpandProperty SerialNumber
```

```Output
3810-1995-1654-4615-2295-2755-89
```

Также для этого вы можете использовать пунктирный стиль синтаксиса. Это избавляет от необходимости передавать простую строку в `Select-Object`.

```powershell
(Get-CimInstance -ClassName Win32_BIOS -Property SerialNumber).SerialNumber
```

```Output
3810-1995-1654-4615-2295-2755-89
```

## <a name="query-remote-computers-with-the-cim-cmdlets"></a>Запрос с удаленных компьютеров с помощью командлетов CIM

Я использую PowerShell от имени локального администратора, который является пользователем домена. При попытке запросить данные с удаленного компьютера с помощью командлета `Get-CimInstance` появляется сообщение об ошибке с отказом в доступе.

```powershell
Get-CimInstance -ComputerName dc01 -ClassName Win32_BIOS
```

```Output
Get-CimInstance : Access is denied.
At line:1 char:1
+ Get-CimInstance -ComputerName dc01 -ClassName Win32_BIOS
+ ``````````````````````````````````````````````````````~~
    + CategoryInfo          : PermissionDenied: (root\cimv2:Win32_BIOS:String) [Get-CimI
   nstance], CimException
    + FullyQualifiedErrorId : HRESULT 0x80070005,Microsoft.Management.Infrastructure.Cim
   Cmdlets.GetCimInstanceCommand
    + PSComputerName        : dc01
```

Когда речь идет о PowerShell, многие люди сталкиваются с проблемами безопасности, но дело в том, что вы имеете точно такие же разрешения в PowerShell, как и в графическом интерфейсе пользователя. Ни больше ни меньше. Из предыдущего примера видно, что пользователь, запускающий PowerShell, не имеет прав на выполнение запроса WMI с сервера DC01. Я могу повторно запустить PowerShell от имени администратора домена, так как `Get-CimInstance` не содержит параметр **Credential**. Но это не совсем удобно, так как потом любая открытая в PowerShell программа будет запускаться с правами администратора домена. С точки зрения безопасности это действие может быть рискованным в зависимости от ситуации.

Используя принцип предоставления наименьших прав, я буду повышать уровень прав учетной записи администратора домена для каждой команды с помощью параметра **Credential**, если команда будет содержать такой же параметр. `Get-CimInstance` не содержит параметр **Credential**, поэтому решение для этого сценария заключается в том, чтобы сначала создать параметр **CimSession**. Затем я использую параметр **CimSession** для запроса WMI на удаленном компьютере вместо имени компьютера.

```powershell
$CimSession = New-CimSession -ComputerName dc01 -Credential (Get-Credential)
```

```Output
cmdlet Get-Credential at command pipeline position 1
Supply values for the following parameters:
Credential
```

Сеанс CIM сохранился в переменной с именем `$CimSession`. Заметьте, что командлет `Get-Credential` я указываю также в круглых скобках, которые позволяют выполнить его первым, предлагая мне ввести альтернативные учетные данные перед созданием нового сеанса. Позже при рассмотрении этой главы я покажу вам более эффективный способ указывать альтернативные учетные данные, а пока важно усвоить основной принцип, чтобы потом уметь разбираться с более сложными понятиями.

Сеанс CIM, созданный в предыдущем примере, теперь можно использовать с командлетом `Get-CimInstance` для запроса информации BIOS из инструментария WMI на удаленном компьютере.

```powershell
Get-CimInstance -CimSession $CimSession -ClassName Win32_BIOS
```

```Output
SMBIOSBIOSVersion : 090006
Manufacturer      : American Megatrends Inc.
Name              : Intel(R) Xeon(R) CPU E3-1505M v5 @ 2.80GHz
SerialNumber      : 0986-6980-3916-0512-6608-8243-13
Version           : VRTUAL - 4001628
PSComputerName    : dc01
```

Использование сеансов CIM вместо ввода имени компьютера дает несколько дополнительных преимуществ. При выполнении нескольких запросов к одному и тому же компьютеру использовать сеанс CIM более эффективно, чем вводить имя компьютера для каждого запроса. При создании сеанса CIM соединение настраивается только один раз.
Затем несколько запросов используют этот же сеанс для получения информации. При использовании имени компьютера необходимы командлеты для установки и разрыва соединения с каждым отдельным запросом.

Командлет `Get-CimInstance` по умолчанию использует протокол WSMan, то есть для подключения к удаленному компьютеру необходимо иметь PowerShell версии 3.0 или выше. По сути, важна не версия PowerShell, а версия стека. Версию стека можно определить с помощью командлета `Test-WSMan`.
Стек должен быть версии 3.0. Эту версию можно найти с помощью PowerShell версии 3.0 и выше.

```powershell
Test-WSMan -ComputerName dc01
```

```Output
wsmid           : http://schemas.dmtf.org/wbem/wsman/identity/1/wsmanidentity.xsd
ProtocolVersion : http://schemas.dmtf.org/wbem/wsman/1/wsman.xsd
ProductVendor   : Microsoft Corporation
ProductVersion  : OS: 0.0.0 SP: 0.0 Stack: 3.0
```

Прежние командлеты WMI используют протокол DCOM, совместимый с более старыми версиями Windows. Но в новых версиях Windows протокол DCOM обычно блокируется брандмауэром. Командлет `New-CimSessionOption` позволяет создать подключение протокола DCOM для использования вместе с `New-CimSession`. Это позволяет использовать командлет `Get-CimInstance` для взаимодействия с такими же старыми версиями Windows, как и у Windows Server 2000. Кроме того, это значит, что установка PowerShell не требуется на удаленном компьютере при использовании командлета `Get-CimInstance` с параметром CimSession, который настроен для использования протокола DCOM.

Создайте параметр протокола DCOM с помощью командлета `New-CimSessionOption` и сохраните его в переменной.

```powershell
$DCOM = New-CimSessionOption -Protocol Dcom
```

Для удобства вы можете сохранить в переменной свои учетные данные администратора домена или учетные данные с повышенными правами, чтобы постоянно не вводить их для каждой команды.

```powershell
$Cred = Get-Credential
```

```Output
cmdlet Get-Credential at command pipeline position 1
Supply values for the following parameters:
Credential
```

Я использую сервер SQL03 по управлением Windows Server 2008 (без R2). Это новейшая операционная система Windows Server, на которой средство PowerShell по умолчанию не установлено.

Создайте параметр **CimSession** на сервере SQL03, используя протокол DCOM.

```powershell
$CimSession = New-CimSession -ComputerName sql03 -SessionOption $DCOM -Credential $Cred
```

Заметьте, что в предыдущем примере я указал в этот раз переменную с именем `$Cred` в качестве значения параметра **Credential**, а не вводил ее вручную.

Выходные данные запроса одинаковые, независимо от используемого базового протокола.

```powershell
Get-CimInstance -CimSession $CimSession -ClassName Win32_BIOS
```

```Output
SMBIOSBIOSVersion : 090006
Manufacturer      : American Megatrends Inc.
Name              : Intel(R) Xeon(R) CPU E3-1505M v5 @ 2.80GHz
SerialNumber      : 7237-7483-8873-8926-7271-5004-86
Version           : VRTUAL - 4001628
PSComputerName    : sql03
```

Командлет `Get-CimSession` используется для просмотра подключенных в данный момент параметров **CimSession**, а также используемых ими протоколов.

```powershell
Get-CimSession
```

```Output
Id           : 1
Name         : CimSession1
InstanceId   : 80742787-e38e-41b1-a7d7-fa1369cf1402
ComputerName : dc01
Protocol     : WSMAN

Id           : 2
Name         : CimSession2
InstanceId   : 8fcabd81-43cf-4682-bd53-ccce1e24aecb
ComputerName : sql03
Protocol     : DCOM
```

Получите и сохраните оба ранее созданных параметра **CimSession** в переменной с именем `$CimSession`.

```powershell
$CimSession = Get-CimSession
```

Отправьте запрос с обоих компьютеров с помощью одной команды, при этом на первом должен использоваться протокол WSMan, а на втором — DCOM.

```powershell
Get-CimInstance -CimSession $CimSession -ClassName Win32_BIOS
```

```Output
SMBIOSBIOSVersion : 090006
Manufacturer      : American Megatrends Inc.
Name              : Intel(R) Xeon(R) CPU E3-1505M v5 @ 2.80GHz
SerialNumber      : 0986-6980-3916-0512-6608-8243-13
Version           : VRTUAL - 4001628
PSComputerName    : dc01

SMBIOSBIOSVersion : 090006
Manufacturer      : American Megatrends Inc.
Name              : Intel(R) Xeon(R) CPU E3-1505M v5 @ 2.80GHz
SerialNumber      : 7237-7483-8873-8926-7271-5004-86
Version           : VRTUAL - 4001628
PSComputerName    : sql03
```

Я написал несколько статей в блогах о командлетах WMI и CIM. В одной из самых полезных статей описывается созданная мной функция, которая позволяет автоматически определить нужный протокол WSMan или DCOM, а также настроить сеанс CIM, не указывая протокол вручную. Эта статья называется [PowerShell Function to Create CimSessions to Remote Computers with Fallback to Dcom][] (Использование функции PowerShell для создания параметров CimSession на удаленных компьютерах с помощью отката и DCOM).

Завершив работу с сеансами CIM, удалите их с помощью командлета `Remove-CimSession`. Чтобы удалить все сеансы CIM, передайте `Get-CimSession` в `Remove-CimSession`.

```powershell
Get-CimSession | Remove-CimSession
```

## <a name="summary"></a>Сводка

В этой главе вы узнали об использовании PowerShell для работы с инструментарием WMI на локальном и на удаленном компьютерах. Кроме того, вы узнали о том, как использовать командлеты CIM для работы с удаленными компьютерами с помощью протокола WSMan или DCOM.

## <a name="review"></a>Просмотр

1. Чем отличаются командлеты WMI и CIM?
1. В каком протоколе командлет `Get-CimInstance` используется по умолчанию?
1. В чем преимущества использования сеанса CIM перед вводом имени компьютера с помощью `Get-CimInstance`?
1. Как указать отличный от используемого по умолчанию альтернативный протокол вместе с `Get-CimInstance`?
1. Как закрыть или удалить сеансы CIM?

## <a name="recommended-reading"></a>Рекомендуем прочесть

- [about_WMI][]
- [about_WMI_Cmdlets][]
- [about_WQL][]
- [Модуль CimCmdlets][]
- [Видео: Использование командлетов CIM и сеансов CIM][]

<!-- link references -->
[about_WMI]: /powershell/module/microsoft.powershell.core/about/about_wmi
[about_WMI_Cmdlets]: /powershell/module/microsoft.powershell.core/about/about_wmi_cmdlets
[about_WQL]: /powershell/module/microsoft.powershell.core/about/about_wql
[Модуль CimCmdlets]: /powershell/module/cimcmdlets/
[Видео: Использование командлетов CIM и сеансов CIM]: https://mikefrobbins.com/2013/09/12/phillyposh-user-group-meeting-presentation-follow-up-powershell-second-hop-problem-with-cimsessions/
[PowerShell Function to Create CimSessions to Remote Computers with Fallback to Dcom]: https://mikefrobbins.com/2014/08/28/powershell-function-to-create-cimsessions-to-remote-computers-with-fallback-to-dcom/ (Использование функции PowerShell для создания параметров CimSession на удаленных компьютерах с помощью отката и DCOM).
