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
* [Архитектура MVP](#Архитектура MVP)
   * [Описание протоколов](## Описание протоколов)
   * [Реализации протоколов](## Реализации протоколов)
   * [Порядок расположения протоколов](## Порядок расположения протоколов)

# Git-Flow
## Работа с коммитами
Каждый коммит должен содержать в себе логически законченное изменение. В него не должны попадать какие либо другие изменения.\
Комментарий к коммиту должен быть написан на русском языке, содержать ID в квадратных скобках, название задачи и краткое описание изменения, отделённое пустой строкой.

**Пример комментария:**
> [EW_IOS-98] требуется собрать приложение с использованием IBM Maas 360 SDK 2.95
> 
> `- Обновил структуру и настройки проекта`\
> `- Обновил фреймворк MaaS`

## Работа с ветками

**Важно:** git плохо работает с нашими буквами "й" и "ё",  следует избегать их использования в именах веток.

### master
Основная стабильная ветка. В неё мёржатся ветки релизов после стабилизации.

### develop
Основная ветка разработки. В неё мёржатся ветки фич, рефакторингов и багов после тестирования.

### features/<feature_id_in_issue_tracker>_<feature_name_in_issue_tracker>
Создаётся под конкретную задачу на разработку фичи в Джире.

**Пример:**
> features/EW_IOS-98_Реализовать_независимую_отправку_и_получение_данных_с_сервера

Создаётся из [develop](#develop), мёржится в [develop](#develop).

После завершения работы над задачей необходимо создать [мёрж реквест](#Создание-мёрж-реквестов) из ветки с фичей в ветку develop.\
Ветка мёржится после прохождения Code Review и цикла тестирования.

### refactoring/<refactoring_name>
Создаётся под изменение кода без расширения функциональности.

Создаётся из [develop](#develop), мёржится в [develop](#develop)

### releases/release-<release_version>
В этой ветке стабилизируется версия, которая в итоге будет передана Заказчику.\
В неё мёржится **только** багфикс.

Создаётся из [develop](#develop), мёржится в [develop](#develop) и [master](#master)

### bugs/<bug_id_in_issue_tracker>_<bug_name_in_issue_tracker>
Создаётся и мёржится из любого типа ветки. Зависит от того, куда необходимо внести исправление. Уточнить у аналитика.

**Примеры:**
> bugs/EW_IOS-98_исправить_падение

После завержшния работы над задачей необходимо создать [мёрж реквест](#Создание-мёрж-реквестов) из ветки с багом в ветку, из которой была создана ветка с багом.\
Ветка мёржится после прохождения Code Review и цикла тестирования.

## Создание мёрж реквестов

Заголовок должен содержать ID в квадратных скобках и название задачи.\
При создании мёрж реквеста должна быть включена опция "Delete source branch when merge request is accepted."

# Архитектура MVP

## Описание протоколов

Каждый `view сontroller`, `cell` или сложная `view` должны образовывать свой `MVP` модуль (`Model-View-Presenter`). У этого модуля должно быть фиксированное название. 

**Пример:**
Будем делать список событий. MVP модуль будет называться `EventList`

Взаимодействие между `view` и `presenter` описывается двумя протоколами, по одному на каждую сущность. Каждый протокол состоит из двух секций со свойствами и методами, разделенных с помощью `// MARK: -`.

**Протокол для `view`:**
```swift
protocol EventListViewProtocol: AnyObject {

    // MARK: - Properties

    var presenter: EventListPresenterProtocol! { get set }

    // MARK: - Methods

    func showLoadingIndicator()
    func hideLoadingIndicator()

    func showError(title: String, message: String)

    func reloadEventList()
}
```

**Важно:** Методы в протоколе для `view` желательно начинать с `reload`, `show` или `hide`. 

**Протокол для `presenter`:**
```swift
protocol EventListPresenterProtocol: AnyObject {

    // MARK: - Properties

    var view: EventListViewProtocol! { get set }

    var numberOfEvents: Int { get }

    // MARK: - Methods

    func viewDidLoad()
    func viewDidSelectEvent(atIndex index: Int)

    func evenListCellPresenter(atIndex index: Int) -> EventListCellPresenterProtocol?
}
```

**Важно:** Во избежании циклических ссылок ссылка с `presenter` на `view` слабая!

Методы в протоколе для `presenter` делятся на 3 логические группы:

- `View` говорит, что с ней произошло:

**Пример:**
```swift
func viewDidLoad()
func viewDidSelectEvent(atIndex index: Int)
```
**Важно:** Начинается со слова `view`!

- Методы порождающие другие презентеры:

**Пример:**
```swift
func eventListCellPresenter(atIndex index: Int) -> AssignmentCellPresenterProtocol?
```
**Важно:** Только `presenter` может порождать другие `presenter`ы (или `coordinator` в случае с  MVPc (Model - View - Presenter c coordinator'ами).

Получение дочерних презентеров должно быть оформлено как `func`. Использование `computed porperties` недопустимо!

- Редкие методы с параметрами запрашивающие для `view` подготовленные данные. Пример для `UISegmentedControl`:

**Пример:**
```swift
func eventListSegmentTitle(for index: Int) -> String?
```

**Важно:** Методы в протоколе и в файлах реализации должны располагаться в порядке выше. 

## Реализация протоколов

При реализации протокола для `view`, выносим указание следованию протоколу в `extension`, потому что зачастую еще идет наследование от класса из `UIKit`.

**Пример:**
```swift
extension EventListViewController: EventListViewProtocol {}
```

При реализации протокола для `Presenter`, указание следованию протоколу сразу в указании класса.

**Пример:**
```swift
class EventListPresenter: EventListPresenterProtocol {}
```

В файлах `view` и `presenter` весь код разделен на секции с помощью `// MARK: -`

**Важно:** За логику layout'а и как именно отображать подготовленные данные отвечает `view`.

**Важно:** При возвращении в `presenter` на главный поток желательно использовать `CFRunLoopPerformBlock(CFRunLoopGetMain(), CFRunLoopMode.defaultMode.rawValue) {}`, где это возможно.

## Порядок расположения протоколов

Описание протоколов выносится на тот же уровень что и Storyboard модуля, в отдельный файл `НазваниеМодуляProtocols`. Протоколы в нём группируются по модулям, `view` рядом с `presenter`, сортировка от основных к вспомогательным.

**Пример:**
```swift
protocol MainViewProtocol: AnyObject { }
protocol MainPresenterProtocol: AnyObject { }
protocol HeaderProtocol: AnyObject { }
protocol HeaderPresenterProtocol: AnyObject { }
protocol CellViewProtocol: AnyObject { }
protocol CellPresenterProtocol: AnyObject { }
```

**Важно:** Протоколы делегатов распологаются под протоколами делегирующих.





