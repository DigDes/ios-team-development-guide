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

Каждый `view сontroller`, `ячейка` или сложная `view` должны образовывать свой `MVP` модуль (`Model-View-Presenter`). Взаимодействие между `view` и `presenter` описывается двумя протоколами, по одному на каждую сущность:

```swift
protocol AssignmentListViewProtocol: AnyObject {

    // MARK: - Properties

    var presenter: AssignmentListPresenterProtocol! { get set }

    // MARK: - Methods

    func showAssignment()

    func showLoadingIndicator(_ show: Bool)
    func showError(title: String, message: String)

    func reloadAssignments()
}

// MARK: -

protocol AssignmentListPresenterProtocol: AnyObject {

    // MARK: - Properties

    var view: AssignmentListViewProtocol! { get set }

    var numberOfAssignments: Int { get }

    // MARK: - Methods

    func viewDidLoad()
    func viewDidSelectAssignment(atIndex index: Int)

    func assignmentCellPresenter(atIndex index: Int) -> AssignmentCellPresenterProtocol?
}
```
**Важно:** Во избежании циклических ссылок ссылка с `presenter` на `view` слабая и помечается как  `weak`

Методы в протоколе для `view` желательно начинать с `reload` или `show`. 

Методы в протоколе для `presenter` делятся на 3 логические группы:

- `View` говорит, что с ней произошло:

```swift
func viewWillAppear()
func viewWillDisplayItem(at indexPath: IndexPath)
func viewDidEndScrolling()
```
**Важно:** Начинается со слова `view`!

- Методы порождающие другие презентеры:

```swift
func collegialBodyListPresenter() -> CollegialBodyListPresenterProtocol?
func assignmentCellPresenter(atIndex index: Int) -> AssignmentCellPresenterProtocol?
func materialActionCellPresenter(atIndex index: Int) -> MaterialActionListCellPresenter?
```
**Важно:** Только `presenter` может порождать другие `presenter`ы. Получение дочерних презентеров должно быть оформлено как `func`. Использование `computed porperties` недопустимо!

- Редкие методы с параметрами запрашивающие для `view` подготовленные данные. Пример для `UISegmentedControl`:

```swift
func eventListSegmentTitle(for index: Int) -> String?
```

**Важно:** Методы в протоколе и в файлах реализации должны располагаться в порядке выше. 

## Реализация протоколов

Все сущности должны быть разделены `// MARK: -`:

