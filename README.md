<a href="https://pypi.org/project/pyrepka/" target="_blank" rel="noreferrer"><img src="https://img.shields.io/pypi/v/PyRepka"/></a>
<a href="https://pypi.org/project/pyrepka/" target="_blank" rel="noreferrer"><img src="https://img.shields.io/pypi/pyversions/pyrepka"/></a>
<a href="https://p2pbot.pro/repka" target="_blank" rel="noreferrer"><img src="https://img.shields.io/badge/repka-blue?link=https%3A%2F%2Fp2pbot.pro%2Frepka"/></a>


# PyRepka

PyRepka - библиотека для работы с API реферальной программой "Репка"

### РЕПКА это:

- 🤖 Единая реферальная программа, для большой системы ботов.
- 💵 Система позволяющая получать бонусы за приглашения и тратить их в любом боте с поддержкой РЕПКИ.
- 👑 Приглашай пока тебя не пригласили. Первым быть выгодно!


# Содержание

* [Запуск](#Запуск)
* [Методы](#Методы)
    * [CheckUserRef](#CheckUserRef)
    * [AddUserToRef](#AddUserToRef)
    * [GetReferralUsers](#GetReferralUsers)
    * [GetUserBalance](#GetUserBalance)
    * [Purchase](#Purchase)
    * [Replenishment](#Replenishment)
* [Примеры](#Примеры)


# Запуск

### Установка

Установка через pip:

```python
pip install pyrepka
```

### Инициализация:

```python
from pyrepka import PyRepka

py_repka = PyRepka(bot_access_token, bot_username, ip, port)
```

| Параметр | Тип | Описание |
|-|-|-|
| bot_access_token | str | Токен для подключения к Репке |
| bot_username | str | Юзернейм бота |
| ip | str | IP-адрес сервера, на котором работает API |
| port | str | Порт сервера, на котором работает API |

*Все эти данные выдает разработчик API*


# Методы


### CheckUserRef

Проверить, является ли пользователь участником реферальной программы

GET-запрос

| Параметр | Тип | Описание |
|-|-|-|
| user_id | int | ID пользователя, которого необходимо проверить |

#### Пример использования:

```python
py_repka.check_user_ref(user_id)
```


---


### AddUserToRef

Добавить пользователя в реф. программу

POST-запрос

| Параметр | Тип | Описание |
|-|-|-|
| user_id | int | ID пользователя, который добавляется в реф. программу |
| first_name | str | Имя пользователя |
| last_name | str | Фамилия пользователя |
| username | str | Юзернейм пользователя |
| inviter_id | int / None | ID пользователя, который пригласил, если он есть |

#### Пример использования:

```python
py_repka.add_user_to_ref(
    user_id,
    first_name,
    last_name,
    username,
    inviter_id
)
```


---


### GetReferralUsers

Получить количество пользователей на каждом уровне

GET-запрос

| Параметр | Тип | Описание |
|-|-|-|
| user_id | int | ID пользователя, у которого нужно узнать количество пользователей на каждом уровне |

#### Пример использования:

```python
py_repka.get_referral_users(user_id)
```


---


### GetUserBalance

Получить баланс пользователя

GET-запрос

| Параметр | Тип | Описание |
|-|-|-|
| user_id | int | ID пользователя, баланс которого нужно узнать |

#### Пример использования:

```python
py_repka.get_user_balance(user_id)
```


---


### Purchase

Покупка (за баллы)

POST-запрос

| Параметр | Тип | Описание |
|-|-|-|
| user_id | int | ID пользователя, который совершает покупку |
| points | int | Количество баллов, которые будут списаны с баланса пользователя |
| reason | str | Название операции |

#### Пример использования:

```python
py_repka.purchase(
    user_id,
    points,
    reason
)
```


---


### Replenishment

Зафиксировать факт пополнения баланса пользователя с помощью эквайринга

POST-запрос

| Параметр | Тип | Описание |
|-|-|-|
| user_id | int | ID пользователя, чей баланс пополняется |
| points | int | Количество баллов, которое пополняется на баланс пользователя |
| reason | str | Название операции |

#### Пример использования:

```python
py_repka.replenishment(
    user_id,
    points,
    reason
)
```


# Примеры

#### Пример запуска бота на aiogram 3

```python
from pyrepka import PyRepka

# Инициализация объекта реферальной программы
py_repka = PyRepka(bot_access_token, bot_username, ip, port)

@router.message(F.text.contains("/start"))
async def start_handler(message: Message):

    # ID пригласителя
    try:
        invited_by = message.text.split()[1]
    except: 
        invited_by = None

    # Добавление в реферальную программу
    py_repka.add_user_to_ref(
        message.from_user.id,
        message.from_user.first_name,
        message.from_user.last_name,
        message.from_user.username,
        invited_by
    )
```


---


#### Пример оплаты чего-либо на aiogram 3

```python
from pyrepka import PyRepka

# Инициализация объекта реферальной программы
py_repka = PyRepka(bot_access_token, bot_username, ip, port)

@router.callback_query(F.data.contains("purchase"))
async def purchase_handler(callback: CallbackQuery):

    # Списывание средств баланса
    response = py_repka.purchase(
        callback.from_user.id,
        config.PRICE,
        messages.TEXT
    )

    # Проверка ответа на списывание средств с баланса
    if response.status_code == 200:
        # Вывод сообщения об успешном списании средств с баланса
        await callback.answer(
            messages.SUCCESS_PURCHASE,
            show_alert=True
        )

    # В случае ошибки
    else:
        # Вывод сообщения о нехватке средств на балансе
        await callback.answer(
            messages.NO_MONEY,
            show_alert=True
        )
```