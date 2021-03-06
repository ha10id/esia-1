# esia
Модуль идентификации и авторизации пользователей через ЕСИА для Node.js

Перед использованием ознакомтесь с [документацией](http://minsvyaz.ru/ru/documents/4240/).

## использование

1. Установите
```js
npm i -S esia
```

2. Создайте экземпляр подключения к ЕСИА

```js
  const fs = require('fs');
  const esia = require('esia');

  // читается сертификат используемый в ЕСИА
  const cert = fs.readFileSync('./certificate.cer', 'utf8');
  // и ключ
  const key =  fs.readFileSync('./privkey.key', 'utf8');

  const esiaConnection = esia({
    clientId: 111111,
    redirectUri: 'https://my-site.com/esiacode/',
    scope: 'openid id_doc',
    certificate: cert,
    key: key
  });
```

3. Направьте пользвателя в ЕСИА для получения подтверждения

```js
// сгенерируйте url для пользователя
const authUrl = esiaConnection.getAuth().url
```

4. После того как пользователь авторизуется и даст разрешение на доступ к своим данным, он будет перенаправлен по адресу, указанному в redirectUri. Вы получите параметр code, который нужно будет использовать для запроса данных
```js
esiaConnection
  .getAccess(code)
  .then(result => {
    //  обработка результата
  })
  .catch(err => {
    // обработка ошибки
  });
```

Метод getAccess возвращает Promise, в который приходит объект результата. Этот объект содержит два поля:
- marker - маркер доступа. Объект, содержащий поля:
    - response - объект, ответ от ЕСИА при запросе маркера
    - decodedAccessToken - объект, jwt декодированное значение поля access_token, содержащееся в response.
- data - массив записей данных о пользователе.

В метод getAccess вторым параметром можно передать массив путей для получения записей данных о пользователе. Они будут содержаться в ответе, в поле data, описанном выше. Если параметр не передавать, по умолчанию будет использоваться ['/']. Если передать null, то данные о пользователе запрашиваться не будут.

```js
esiaConnection
  .getAccess(code, ['/', '/docs']) // будет запрошена основная информация и список документов
  .then(result => {
    console.log(result.data) // массив из двух записей
  })
  .catch(err => {
    // обработка ошибки
  });

  esiaConnection
    .getAccess(code, null) // не будет запрошена информация о пользователе
    .then(result => {
      console.log(result.data) // пустой массив
    })
    .catch(err => {
      // обработка ошибки
    });
```

Подробнее о получении информации о пользователе читайте в официальной документации ЕСИА.



