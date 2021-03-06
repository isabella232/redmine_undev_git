# RedmineUndevGit

[![Build Status](https://travis-ci.org/Restream/redmine_undev_git.png)](https://travis-ci.org/Restream/redmine_undev_git)
[![Code Climate](https://codeclimate.com/github/Restream/redmine_undev_git.png)](https://codeclimate.com/github/Restream/redmine_undev_git)

## Описание

Плагин UndevGit добавляет в redmine новый тип репозитория UndevGit.
UndevGit умеет делать все то, что делает Git репозиторий, но вдобавок он
добавляет возможность работы с удаленными репозиториями и добавляет
хуки для репозиториев UndevGit.

В процессе чтения репозитория UndevGit клонирует удаленный или локальный
репозиторий и в дальнейшем работает с созданной копией.

## Установка и настройка

### Установка

 1. Скопировать каталог с плагином в plugins
 2. bundle exec rake redmine:plugins:migrate
 3. bundle install в корне редмайна
 4. Перезапустить redmine

### Настройка

В административном меню на вкладке *Repositories* в группе *Enabled SCM*
необходимо включить UndevGit.

В настройках плагина можно указать максимальное количество веток отображаемых
на странице тикета в разделе связанных ревизий (*Associated Revisions*).
Пустое значение или 0 означает отуствие ограничений.
Настройка хуков описывается ниже.

В настройках инсталяции редмайна, в файле `config/configuration.yml` можно указать
папку для хранения локальных копий репозиториев,
указав значение для ключа `scm_repo_storage_dir` . По умолчанию это папка `repos`
в корне редмайна.
Локальные копии репозиториев складываются в папку `repos\[projectidentifier]\[repository_id]`
После удаления репозитория папки удаляются.

## Хуки

Назначение хуков - гибко настраивать как и в каких случаях изменять тикет.
Можно настроить выполнение хуков для определенных веток, проекта и даже отдельного репозитория.
Таким образом появляются глобальные хуки которые срабатывают для всех репозиториев,
хуки проекта - которые срабатывают для всех репозиториев проекта и хуки репозитория.

Если в хуке стоит ветка "*" - то хук применяется один раз при попадании коммита в репозиторий,
а если указаны ветки, то при каждом попадании коммита в ветку (один раз для одной ветки).

### Настройка

Хуки заменяют функционал редмайна, который позволяет закрывать тикеты
используя специальные ключевые слова. Соответственно ключевые слова указанные
в настройках редмайна на вкладке на вкладке *Repository*, (атрибут *Fixing keywords*)
не используются. Для каждого хука нужно определять собственный набор ключевых слов.

Хуки настраиваются в двух местах. Глобальные хуки управляются в настройках редмайна
(пункт меню *Global hooks* над *Plugins*).
Хуки проекта и хуки репозитория доступны в меню настроек проекта на вкладке *Hooks*.

TODO: Настройка чтения реп при обращении (почему лучше ее отключать)

### Приоритеты хуков

Приоритеты хуков определяются так: наивысший приоритет имеют хуки репозитория,
затем хуки проекта и наменьший приоритет у глобальных хуков.
Внутри типов хуки также имееют приоритет, который можно при необходимости изменить.
В случае если для коммита найдено несколько хуков,
выполнен будет только один - с наивысшим приоритетом.

Отдельно стоит отметить что хуки применяемые для всех веток имееют более низкий
приоритет, чем хуки в которых явно перечислены ветки.

### Изменения в журнале

Изменения тикета отражаются в журнале тикета. В случае отсутствия фактических
изменений запись в журнале тикета не появляется.

### Примеры

#### Пример 1

Например: в репозиторий пушится коммит который виден из двух веток (feature, develop)
Есть настроенные хуки:
Хук1 ветки: '*'
Хук2 ветки: 'feature'
В этом случае выполнится только хук2.

#### Пример 2

Допустим есть коммит A с текстом 'fix #1' и есть хуки:
Хук1 глобальный, ветки: '*'
Хук2 глобальный, ветки: 'master'
Хук3 проектный, ветки: 'develop,staging'
Хук4 для репы, ветки: 'staging'
Хук5 для репы, ветки: 'feature'

1-ый пуш: коммит A в ветке feature: выполняется Хук5
2-ой пуш: мержим ветку feature в staging: выполняется Хук4
3-ий пуш: мержим feature в develop: выполняется Хук3
3-ий пуш: мержим feature в master: выполняется Хук2

Хук1 сработает если например запушим коммит B в ветке featureX

## Прикрепление коммитов к тикету

Для того чтобы привязать коммит к тикету, достаточно указать ключевое слово и
номер тикета с решеткой, например так: `refs #124`.
Ключевые слова задаются в настройках редмайна на вкладке *Repository*,
(атрибут *Referencing keywords*).
Если указать '*' в качестве ключевого слова, то достаточно просто указать номер
тикета с решеткой чтобы привязать коммит к тикету.

## Перемещение коммитов (rebase)

UndevGit позволяет определить коммиты которые были перемещены с помощью `git rebase`.
"Старые" коммиты не удаляются из списка чейнджсетов, но помечаются специальной
иконкой со ссылкой на "новый" коммит. Аналогично и "новые" коммиты помечаются иконкой
со ссылкой на "старый" коммит.
Метка видна при просмотре списка коммитов и при просмотре отдельного коммита.
Изменения тикетов повторно не выполняются, но ссылки в Associated Revisions меняются.
Также не учитываются повторно и таймлоги.

## Обновление репозитория по веб хуку

TODO

### Глобальная настройка

### Настройка отдельной репы

### Запуск обновления из крона

Repository.fetch_changeset пропускает репы обновляемые по веб-хуку

## Тестирование

Распаковать тестовые репозитории
    rake test:scm:setup:undev_git

Подготовить БД
    rake RAILS_ENV=test db:drop db:create db:migrate redmine:plugins:migrate

Запустить тесты плагина redmine_undev_plugin
    rake RAILS_ENV=test NAME=redmine_undev_git redmine:plugins:test

## TODO:

Рейк таски: перенос репозиториев Git в UndevGit
Дополнительные поля в чейнджсет
Последовательность чтения (подбробно)
Особенности первого чтения больших реп (настройка chunk_size)
