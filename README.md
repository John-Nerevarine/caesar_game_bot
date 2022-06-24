# Текстовая игра в чат-боте. 
## Описание
Игра на внимательность в формате чат-бота Telegram. Учебный проект.
Бот доступен по ссылке https://t.me/caesargame_bot (если я не забыл оплатить хостинг)

### Название 
Гай Юлий Цезарь (12 июля 100 года до н. э. — 15 марта 44 года до н. э.) — древнеримский государственный и политический деятель, полководец, писатель. В массовой культуре бытует мнение, что Юлий Цезарь мог делать несколько дел одновременно. Не берусь утверждать насколько это соответствует действительности, но в именно из-за этой особенности и была названа игра.

### Суть игры
В начале игры бот загадывает игроку позиции фигур на шахматной доске. Затем говорит, куда передвигается каждая фигура. Фигура может сдвинуться вверх, вниз, влево или вправо на одно поле. За один раз бот называет только один ход каждой фигуры. После совершения всех ходов, игрок должен назвать конечную позицию всех фигур. Количество ходов увеличивается с каждым раундом.

### Сложность
В игре присутствует четыре вида сложности:
- МS-DOS (легко)
- Хамелеон (средне)
- Третья рука (сложно)
- Цезарь (очень сложно)
Сложность определяет количество фигур, передвижения которых должен отследить игрок. Первые три сложности задают 1, 2 и 3 фигуры соответственно. Четвёртая сложность задаёт четыре фигуры и увеличивает количество фигур с каждым раундом.

### Рекорды
За каждый пройденный раунд игроку начисляются очки в количестве, зависящим от номера раунда и количества фигур в игре. Если игрок ошибается, очки сбрасываются. Игра сохраняет индивидуальный рекорд каждого игрока, а также общий рекорд всех игроков.

### Можно улучшить
Можно было сделать фигуры различными, как в шахматах, и совершать ходы в зависимости от вида фигуры.

## Реализация
Для взаимодействия с Telegram Bot API в боте использована библиотека aiogram v2.19. Для хранения данных используется SQLite и библиотека sqlite3.
Бот разбит на 10 модулей. Каждый модуль отвечает за определённый функционал бота.
Список модулей:
- **bot.py**
- **createBot.py**
- **config.py**
- **dataBase.py**
- **keyboards.py**
- **figures.py**
- **systemHandlers.py**
- **mainMenuHandlers.py**
- **gameHandlers.py**
- **caesarHandlers.py**



## Модуль **bot.py**
Основной модуль бота. Служит центральным связующим звеном между остальными модулями, а также запускает бота.

Первой строкой указыватся путь к Python в
виртуальном окружении.
Затем импортируются необходимые модули бота, **executor** из **aiogram.utils**, диспетчер из модуля **createBot**.

В модуле есть только одна функция **on_startup()**, которая выполняется при запуске. Она запускает фукнцию **sqlStart()** модуля **dataBase**, для подключения к базе данных.

После объявления функций регистрируются обработчики из импортированных модулей.

Ну и в последнюю очередь запускается поллинг, который и вызывает функцию **on_startup()**.



## Модуль **createBot.py**
Модуль создания бота, определения "общих" переменных и состояний.

Производится импорт основных элементов бота из библиотеки **aiogram** и токена бота из модуля **config**.
Затем объявляются **MemoryStorage**, непосредственно сам объект Бота, диспетчер, глобальные текстовые переменные и подклассы состояния.
Все события в игре привязаны к состояниям. Определённые действия могут быть совершены только в опредёленном состоянии. И соответственно, действия изменяют текущее состояние.

Все объекты из этого модуля будут импортироваться в другие модули для работы.



## Модуль **config.py**
В этом модуле хранится токен бота и id изображения, которая используется в дальнейшем. Сделано отдельным модулем для простоты загрузки на сервер в случае смены бота или изображения.



## Модуль **dataBase.py**
Модуль для работы с базами данных и некоторыми дополнительными функциями.