```swift
class AssignmentListViewController: UIViewController {

    // MARK: - IBOutlets

    @IBOutlet private weak var collegialBodyLabel: UILabel!

    @IBOutlet private weak var filterAllAssignmentsButton: UIButton!
    @IBOutlet private weak var filterMineAssignmentsButton: UIButton!
    @IBOutlet private weak var filterCollegialBodyButton: UIButton!

    @IBOutlet private weak var collegialBodyButton: UIButton!
    @IBOutlet private weak var calendarButton: UIButton!
    @IBOutlet private weak var calendarCloseViewContainer: UIView!
    @IBOutlet private weak var calendarContainerStackView: UIStackView!

    @IBOutlet private weak var visualEffectView: UIVisualEffectView!
    @IBOutlet private weak var assignmentsCollectionView: UICollectionView!

    @IBOutlet private weak var loadingIndicatorContrainerView: UIView!
    @IBOutlet private weak var loadingActivityIndicatorView: UIActivityIndicatorView!

    @IBOutlet private weak var calendarContainerTrailingConstraint: NSLayoutConstraint!

    // MARK: - Properties

    var presenter: AssignmentListPresenterProtocol!

    private let cellIdentifier: String = "AssignmentCollectionViewCell"

    private var assignmentCellGap: CGFloat {
        return traitCollection.horizontalSizeClass == .regular ? 24.0 : 8.0
    }

    private var calendarAnimationDuration: TimeInterval = 0.33

    private var isCalendarAnimating: Bool = false
    private var isCalendarExpanded: Bool = false

    // MARK: - Lifecycle

    override func viewDidLoad() {
        super.viewDidLoad()
        prepareView()
        presenter.viewDidLoad()
    }

    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        navigationController?.setNavigationBarHidden(true, animated: animated)
        navigationController?.setToolbarHidden(true, animated: animated)
    }

    override func viewWillLayoutSubviews() {
        super.viewWillLayoutSubviews()
        assignmentsCollectionView.collectionViewLayout.invalidateLayout()
    }

    override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        switch segue.destination {
        case let calendarVC as CalendarViewController:
            calendarViewController = calendarVC
            calendarViewController?.delegate = self
        case let collegialBodyListVC as CollegialBodyListViewController:
            guard let sender = sender as? UIView else { fatalError("Sender не является UIView") }
            collegialBodyListVC.popoverPresentationController?.sourceView = sender
            collegialBodyListVC.popoverPresentationController?.sourceRect = sender.bounds
            collegialBodyListVC.presenter = presenter.collegialBodyListPresenter
            collegialBodyListVC.presenter.view = collegialBodyListVC
            collegialBodyListViewController = collegialBodyListVC
        case let meetingVC as MeetingViewController:
            meetingVC.presenter = presenter.meetingPresenter
            meetingVC.presenter.view = meetingVC
        case let commentNC as UINavigationController where commentNC.viewControllers.first is CommentViewController:
            guard let commentVC = commentNC.viewControllers.first as? CommentViewController else { return }
            commentVC.presenter = presenter.commentPresenter
            commentVC.presenter.view = commentVC
        default:
            return
        }
    }

    // MARK: - IBActions

    @IBAction private func filterMineAssignmentsButtonClicked(_ sender: UIButton) {
        presenter.viewDidSelectMineRelativity()
    }

    @IBAction private func calendarButtonClicked(_ sender: UIButton) {
        animateCalendar()
    }

    // MARK: - Private

    private func prepareView() {
        filterAllAssignmentsButton.setTitle(presenter.allRelativityTitle, for: .normal)
        filterMineAssignmentsButton.setTitle(presenter.mineRelativityTitle, for: .normal)

        collegialBodyLabel.text = presenter.collegialBodyDefaultTitle

        assignmentsCollectionView.register(UINib(nibName: cellIdentifier, bundle: nil), forCellWithReuseIdentifier: cellIdentifier)
    }
}

// MARK: - UICollectionViewFlowLayout

extension AssignmentListViewController: UICollectionViewDelegateFlowLayout {

    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, insetForSectionAt section: Int) -> UIEdgeInsets {
        collectionView.scrollIndicatorInsets = UIEdgeInsets(top: visualEffectView.frame.height - assignmentCellGap, left: 0, bottom: 0, right: 0)
        return UIEdgeInsets(top: visualEffectView.frame.height, left: assignmentCellGap, bottom: assignmentCellGap, right: assignmentCellGap)
    }

    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, minimumLineSpacingForSectionAt section: Int) -> CGFloat {
        return assignmentCellGap
    }

    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, minimumInteritemSpacingForSectionAt section: Int) -> CGFloat {
        return assignmentCellGap
    }

    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> CGSize {
        return CGSize(width: collectionView.bounds.width - 2.0 * assignmentCellGap, height: 200.0)
    }
}

// MARK: - UICollectionViewDataSource

extension AssignmentListViewController: UICollectionViewDataSource {

    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return presenter.numberOfAssignments
    }

    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        guard let cell = collectionView.dequeueReusableCell(withReuseIdentifier: cellIdentifier, for: indexPath) as? AssignmentCollectionViewCell else {
            fatalError("CollectionViewCell не имеет тип \(cellIdentifier)")
        }

        guard let cellPresenter = presenter.assignmentCellPresenter(atIndex: indexPath.row) else {
            fatalError("Не удалось получить presenter для \(cellIdentifier)")
        }

        cell.presenter = cellPresenter
        cell.presenter.view = cell

        return cell
    }
}

// MARK: - UICollectionViewDelegate

extension AssignmentListViewController: UICollectionViewDelegate {

    func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {
        presenter.viewDidSelectAssignment(atIndex: indexPath.row)
    }
}

// MARK: - AssignmentListViewProtocol

extension AssignmentListViewController: AssignmentListViewProtocol {

    func showAssignment() {
        performSegue(withIdentifier: meetingSegueIdentifier, sender: self)
    }

    func showLoadingIndicator(_ show: Bool) {
        loadingIndicatorContrainerView.isHidden = !show
        show ? loadingActivityIndicatorView.startAnimating() : loadingActivityIndicatorView.stopAnimating()
    }

    func showError(title: String, message: String) {
        let errorAlert = UIAlertController(title: title, message: message, preferredStyle: .alert)
        errorAlert.addAction(UIAlertAction(title: "Ок".localized, style: .cancel, handler: nil))
        present(errorAlert, animated: true)
    }

    func reloadAssignments() {
        assignmentsCollectionView.reloadData()
    }
}
```

**Важно:** За логику layout'а и как именно отображать подготовленные данные отвечает `view`.

```swift
class MaterialActionListTableViewCell: UITableViewCell {

    // MARK: - IBOutlets

    @IBOutlet private weak var actionTitleLabel: UILabel!

    // MARK: - Properties

    var presenter: MaterialActionListCellPresenterProtocol! {
        didSet {
            bindPresenter()
        }
    }

    // MARK: - Private

    private func bindPresenter() {
        actionTitleLabel.text = presenter.title
    }
}

// MARK: - MaterialActionListCellViewProtocol

extension MaterialActionListTableViewCell: MaterialActionListCellViewProtocol {}
```

```swift
class MaterialActionListCellPresenter: MaterialActionListCellPresenterProtocol {

    // MARK: - Properties

    weak var view: MaterialActionListCellViewProtocol!

    var title: String

    // MARK: - Initializers

    init(with title: String) {
        self.title = title
    }
}
```

**Важно:** При возвращении в `presenter` на главный поток желательно использовать `CFRunLoopPerformBlock(CFRunLoopGetMain(), CFRunLoopMode.defaultMode.rawValue) {}`, где это возможно.
