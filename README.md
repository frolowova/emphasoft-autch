# emphasoft

## Project info
```
В данном ТЗ реализовано 
1. Доступ к серверу с использованием fetch.
2. Обработка ответов с сервера.
3. Валидация имени и пароля на клиентской стороне по правилам описанным в API (pattern).
4. Выпадающий информационный блок о состоянии валидации и успешности авторизации.
5. Компонент списка пользователей, к которому можно получить доступ, только после успешной авторизации на сервере.
6. При успешной авторизации загружается список и осуществляется автоматическая прокрутка (скроллинг) до его начала.
7. Сортировка списка по ID.
8. Фильтрация списка по имени пользователя 
(без учёта регистра и с неточным совадением (например, поисковый запрос "ра" выдаст не только "Рафаель", но и варианты "Ира" "ира" "Юра"...)). 
При открытие блока фильтрации фокус сразу устанавливается на input.
Закрытие блока фильтрации очищает фильтр и выводит весь список.
9. Уменьшение шрифта заголовков списка при малой ширине экрана.


Не использовал vuetify, хотя мог. 
Не заострял внимание на дизайне, так как решил время уделить логике. Но в большей степени 
ориентировался на мобильное отображение, поэтому кнопка внизу, инпуты в середине. 
Список пользователей отображается в том же окне, так как указаний по этому поводу не было, поэтому решил поработать с прокруткой страницы.
```

### adress
```
Страница проекта: (https://frolowova.github.io/emphasoft-autch-page/)
```

