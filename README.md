## Содержание
* [Git-Flow](#Git-Flow)
   * [Работа с коммитами](#Работа-с-коммитами)
   * [Работа с ветками](#Работа-с-ветками)
      * [master](#master)
      * [develop](#develop)
      * [features](#featuresfeature_id_in_issue_tracker_feature_name_in_issue_tracker)
      * [refactoring](#refactoringrefactoring_name)
      * [releases](#releasesrelease-release_version)
      * [bugs](#bugsbug_id_in_issue_tracker_bug_name_in_issue_tracker)
   * [Создание мёрж реквестов](#Создание-мёрж-реквестов)

# Git-Flow
## Работа с коммитами
Каждый коммит должен содержать в себе логически законченное изменение. В него не должны попадать какие либо другие изменения.
Комментарий к коммиту должен быть написан на русском языке, содержать ID в квадратных скобках, название задачи и краткое описание изменения, отделённое пустой строкой.

Пример комментария:
> [EW_IOS-98] требуется собрать приложение с использованием IBM Maas 360 SDK 2.95
> 
> - Обновил структуру и настройки проекта
> - Обновил фреймворк MaaS

## Работа с ветками

**Важно:** git плохо работает с нашими буквами "й" и "ё",  следует избегать их использования в именах веток.

### master
Основная стабильная ветка. В неё мёржатся ветки релизов после стабилизации.

### develop
Основная ветка разработки. В неё мёржатся ветки фич, рефакторингов и багов после тестирования.

### features/<feature_id_in_issue_tracker>_<feature_name_in_issue_tracker>
Создаётся под конкретную задачу на разработку фичи в Джире.

Пример: 
> features/EW_IOS-98_Реализовать_независимую_отправку_и_получение_данных_с_сервера

Создаётся из [develop](#develop), мёржится в [develop](#develop).

После завершения работы над задачей необходимо создать [мёрж реквест](#Создание-мёрж-реквестов) из ветки с фичей в ветку develop. 
Ветка мёржится после прохождения Code Review и цикла тестирования.

### refactoring/<refactoring_name>
Создаётся под изменение кода без расширения функциональности.

Создаётся из [develop](#develop), мёржится в [develop](#develop)

### releases/release-<release_version>
В этой ветке стабилизируется версия, которая в итоге будет передана Заказчику. 
В неё не мёржится только багфикс.

Создаётся из [develop](#develop), мёржится в [develop](#develop) и [master](#master)

### bugs/<bug_id_in_issue_tracker>_<bug_name_in_issue_tracker>
Создаётся и мёржится из любого типа ветки. Зависит от того, куда необходимо внести исправление. Уточнить у аналитика.

Примеры:
> bugs/EW_IOS-98_исправить_падение

После завержшния работы над задачей необходимо создать [мёрж реквест](#Создание-мёрж-реквестов) из ветки с багом в ветку, из которой была создана ветка с багом.
Ветка мёржится после прохождения Code Review и цикла тестирования.

## Создание мёрж реквестов

Заголовок должен содержать ID в квадратных скобках и название задачи
При создании мёрж реквеста должна быть включена опция "Delete source branch when merge request is accepted."
