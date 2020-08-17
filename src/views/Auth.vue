<template>
  <div class="wrapp">
    <div class="auth">
      <div class="auth-block" ref="authblock">
        <div class="auth-block__infoBlock" v-show="showInfo">
          <p v-for="(mess, index) in messInfo" :key="index">{{mess}}</p>
        </div>
        <input
          v-model="userName"
          class="auth-block__inp"
          type="text"
          placeholder="Имя пользователя"
        />
        <input v-model="userPass" class="auth-block__inp" type="password" placeholder="Пароль" />
        <button class="auth-block__btn" @click="authClick">Войти</button>
      </div>
    </div>
    <div class="users-list">
      <UsersList :token="token" v-if="filterShow" ref="toplist"></UsersList>
    </div>
  </div>
</template>

<script>
import UsersList from "@/components/UsersList.vue";

export default {
  name: "Auth",
  components: {
    UsersList,
  },
  data() {
    return {
      userName: "",
      userPass: "",
      messInfo: [],
      showInfo: false,
      filterShow: false,
      token: "",
    };
  },
  methods: {
    authClick() {
      // Очищаем массив с ошибками
      this.messInfo.length = 0;
      // Отключаем отображение блока сообщений об ошибках валидации
      this.showInfo = false;
      /*
      Задача запустить проверку (валидацию) если успех запустить отправку запроса
      иначе выдать сообщение об ошибке 
      */
      const validName = this.validName(this.userName); // Получаем или "" или текст ошибки
      const validPass = this.validPass(this.userPass); // Получаем или "" или текст ошибки
      // Если сообщений об ошибке не будет, то валидация пройдена => отправляем запрос на сервер
      if (!validName && !validPass) {
        this.sendRequest(this.userName, this.userPass);
      }
      // Иначе записываем данные об ошибке
      else {
        if (validName) this.messInfo.push(validName);
        if (validPass) this.messInfo.push(validPass);
        // Выводим данные об ошибке в соответствующий блок
        this.showError(this.messInfo);
      }
    },

    validName(name) {
      // Делаем проверку (валидацию) на string и на наличие значений
      if (typeof name !== "string") return "Введённое значение имени не string";
      if (name.length == 0) return "Введите имя";
      if (name.length > 150)
        return "Введённое имя больше 150 символов, что недопустимо";
      // Если корректное значение имени, то возвращаем пустую строку
      if (/^[\w.@+-]+$/.test(name)) return "";
      // Если ничего не подошло, то ошибка в ведённом имени, что и отправляем
      return "Некорректное значение имени";
    },
    validPass(pass) {
      if (typeof pass !== "string") return "Введённый пароль не string";
      if (pass.length == 0) return "Введите пароль";
      if (pass.length < 8) return "Пароль не может быть меньше 8 символов";
      if (pass.length > 128) return "Слишком длинный пароль";
      // Если корректное значение пароля, то возвращаем пустую строку
      if (/^(?=.*[A-Z])(?=.*\d).{8,}$/.test(pass)) return "";
      return "Недопустимое значение пароля (должна быть хоть одна Заглавная английская буква и цифра)";
    },

    showError(arrMess) {
      if (arrMess.length > 0) {
        // Отображаем блок сообщений об ошибках валидации
        this.showInfo = true;
      } else this.showInfo = false;

      // Через время закрываем блок сообщений об ошибках
      setTimeout(() => {
        this.messInfo.length = 0;
        this.showInfo = false;
      }, 5000);
    },

    // Запрос авторизации. Если пройден успешно, то авторизуемся,
    // открываем список пользователей
    // и скроллим экран до него.
    sendRequest(user, pass) {
      const authData = {
        username: user,
        password: pass,
      };
      const url =
        "http://emphasoft-test-assignment.herokuapp.com/api-token-auth/";
      fetch(url, {
        method: "post",
        headers: {
          "Content-type": "application/json; charset=utf-8",
        },
        body: JSON.stringify(authData),
      })
        .then((response) => response.json())
        .then((data) => {
          console.log("Request succeeded with JSON response", data);
          if (data.token) {
            console.log("Доступ получен");
            this.accessOk(data.token);
            this.clearInp(); // Чистим инпут
            // Метод Открытия списка (showList = true)
            this.filterShow = true;
            // Пролистывание до верхней границы списка
            this.scroll();
          } else {
            console.log("Неверный логин или пароль");
            this.accessOk("no");
          }
        })
        .catch(function (error) {
          console.log("Request failed", error);
        });
    },

    accessOk(token) {
      console.log(token);
      this.showInfo = true;
      if (token != "no") {
        this.messInfo.push("Доступ получен");
        this.token = token; // Полученный токен отправим в props компонента listsUser
      } else this.messInfo.push("Неверный логин или пароль");

      setTimeout(() => {
        this.messInfo.length = 0;
        this.showInfo = false;
      }, 5000);
    },

    clearInp() {
      this.userName = "";
      this.userPass = "";
    },

    // Проскролить до списка
    scroll() {
      // Убираем в setTimeout, так как сначала надо дождаться открытие блока со списком
      setTimeout(() => {
        let top = window.innerHeight - 40;
        window.scrollTo({ top: top, behavior: "smooth" });
      }, 0);
    },
  },
};
</script>

<style lang="scss" scoped>
.auth {
  height: calc(100vh - 120px);
}
.auth-block {
  display: flex;
  flex-direction: column;
  max-width: 450px;
  height: 100%;
  margin: 0 auto;
  justify-content: center;
  justify-items: center;
  align-items: center;
  position: relative;
}
.auth-block__infoBlock {
  position: fixed;
  top: 70px;
  margin-top: 10px;
  background-color: rgb(252, 58, 58);
  color: white;
  width: 80%;
  min-height: 40px;
  border-radius: 10px;
  opacity: 0.7;
  border: 1px solid red;
  animation: showPanelInfo 0.6s;
}
@keyframes showPanelInfo {
  from {
    top: -50px;
  }
  to {
    top: 70px;
  }
}
.auth-block__inp {
  width: 100%;
  margin-top: 30px;
  line-height: 2rem;
  font-size: 18px;
  border: none;
  box-shadow: 0 1px gray;
  outline: none;
}
.auth-block__inp:focus {
  box-shadow: 0 1px blue;
}
.auth-block__btn {
  position: absolute;
  bottom: 50px;
  justify-self: center;
  width: 200px;
  height: 3rem;
  font-size: 18px;
  background: rgb(18, 161, 25);
  color: white;
  border-radius: 10px;
  border: none;
  box-shadow: none;
  outline: none;
  transition-duration: 0.3s;
}
.auth-block__btn:hover {
  background: rgb(46, 184, 53);
}
.auth-block__btn:focus {
  box-shadow: 2px 2px 2px rgb(76, 101, 134);
}
.auth-block__btn:active {
  background: rgb(13, 128, 19);
}
.users-list span {
  cursor: pointer;
  color: blue;
}

@media screen and (max-width: 457px) {
  .auth {
    height: calc(100vh - 157px);
  }
}
@media screen and (max-width: 283px) {
  .auth {
    height: calc(100vh - 195px);
  }
}
</style>