| Функция | Описание |
|----:|:----|
|sqlStart()|Функция создаёт новую или соединяется с существующей базой данных. База данных состоит из двух таблиц: таблица общего рекорда (*id, record, userName, string_number*) и таблица с персональными данными для каждого пользователя (*id, record, score, round, step, figures, startPos, directions, positions*).|
|stringToIntList(string, i=0)|Функция десериализации строки. Для хранения данных о ходах и позициях были использованы двумерные списки. Хранить в базе данных список как объект возможности нет. Логично было бы использовать сериализацию списка в JSON, но так как проект учебный, было решено сделать свою сериализацию/десериализацию. Перед занесением в базу данных списки переводятся в строку. С этим справляется стандартная функция str(). Но, чтобы из такой строки получить обратно список, стандартных средств python недостаточно. Функция list() возвращает из строки список символов, а мне нужно было получить первоначальный двумерный список. Для этого была написана эта функция. Так как функционал программы не предусматривает списков большего уровня вложения, чем двумерные, можно было написать более простую функцию. Но опять же, проект учебный, поэтому решено было написать универсальную функцию, которая могла бы вернуть список любого уровня вложения. Также я подумал, что для этого подходит использование рекурсивной функции. Была написана функция, которая в качестве аргументов принимает строку и номер символа, с которого должен начинаться анализ. Функция проходится по символам строки, при встрече символа '[', запускает саму себя, далее компонует строки по символу ',', возвращает список и номер символа на котором остановила анализ. Эта функция работает не совсем правильно, она возвращает изначальный список, но заключённый в новый список. Для решения этой проблемы, а так же для избавления от возвращения номера символа была написана функция-оболочка, которая возвращала требуемый список. Это своего рода "костыль", но работающий.|
|getStringList(string)|Функция-оболочка функции **stringToIntList()**.|
|answerProcessing(answer)|Функция-обработчик ответа пользователя. Убирает ненужные символы и пробелы и приводит строку в вид пригодный для сравнения со строкой финальных позиций фигур.|
|addUser(userId)|Создаёт в базе данных новую строку для пользователя с ID в телеграмме **userId**.|
|isNew(searchingId)|Ищёт в базе данных пользователя с ID **searchingId**. Возвращает **True** или **False** в зависимости от результата.|
|getBestOfAll()|Возвращает общий рекорд среди всех пользователей.|
|updateBestOfAll(userId, userRecord, name)|Обновляет общий рекорд.|
|getBestPersonal(userId)|Возвращает личный рекорд по ID пользователя.|
|updateOnePersonal(userId, column, value)|Изменяет значение поля **column** на **value** для пользователя с ID **userId**.|
|updateManyPersonal(userId, columnList, valuesList)|Изменяет значение полей из списка **columnList** на значения из списка**valuesList** для пользователя с ID **userId**.|
|getOnePersonal(userId, column)|Возвращает значение поля **column** для пользователя с ID **userId**.|
|getManyPersonal(userId)|Возвращает список значений всех полей для пользователя с ID **userId**.|



## Модуль **keyboards.py**
Модуль для объявления кнопок, возвращаемых значений этих кнопок, и клавиатур. Из этого модуля они потом импортируются в модули **gameHandlers**, **mainMenuHandlers**, **systemHandlers** и **caesarHandlers**.



## Модуль **figures.py**
Модуль основной игровой логики, определяет стартовые позиции, ходы и финальные позиции фигур. 

Импортируются только методы **randint** и **choices** модуля **random**.

Единственная функция модуля **figureSteps(figure=1, rounds=1)** в качестве аргументов принимает количество фигур иколичество раундов. Функция случайным образом выбирает позицию фигур на поле, записывает их в два списка – один останется без изменений, второй будет изменяться в соответствии с ходами для получения финальных позиций. Затем функция выбирает для каждой фигуры возможные ходы, совершает их, и записывает в третий список. Возвращает список начальных позиций, список ходов и список финальных позиций. 



## Модуль **systemHandlers.py**
Обработка системных событий. Так как функционирование бота завязано на машине состояний, особое внимание при описании функций в данном и последующих модулях будет уделяться состояниям, из которых функция вызывается, и которые задаёт.

