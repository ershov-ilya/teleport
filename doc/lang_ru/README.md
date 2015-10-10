# Teleport

Teleport является расширяемым скриптовым инструментом для работы с одной или несколькими локальными установками MODx Revolution.

Teleport в настоящее время функционирует приемущественно в качестве упаковочного инструмента который расширяет Транспортный API MODX'а и даёт команды для извлечения и инъекций кастомных слепков сайтов на MODX.Телепорт в настоящее время функции, прежде всего, в качестве упаковочного инструментария, который расширяет API, MODX транспорта и обеспечивает команды для извлечения и инъекционных настраиваемые снимки развертывания MODX. Но он может быть легко расширен для выполнения бесконечного множества действий, связанных с MODX.


## Требования

Для того, чтобы использовать телепорт, ваша рабочая среда должна, по крайней мере соответствовать следующим требованиям:

* PHP >= 5.4
* MODX Revolution >= 2.1 (MySQL)

Вы также должны иметь возможность запускать PHP с помощью CLI SAPI.

_ПРИМЕЧАНИЕ: В настоящее время, все варианты шаблонов извлечения Teleport поддерживают только MySQL инсталяции MODX Revolution._

Использование в среде Linux с расширением PHP POSIX может дать возможность использовать передовые функции переключения пользователелей.

Teleport стремится быть мультиплатформенным инструментом, и в настоящее время работает одинаково хорошо и в Linux и в OS X окружении. Поддержка на платформе Windows в настоящее время неизвестна - требуются разработчики в среде Windows.


## Установка

Есть несколько методов установки Teleport. Самый простой пууть начать - это установка PHAR дистрибутива Teleport.

_ВАЖНО: Используя любой из способов установки, убедитесь, что вы выполняете Teleport под тем же пользователем, под которым работает PHP когда исполняется веб-сервром. В противном случае может ваш MODx сайт может быть повреждён путем инъекии и/или кэширования файлов с некорректным владельцем файлов._

### Скачиивание и установка Phar

Создайте рабочую директорию для Teleport'а и перейдите в неё:

    mkdir ~/teleport/ && cd ~/teleport/

Скачайте последнюю версию [`teleport.phar`](http://modx.s3.amazonaws.com/releases/teleport/teleport.phar "teleport.phar") дистрибутива Teleport в вашу рабочую директорию Teleport'а.

Создайте **Профиль** MODX сайта:

    php teleport.phar --action=Profile --name="MyMODXSite" --code=mymodxsite --core_path=/path/to/mysite/modx/core/ --config_key=config

**Извлеките Слепок** MODX сайта который вы только что _профилировали_:

    php teleport.phar --action=Extract --profile=profile/mymodxsite.profile.json --tpl=phar://teleport.phar/tpl/develop.tpl.json


### Другие методы установки

Как вариат, вы можете установить Teleport из исходников с использованием [Composer'а](http://getcomposer.org/). Изучите больше об использовании [git clone](doc/install/git-clone.md) или [release archive](doc/install/releases.md).

_ВАЖНО: Если вы хотите использовать Teleport HTTP Server вы не можете использовать дистрибутив Phar. Вам ПРИДЁТСЯ использовать один из других методов установки._

### Teleport в вашем PATH

С любым типом установки вы можете создать исполнимую символьную ссылку и назвать её **teleport** указывающую на bin/teleport, или прямо на teleport.phar. Вы сможете просто писать `teleport` вместо `bin/teleport` или `php teleport.phar` для вызова приложения teleport.


## Базовое использование

Во всех случаях используйте teleport, в зависимости от того, как вы установили приложение. Например, если вы установили из исходников, пишите `bin/teleport` вместо `php teleport.phar`; если вы создали символьную ссылку на teleport.phar, пишите `teleport` вместо `php teleport.phar` во всех примерах команд. Последующие примеры предполагают, что вы установили дистрибутив teleport.phar.

_ПРИМЕЧАНИЕ: **Перед** использованием Teleport на сайте MODX, вам необходимо **создать Teleport Профиль** из установленного сайта._

### Create a MODX Site Profile

You can create a Teleport Profile of an existing MODX site using the following command:

    php teleport.phar --action=Profile --name="MySite" --code=mysite --core_path=/path/to/mysite/modx/core/ --config_key=config

The resulting file would be located at profile/mysite.profile.json and could then be used for Extract or Inject commands to be run against the site represented in the profile.

Learn more about [Teleport Profiles](doc/use/profile.md).

### Extract a Snapshot of a MODX Site

You can Extract a Teleport snapshot from a MODX site using the following command:

    php teleport.phar --action=Extract --profile=profile/mysite.profile.json --tpl=phar://teleport.phar/tpl/develop.tpl.json

The snapshot will be located in the workspace/ directory if it is created successfully.

You can also Extract a Teleport snapshot and push it to any valid stream target using the following command:

    php teleport.phar --action=Extract --profile=profile/mysite.profile.json --tpl=phar://teleport.phar/tpl/develop.tpl.json --target=s3://mybucket/snapshots/ --push

In either case, the absolute path to the snapshot is returned by the process as the final output. You can use this as the path for an Inject source.

_ПРИМЕЧАНИЕ: The workspace copy is removed after it is pushed unless you pass --preserveWorkspace to the CLI command._

Learn more about the [Teleport Extract](doc/use/extract.md) Action.

### Inject a Snapshot into a MODX Site

You can Inject a Teleport snapshot from any valid stream source into a MODX site using the following command:

    php teleport.phar --action=Inject --profile=profile/mysite.profile.json --source=workspace/mysite_develop-120315.1106.30-2.2.1-dev.transport.zip

_ПРИМЕЧАНИЕ: If the source is not within the workspace/ directory a copy will be pulled to that location and then removed after the Inject completes unless --preserveWorkspace is passed._

Learn more about the [Teleport Inject](doc/use/inject.md) Action.

### UserCreate

You can create a user in a profiled MODX site using the following command:

    php teleport.phar --action=UserCreate --profile=profile/mysite.profile.json --username=superuser --password=password --sudo --active --fullname="Test User" --email=testuser@example.com

_ПРИМЕЧАНИЕ: This uses the security/user/create processor from the site in the specified profile to create a user, and the action accepts any properties the processor does._

Learn more about the [Teleport UserCreate](doc/use/user-create.md) Action.


## Get Started

Learn more about Teleport in the [documentation](doc/index.md).

## License

Teleport is Copyright (c) MODX, LLC

For the full copyright and license information, please view the [LICENSE](./LICENSE "LICENSE") file that was distributed with this source code.
