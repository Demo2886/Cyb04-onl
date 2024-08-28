# Git

Git — это распределенная система управления версиями, которая отслеживает изменения в коде с течением времени. 

Это позволяет нескольким разработчикам работать над одним и тем же проектом, не наступая друг другу на пятки.

Основные команды Git:

1. Команда git init
 Инициализирует новый репозиторий Git. Это все равно, что сказать: «Эй, Гит, начни следить за этим проектом!»

2. git clone [URL]
 Создает копию удаленного репозитория
  на локальном компьютере. Это то, как вы загружаете проект, чтобы начать вносить свой вклад.

3. git add [файл]
 Подготавливает изменения для коммита. Думайте об этом как о том, чтобы положить мелочь в корзину перед оформлением заказа.

4. git commit -m "[сообщение]"
 Фиксирует проиндексированные изменения с помощью описательного сообщения. Это похоже на создание моментального снимка вашего проекта в определенный момент времени.

5. Команда git push
 Отправляет зафиксированные изменения в удаленный репозиторий. Поделитесь своей работой со всем миром (или хотя бы со своей командой)!

6. Команда git pull
 Извлекает изменения из удаленного репозитория и объединяет их с текущей ветвью. Поддерживайте свою локальную копию в актуальном состоянии.

7. Ветка Git
 Список всех местных отделений. Полезно для просмотра веток функций, которые у вас есть.

8. git checkout -b [имя-ветки]
 Создает новую ветку и переключается на нее. Идеально подходит для работы над новыми функциями, не затрагивая основной код.

9. git merge [ветка]
 Объединяет указанную ветвь с текущей ветвью. Таким образом, вы интегрируете свою новую функцию обратно в основной код.

10. Статус git
 Отображает состояние изменений как неотслеживаемых, измененных или подготовленных. Проверка работоспособности вашего проекта!

11. Журнал Git
 Отображает журнал всех коммитов. Как машина времени для вашего кода.

12. Команда Git Stash
 Временно откладывает на полку изменения, которые вы внесли в рабочую копию, чтобы можно было работать над чем-то другим, а затем возвращается и повторно применяет их позже.

Советы от профессионалов:
- Используйте содержательные сообщения о коммитах. В будущем вы (и ваши товарищи по команде) будете вам благодарны.
- Делайте коммиты часто. Небольшими, частыми коммитами легче управлять, чем большими, нечастыми.
- Используйте ветки для новых функций или экспериментов. Содержите главную ветвь в чистоте и устойчивом состоянии.
- Всегда тяните, прежде чем нажимать, чтобы избежать конфликтов.


<div id="header" align="center">
  <img src="https://github.com/Demo2886/Cyb04-onl/blob/master/%F0%9F%92%80InfoMysor%F0%9F%92%80/img/gitWorkflow.gif" width="100"/>
</div>

# GET, POST, PUT, DELETE… 

A list of common HTTP “verbs” in one diagram. The method to download the high-resolution PDF is available at the end.

1. HTTP GET
This retrieves a resource from the server. It is idempotent. Multiple identical requests return the same result.

2. HTTP PUT
This updates or Creates a resource. It is idempotent. Multiple identical requests will update the same resource.

3. HTTP POST
This is used to create new resources. It is not idempotent, making two identical POST will duplicate the resource creation.

4. HTTP DELETE
This is used to delete a resource. It is idempotent. Multiple identical requests will delete the same resource.

5. HTTP PATCH
The PATCH method applies partial modifications to a resource.

6. HTTP HEAD
The HEAD method asks for a response identical to a GET request but without the response body.

7. HTTP CONNECT
The CONNECT method establishes a tunnel to the server identified by the target resource.

8. HTTP OPTIONS
This describes the communication options for the target resource.

9. HTTP TRACE
This performs a message loop-back test along the path to the target resource.

10. HTTP LINK
The LINK method establishes a link between resources.

11. HTTP UNLINK
The UNLINK method removes a link between resources.

12. HTTP COPY
The COPY method copies a resource.

13. HTTP LOCK
The LOCK method acquires a lock on a resource.

14. HTTP UNLOCK
The UNLOCK method releases a lock on a resource.

<div id="header" align="center">
  <img src="https://github.com/Demo2886/Cyb04-onl/blob/master/%F0%9F%92%80InfoMysor%F0%9F%92%80/img/HTTPRequest.gif" width="100"/>
</div>