| Функция | Описание |
|----:|:----|
|command_start(message: types.Message, state: FSMContext)|Функция выводит разное приветственное сообщение в зависимости от того, в первый ли раз пользователь зашёл в бота, и выводит главное меню. Вызывается из любого состояния по команде **/start**. Задаёт состояние **Menu.mainMenu**.|
|callback_records_back(callback_query: types.CallbackQuery, state: FSMContext)|Функция возврата в главное меню из меню рекордов, меню просмотра правил и меню выбора сложности по кнопке **Назад**. Вызывается из состояний **Menu.records**, **Menu.rules** и **Game.difficulty**. Задаёт состояние **Menu.mainMenu**.|
|callback_restart(callback_query: types.CallbackQuery, state: FSMContext)|Функция возврата в главное меню по кнопке **Начать снова** после проигранного раунда. Вызывается из любого состояния, задаёт состояние **Menu.mainMenu**.|
|command_help_difficulty(message: types.Message)|Функция вызова подсказки по уровням сложности по команде **/help** в меню выбора сложности. Вызывается из состояния **Game.difficulty**, задаёт состояние **System.helpDiff**.|
|callback_back_difficulty(callback_query: types.CallbackQuery, state: FSMContext)|Функция возврата к меню выбора сложности из подсказки по кнопке **Продолжить**. Вызывается из состояния **System.helpDiff**, задаёт состояние **Game.difficulty**.|
|command_help_answer_game(message: types.Message)|Функция вызова подсказки о формате ответа по команде **/help** при выборе ответа. Вызывается из состояния **Game.answer**, задаёт состояние **System.helpAnswerGame**.|
|command_help_answer_caesar(message: types.Message)|Функция вызова подсказки о формате ответа по команде **/help** при выборе ответа на сложности **Цезарь**. Вызывается из состояния **Caesar.answer**, задаёт состояние **System.helpAnswerCaesar**.|
|callback_backHelp_game(callback_query: types.CallbackQuery, state: FSMContext)|Возврат к выбору ответа из подсказки по кнопке **Продолжить**. Вызывается из состояния **System.helpAnswerGame**, задаёт состояние **Game.answer**.|
|callback_backHelp_caesar(callback_query: types.CallbackQuery, state: FSMContext)|Возврат к выбору ответа из подсказки по кнопке **Продолжить** на сложности **Цезарь**. Вызывается из состояния **System.helpAnswerCaesar**, задаёт состояние **Caesar.answer**.|
|registerHandlersSystem(dp : Dispatcher)|Регистрация обработчиков. Вызывается из модуля **bot.py**.|



## Модуль **mainMenuHandlers.py**
Обработка событий главного меню.

| Функция | Описание |
|----:|:----|
|callback_records(callback_query: types.CallbackQuery, state: FSMContext)|Функция выводит лиичный и общий рекоды на экран по кнопке **Рекорды**. Вызывается из состояния **Menu.mainMenu**, задаёт состояние **Menu.records**.|
|callback_rules(callback_query: types.CallbackQuery, state: FSMContext)|Вывыодит на экран описание и правила игры, а также показывает изображение шахматного поля с подписями координат. Вызывается из состояния **Menu.mainMenu**, задаёт состояние **Menu.rules**.|
|scan_message(msg: types.Message)|Служебная функция. Если отправить боту любое изображение, в терминал выведется его ID, а для пользователя ничего не произойдёт. ID изображения нужен, чтобы поменять изображение шахматной доски в описании правил. Вызывается из любого состояния, оставляет текущее состояние без изменений.|
|registerHandlersMainMenu(dp : Dispatcher)|Регистрация обработчиков. Вызывается из модуля **bot.py**.|



## Модуль **gameHandlers.py**
Данный модуль содержит обработчики событий игрового процесса на трёх из четырёх уровнях сложности игры. Сначала у пользователя спрашивают, на какой сложности он собирается играть. В зависимости от его выбора, ему даётся от одной до четырёх фигур для игры. Затем запускается функция получения ходов. После этого начинается процесс игры. Каждый ход в базе данных обновляется счётчик ходов. Как только количество ходов сравняется с номером раунда, игра заканчивается. Игрок вводит ответ. После чего ему предлагается либо выйти в главное меню, либо продолжить играть.

