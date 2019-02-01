Этот документ является более развёрнутым вариантом документа подготовленого командой Microsoft и размещёного на  странице https://docs.microsoft.com/en-us/azure/marketplace/azure-partner-customer-usage-attribution. 

В данном примере используется 5.0 версия PowerShell и [PоwerShell модуль для Azure](https://docs.microsoft.com/en-us/powershell/azure/overview?view=azps-1.2.0) версии 1.1.0. 

# ARM шаблон и GUID
[ARM шаблон](https://docs.microsoft.com/en-us/azure/templates/) - это шаблон используемый [Azure Resource Manager](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview) для автоматического развёртывания ресурсов в подписке Azure. ARM шаблон позволяет описать ресурсы в формате [JSON](https://json.org/) (Java Script Object Notation). Больше информации о структуре шаблона можно найти [здесь](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates).

Каждое развёртывание в Azure идентифицируется по 3 параметрам: 
* ID подпискa;
* ресурснaя группа;
* название операции развёртывания (deployment).

При создании ARM шаблона в дополнение к уже описанным в шаблоне ресурсам добавляется ресурс типа  “Microsoft.Resources/deployments”, a GUID используется как название этого ресурса (см. картинку ниже).  Это название в дальнейшем используется как фильтр для отслеживания ресурсов запущенных создателем программного обеспечения в подписке клиента.

![alt text](https://github.com/LTUTE/ARM-ISV-GUID/blob/master/Pictures/delploy-resourse.jpg)

Тот же самый GUID можно использовать для развёртывания разных продуктов у клиента. Более удобная опция, когда каждый продукт использует свой уникальный GUID. Например, GUID1 для программы А, GUID2 для программы Б и т.д. Это позволяет получать более опрятные отчёты о потреблении ресурсов в [Cloud Partner Portal](https://cloudpartner.azure.com/).

# Процедура создания GUID

1.	Создаём GUID выбрав одну из ниже указанных опции:
* Веб [форма для генераторации GUID](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR3i8TQB_XnRAsV3-7XmQFpFUMVRVVFFLTDFLS0E2QzNYSkFZR1U3WVJCTSQlQCN0PWcu) - GUID высылается на указанную вами почту;
* PowerShell команда: 
```PowerShell 
PS > New-Guid 
```
* Linux и MacOS: 
```bash 
user:~$ uuidgen
```

# Регистрация GUID в Cloud Publisher Portal. 
Если нет учётной записи в Cloud Publisher Portal, то создаём новую запись. Процедура создания новой учётной записи описана [здесь](https://docs.microsoft.com/en-us/azure/marketplace/become-publisher). 


1. Подсоединяемся к [Cloud Partner Portal](https://cloudpartner.azure.com/). В  левом верхнем углу выбираем *“Publisher profile”*

![alt text](https://github.com/LTUTE/ARM-ISV-GUID/blob/master/Pictures/publisherprofile.png)

2. В секции *“Azure Application Usage Tracking GUIDs”* нажимаем ссылку “Add tracking GUID” и в появившееся окно добавляем сгенерированный GUID и короткое описание.

![alt text](https://github.com/LTUTE/ARM-ISV-GUID/blob/master/Pictures/App-usage-tracking-guid.png)

3. Сохраняем изменения.

![alt text](https://github.com/LTUTE/ARM-ISV-GUID/blob/master/Pictures/save-guid.png)

# Добавление GUID в существующую ресурсную группу с помощью PowerShell
Описанный ниже метод позволяет добавить GUID к уже сушествующим ресурсам. Метод основан на использовании функции [экспортирования ARM шаблона для ресурсной группы](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-export-template#export-the-template-from-resource-group) и [инкрементного внедерения](https://docs.microsoft.com/en-us/azure/azure-resource-manager/deployment-modes) ARM шаблона. 

__ВАЖНО!__ Этот метод не подходт для равёртываний групп в которых описано больше 200 ресурсов. Так же, в зависимости от того как изначально были развёрнуты ресурсы в Azure и были ли использованы расширения Азуре ([Azure VM/SQL extentions](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-tutorial-deploy-vm-extensions)], секреты, пароли и т.д, экспортированный шаблон может требовать большого количества модификаций и понимания того как работает ARM. Один из способов оценить количевство работы - это взглянуть на экспортированный шаблон и оценить количевство параметров с нулевым значением, зависимостей между ресурсами ( поля ```depends on ``` ) - эти значения надо будет модифицировать/удалить вручную. Так же, при возможности, запуск шаблона тестируем в тестовой среде.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "extensions_configureAppVM_storageAccountName": {
            "defaultValue": null, //нулевое значение
            "type": "SecureString"
        },
        "extensions_configureAppVM_storageAccountKey": {
            "defaultValue": null, //нулевое значение
            "type": "SecureString"
        },...
	"resources": [
        {...
            "dependsOn": [
                "[resourceId('Microsoft.Compute/virtualMachines', parameters('virtualMachines_appvm_0_name'))]",
                "[resourceId('Microsoft.Compute/virtualMachines', parameters('virtualMachines_appvm_1_name'))]"
            ]
	    }, ...
```
	
## Экспортирование шаблона через портал Azure
1.	Переходим на *Azure portal -> Resource Groups -> Интересующая нас группа*. Запоминаем регион в котором находится ресурсная группа.

![alt text](https://github.com/LTUTE/ARM-ISV-GUID/blob/master/Pictures/RG.png)
 
2.	В ресурсной группе переходим в секцию *“Settings” -> “Automation Scripts”*

 ![alt text](https://github.com/LTUTE/ARM-ISV-GUID/blob/master/Pictures/automationscript.png)
 
3.	Выбираем *“Download”*

 ![alt text](https://github.com/LTUTE/ARM-ISV-GUID/blob/master/Pictures/download_tempalte.png)
 
4.	Из присланного архива открываем документ под названием “Template.json”. Он содержит информацию о текущей конфигурации ресурсов в группе.
 ![alt text](https://github.com/LTUTE/ARM-ISV-GUID/blob/master/Pictures/templatezip.png)
5.	 В секцию ресурсы ("resources":)  добавляем код, и как значение ключа “name”, добавляем значение ключа с префиксом “pid-“, например: "pid-xxxxxxx-xxxx-xxxx-xxxx-xxxxxx93897". Сохраняем “template.json” в удобном для нас месте.
`# добавьте этот ресурс в секцию resources в mainTemplate.json шаблон, а сгенерированное значение GUID добавьте как значение в поле "name" `
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
## Запуск шаблона с помощью PowerShell

1.	Открываем PowerShell (команды в зависимости от версии PowerShell могут отличаться). Пример показан для версии PowerShell 5.0 с использованием Az модуля. 
2.	Внедряем Azure PowerShell модуль в режиме администратора: 
```PowerShell
PS > Install-Module -Name Az -AllowClobber
```
или в пользовательском режиме:
```PowerShell
Install-Module -Name Az -AllowClobber -Scope CurrentUser
```
3.	Подключаемся к Аzure: 
```PowerShell
PS > Connect-AzAccount
```
4.	По умолчанию, мы подключаемся в подписку по умолчанию (default subscription). При необходимости, переключаемся к подписке содержащей интересующие нас ресурсы: 
```PowerShell
PS > Select-AzSubscription -SubscriptionName "имя подписки" 
```
5.	Запускаем PowerShell скрипт для внедрения конфигурации в облако и подаём параметры 
  a.	имя ресурсной группы в которую хотим добавить GUID (в этом документе это “Kubespray”);
  b.	название операции внедрения – может быть любое название;
  c.	название региона в котором находится ресурсная группа.
```PowerShell
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$deploymentName = Read-Host -Prompt "Enter the name for this deployment"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"

New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment -Name $deploymentName -ResourceGroupName $resourceGroupName -TemplateFile "путь к файлу template.json" -Mode Incremental
```
6.	Получаем результат похожий на показанный ниже:

 ![alt text](https://github.com/LTUTE/ARM-ISV-GUID/blob/master/Pictures/ps-arm-deploy.png)
 
 #Тестирование полученных результатов
1.	С помощью PowerShell скрипта тестируем какие из ресурсов идентифицируются по GUID. Исходный код скрипта можно найти [здесь](https://gist.github.com/stuartleeks/ed84b0cc242b0abed85a9aea0b032fc3). GUID и название ресурсной группы вводятся как параметры скрипта (см. ниже). Используем ту же Azure PowerShell сессию, что и для запуска предидущих команд PowerShell.

 ![alt text](https://github.com/LTUTE/ARM-ISV-GUID/blob/master/Pictures/ps-guid-test.png)
 
2. Получаем результат аналогичный показанному ниже, в котором видны названия ресурсов.
	
 ![alt text](https://github.com/LTUTE/ARM-ISV-GUID/blob/master/Pictures/ps-guid-test-result.png)


