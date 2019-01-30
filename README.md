Этот документ является более развёрнутым вариантом документа подготовленого командой Microsoft и размещёного на  странице https://docs.microsoft.com/en-us/azure/marketplace/azure-partner-customer-usage-attribution. 

# ARM шаблон и GUID
[ARM шаблон](https://docs.microsoft.com/en-us/azure/templates/) - это шаблон используемый [Azure Resource Manager](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview) для автоматического развёртывания ресурсов в подписке Azure. ARM шаблон позволяет описать ресурсы в формате [JSON](https://json.org/) (Java Script Object Notation).
Каждое развёртывание в Azure идентифицируется по 3 параметрам: 
* ID подпискa
* ресурснaя группа
* название операции развёртывания (deployment).

При создании ARM шаблона в дополнение к уже описанным ресурсам добавляется ресурс типа  “Microsoft.Resources/deployments”, a GUID используется как название этого ресурса (см. картинку ниже).  Это название в дальнейшем используется как фильтр для отслеживания ресурсов запущенных создателем программного обеспечения в подписке клиента.

`# добавьте этот ресурс в секцию resources в mainTemplate.json шаблон, а сгенерированное значение GUID добавьте как значение в поле "name" `
```json
{ 
    "apiVersion": "2018-02-01",
    "name": "pid-XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX",
    "type": "Microsoft.Resources/deployments",
    "properties": {
        "mode": "Incremental",
        "template": {
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
            "resources": []
        }
    }
} 
```
Тот же самый GUID можно использовать для развёртывания разных продуктов у клиента. Более удобная опция, когда каждый продукт использует свой уникальный GUID. Например, GUID1 для программы А, GUID2 для программы Б и т.д. Это позволяет получать более опраятные отчёты о потреблении ресурсов в [Cloud Partner Portal](https://cloudpartner.azure.com/).

# Процедура создания GUID

1.	Создаём GUID выбрав одну из удобных опции:
* Веб [форма для генераторации GUID](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR3i8TQB_XnRAsV3-7XmQFpFUMVRVVFFLTDFLS0E2QzNYSkFZR1U3WVJCTSQlQCN0PWcu) - GUID высылаетсыа на указанную вами почту;
* PowerShell команда: 
```PowerShell 
PS > New-Guid 
```
* Linux и MacOS: 
```bash 
user:~$ uuidgen
```

# Регистрируем GUID в Cloud Partner Portal. 
1. В  левом верхнем углу выбираем *“Publisher profile”*

![alt text](https://github.com/LTUTE/ARM-ISV-GUID/blob/master/Pictures/publisherprofile.png)

2. В секции *“Azure Application Usage Tracking GUIDs”* нажимаем ссылку “Add tracking GUID” и в появившееся окно добавляем сгенерированный GUID и короткое описание.
![alt text](https://github.com/LTUTE/ARM-ISV-GUID/blob/master/Pictures/App-usage-tracking-guid.png)
3. Сохраняем изменения.

# Добавление GUID в существующую ресурсную группу с помощью PowerShell
Предлогаемый способ основан на В данном примере используется 5.0 версия PowerShell и PоwerShell модуль для Azure версии 1.1.0. 

1.	Переходим на *Azure portal -> Resource Groups -> Интересующая нас группа*. Запоминаем регион в котором находится ресурсная группа.
 
2.	В ресурсной группе переходим в секцию *“Settings” -> “Automation Scripts”*
 
3.	Выбираем *“Download”*
 
4.	Из присланого архива открываем документ под названием “Template.json”. Он содержит информацию о текущей конфигурации ресурсов в группе.
 
5.	 В секцию ресурсы ("resources":)  добавляем код, и как значение ключа “name”, добавляем значение ключа с префиксом “pid-“, например: "pid-xxxxxxx-xxxx-xxxx-xxxx-xxxxxx93897". Сохраняем “template.json” в удобном для нас месте.
```json
       { 
            "apiVersion": "2018-02-01",
            "name": "pid-xxxxxxx-xxxx-xxxx-xxxx-xxxxxx93897", 
            "type": "Microsoft.Resources/deployments",
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "resources": []
                }
            }
        },
```
6.	Открываем PowerShell (команды в зависимости от версии PowerShell могут отличаться). Пример показан для версии PowerShell 5.0 с использованием Az модуля. 
a.	Внедряем Azure PowerShell модуль в режиме администратора: 
```PowerShell
PS > Install-Module -Name Az -AllowClobber
```
или пользовательском режиме:
```PowerShell
Install-Module -Name Az -AllowClobber -Scope CurrentUser
```
b.	Подключаемся к Аzure: 
```PowerShell
Connect-AzAccount
```
c.	По умолчанию вы подключитесь в вашу подписку по умолчанию, при необходимости переключитесь к подписке содержащей интересующие вас ресурсы: 
```PowerShell
Select-AzSubscription -SubscriptionName "имя подписки" 
```
d.	Запускаем PowerShell скрипт для внедрения конфигурации в облако и подаём параметры 
i.	имя ресурсной группы в которую хотим добавить GUID, в описанном в этом документе это “Kubespray”
ii.	название операции внедрения – может быть любое название
iii.	название региона в котором находится ресурсная группа.
```PowerShell
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$deploymentName = Read-Host -Prompt "Enter the name for this deployment"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"

New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment -Name $deploymentName -ResourceGroupName $resourceGroupName -TemplateFile "путь к файлу template.json" -Mode Incremental
```
e.	Получаем результат похожий на указанный ниже:
 
9.	С помощью PowerShell скрипта тестируем какие из ресурсов идентифицируются по GUID. Исходный код скрипта можно найти [здесь](https://gist.github.com/stuartleeks/ed84b0cc242b0abed85a9aea0b032fc3).  GUID и название ресурсной группы вводятся как параметры скрипта. Используем ту же PowerShell сессию, что и для запуска предидущих команд PowerShell. 
 
	Получаем результат аналогичный показанному ниже, в котором видны названия ресурсов.
 