| Функция | Описание |
|----:|:----|
|callback_start(callback_query: types.CallbackQuery, state: FSMContext)|Функция вывода меню выбора сложности. Может быть вызвана в трёх случаях. Первый: по кнопке **Начать** в главном меню - из состояния **Menu.mainMenu**. Второй: по кнопке **Начать снова** после завершения игры на одной из перрвых трёх сложностей - из состояния **Game.finish**. Третий: по кнопке **Начать снова** после завершения игры на сложности **Цезарь** - из состояния **Caesar.finish**. Во всех случаях задаёт состояние **Game.difficulty**|
|callback_game(callback_query: types.CallbackQuery, state: FSMContext)|Функция начала игры. В зависимости от сложности задаёт количество фигур, вычисляются стартовые и финальные позиции, ходы фигур. Пользователю выводятся стартовые позиции и предлагается начать ход. Вызывается из состояния **Game.difficulty**, задаёт состояние **Game.step**.|
|callback_step_again(callback_query: types.CallbackQuery, state: FSMContext)|Функция продолжения игры после успешного завершения раунда. Вычисляются стартовые и финальные позиции, ходы фигур. Функция продолжения игры отличается от функции первого запуска тем, что не обнуляет счётчик очков. Пользователю выводятся стартовые позиции и предлагается начать ход. Вызывается из состояния **Game.finish**, задаёт состояние **Game.step**.|
|callback_step(callback_query: types.CallbackQuery, state: FSMContext)|Функция хода фигуры. Выводит сообщение о направлении хода фигуры. Вызывается из состояния **Game.step**. Если ходы закончились, сообщает об этом пользователю и предлагает ввести финальные позиции фигур. В таком случае задаёт состояние **Game.answer**.|
|command_answer(message: types.Message, state: FSMContext)|Функция обработки ответа от пользователя. Проверяет правильность ответа, наличие личного и общего рекордов, выводит сообщение соответствующее победе или поражению. Вызывается из состояния **Game.answer**, задаёт состояние **Game.finish**.|
|registerHandlersGame(dp : Dispatcher)|Регистрация обработчиков. Вызывается из модуля **bot.py**.|



## Модуль **caesarHandlers.py**
Данный модуль содержит обработчики событий игрового процесса на червёртой сложности игры. Игровой процесс отличается тем, что в каждом новом раунде добавляется одна фигура. В остальном игровой процесс идентичен.

| Функция | Описание |
|----:|:----|
|callback_caesar_game(callback_query: types.CallbackQuery, state: FSMContext)|Функция начала игры. В задаёт начальное количество фигур равное четырём, вычисляются стартовые и финальные позиции, ходы фигур. Пользователю выводятся стартовые позиции и предлагается начать ход. Вызывается из состояния **Game.difficulty**, задаёт состояние **Caesar.step**.|
|callback_caesar_step_again(callback_query: types.CallbackQuery, state: FSMContext)|Функция продолжения игры после успешного завершения раунда. Вычисляются стартовые и финальные позиции, ходы фигур. Функция продолжения игры отличается от функции первого запуска тем, что не обнуляет счётчик очков и добавляет одну фигуру. Пользователю выводятся стартовые позиции и предлагается начать ход. Вызывается из состояния **Caesar.finish**, задаёт состояние **Caesar.step**.|
|callback_caesar_step(callback_query: types.CallbackQuery, state: FSMContext)|Функция хода фигур. Выводит сообщение о направлении хода фигур. Вызывается из состояния **Caesar.step**. Если ходы закончились, сообщает об этом пользователю и предлагает ввести финальные позиции фигур. В таком случае задаёт состояние **Caesar.answer**.|
|command_caesar_answer(message: types.Message, state: FSMContext)|Функция обработки ответа от пользователя. Проверяет правильность ответа, наличие личного и общего рекордов, выводит сообщение соответствующее победе или поражению. Вызывается из состояния **Caesar.answer**, задаёт состояние **Caesar.finish**.|
|registerHandlersCaesar(dp : Dispatcher)|Регистрация обработчиков. Вызывается из модуля **bot.py**.|

# Спасибо за внимание!