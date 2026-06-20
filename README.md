# Решение задания CTF ИТМО

## Цель
Получить флаг, войдя в систему как администратор на сайте `http://89.169.160.128:8080`.

## Инструменты
- Браузер с инструментами разработчика (F12)
- Онлайн-декодер JWT: [jwt.io](https://jwt.io)
- Ресурс с полезными нагрузками: [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)

---

## Этапы решения

### 1. SQL-инъекция для обхода аутентификации
На странице входа я использовал классическую полезную нагрузку в поле **Username**:

```text
admin' OR '1'='1'--
```

Поле Password было оставлено пустым (или можно было ввести любое значение, например 123).
Это позволило обойти проверку пароля, так как условие '1'='1' всегда истинно.
После нажатия на кнопку входа я успешно авторизовался как обычный пользователь student и попал на страницу профиля.

<img width="960" height="482" alt="acc" src="https://github.com/user-attachments/assets/77d2f923-081d-49e9-addf-d8ac1807f5fa" />


### 2. Получение оригинального JWT-токена
В открывшейся странице профиля я открыл инструменты разработчика (клавиша F12) и перешёл во вкладку Application (в Firefox — Хранилище).
В разделе Local Storage я нашёл ключ с именем token и скопировал его значение.
Оригинальный токен выглядел так (он уже содержал алгоритм none, так как это была промежуточная версия):

```text
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiIxIiwidXNlcm5hbWUiOiJzdHVkZW50IiwibmFtZSI6IklUTU8gU3R1ZGVudCIsInJvbGUiOiJ1c2VyIiwiaWF0IjoxNzgxOTY2NTQ1fQ.
```

При декодировании полезной нагрузки (Payload) на jwt.io я увидел, что моя роль — "user":

```
json
{
  "sub": "1",
  "username": "student",
  "name": "ITMO Student",
  "role": "user",
  "iat": 1781966545
}
```
### 3. Подделка JWT (атака на алгоритм none)
Чтобы стать администратором, я изменил токен следующим образом:

Заголовок (Header):
Я оставил алгоритм none, так как это позволяет серверу не проверять подпись:

```
json
{
  "alg": "none",
  "typ": "JWT"
}
```
Полезная нагрузка (Payload):
Я изменил имя пользователя на admin и роль на admin:
```
json
{
  "sub": "1",
  "username": "admin",
  "name": "Admin",
  "role": "admin",
  "iat": 1781966545
}
```
<img width="960" height="482" alt="jwt" src="https://github.com/user-attachments/assets/a7c9746b-4ce7-4b6b-aa50-480f946bea92" />

Подпись (Signature):
Так как алгоритм установлен в none, подпись не требуется. Я удалил последнюю часть исходного токена (после третьей точки), оставив точку в конце, чтобы сделать подпись пустой.

В результате я получил подделанный токен:

```text
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiIxIiwidXNlcm5hbWUiOiJhZG1pbiIsIm5hbWUiOiJBZG1pbiIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTc4MTk2NjU0NX0.
```

<img width="960" height="484" alt="acc admin" src="https://github.com/user-attachments/assets/cdec2d16-8365-40bd-9729-50cdf22bfb46" />

### 4. Внедрение поддельного токена и получение флага
Я вернулся на страницу профиля, открыл инструменты разработчика (F12), снова перешёл в Local Storage и заменил значение ключа token на новый поддельный токен.
Затем я обновил страницу (F5) и нажал на кнопку Admin в верхней части экрана.
Система распознала меня как администратора и отобразила флаг.

Полученный флаг
```text
ITMO{sql_injection_to_unsigned_jwt_admin}
```
<img width="960" height="482" alt="flag" src="https://github.com/user-attachments/assets/c8a64ce0-0c35-4f7b-b055-2ea11382a177" />
