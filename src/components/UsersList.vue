<template>
  <div class="user-list">
    <!-- HEADER -->
    <div class="user-list__header">
      <!-- Header ID -->
      <div v-if="sortName == 'sortUp' || ''">
        <span
          @click="sortID()"
          class="user-list__header_title mdi mdi-18px mdi-sort-numeric-descending"
        >ID</span>
      </div>
      <div v-else>
        <span
          @click="sortID()"
          class="user-list__header_title mdi mdi-18px mdi-sort-numeric-ascending"
        >ID</span>
      </div>
      <!-- Header UserName -->
      <div v-if="!fShow">
        <span
          class="user-list__header_title mdi mdi-18px mdi-filter"
          @click="fShow = true, addFocusToInp()"
        >User name</span>
      </div>
      <div v-else>
        <span
          class="user-list__header_title mdi mdi-18px mdi-filter-remove filerActive"
          @click="fShow = false"
        >User name</span>
      </div>
      <!-- Header FirstName -->
      <div>
        <span>First name</span>
      </div>
    </div>
    <!-- Строка фильтрации -->
    <div class="filter-line" v-if="fShow">
      <input
        type="text"
        class="user-list__header_filter-inp"
        ref="inpFocus"
        placeholder="Введите имя"
        v-model="userNameSearch"
      />
    </div>

    <!-- СПИСОК Пользователей -->
    <div class="user-list__content" v-for="item of uList[0]" :key="item.id">
      <span>{{item.id}}</span>
      <span>{{item.username}}</span>
      <span>{{item.first_name}}</span>
    </div>
  </div>
</template>

<script>
export default {
  name: "UsersList",
  props: {
    token: String,
  },
  data() {
    return {
      usersList: [],
      cacheUsersList: [], // Для кэширования списка пользователей полученных с сервера
      sortName: "",
      fShow: false,
      userNameSearch: "",
    };
  },
  // Если закрыть окно фильтра, то фильтрация должна сброситься
  watch: {
    fShow() {
      this.userNameSearch = "";
      this.usersList[0] = [...this.cacheUsersList[0]];
    },
  },
  computed: {
    uList() {
      /*
      Правило такое, что можно находить слова без учёта регистра.
      При этом поиск по не точному совпадению 
      Т.е. Поисковый запрос "ра" выдаст варианты "Ира" "Юра" "ира" "Рафаель"...
      */

      // Если строка пуста, то отображать весь список
      if (this.userNameSearch.length == 0) {
        console.log("Пока пустое значение в поисковом имени");
        /*
        Если инпут пустой, то ничего фильтровать не требуется, а значит 
        отображаемый массив должен быть = основному (кешированному) массиву.
        Но проверку осуществить можно, только если в массиве есть значения 
        (usersList.length != undefined), а иначе ошибка выпадет.
        */
        if (this.usersList.length) {
          if (this.usersList[0].length != this.cacheUsersList[0].length)
            this.usersList[0] = [...this.cacheUsersList[0]];
        }
      } else {
        this.usersList[0] = this.cacheUsersList[0].filter((dataUser) => {
          let sUser = this.userNameSearch.toLowerCase();
          if (dataUser.username.toLowerCase().indexOf(sUser) != -1)
            return dataUser;
        });
      }
      return this.usersList;
    },
  },
  methods: {
    autorization() {
      let options = {};
      if (!options.headers) options.headers = {};
      options.headers.Authorization = `Token ${this.token}`;
      return options;
    },
    requestUserList(options) {
      // options - это объект с токеном для авторизации
      console.log("Пробуем авторизовано получить список пользователей");
      const url =
        "http://emphasoft-test-assignment.herokuapp.com/api/v1/users/";
      fetch(url, options)
        .then((response) => response.json())
        .then((res) => {
          this.usersList.push(res);
          console.log(this.usersList);
          // Чтобы полный список не изменялся
          // и за ним не пришлось обращаться повторно на сервер, мы его кэшируем
          this.cacheUsersList.push(this.usersList[0]);
        });
    },
    sortID() {
      if (this.usersList.length) {
        // Сортируем по возрастанию или убыванию в зависимости от прошлого состояния
        switch (this.sortName) {
          case "sortUp":
            this.usersList[0].sort((a, b) =>
              a.id > b.id ? 1 : b.id > a.id ? -1 : 0
            );
            this.sortName = "sortDown";
            break;
          case "sortDown":
            this.usersList[0].sort((a, b) =>
              a.id < b.id ? 1 : b.id < a.id ? -1 : 0
            );
            this.sortName = "sortUp";
            break;
          default:
            this.usersList[0].sort((a, b) =>
              a.id > b.id ? 1 : b.id > a.id ? -1 : 0
            );
            this.sortName = "sortDown";
        }
      }
    },

    addFocusToInp() {
      this.$nextTick(() => {
        this.$refs.inpFocus.focus();
      });
    },
  },

  mounted() {
    // autorization() возвращает токен
    this.requestUserList(this.autorization());
  },
};
</script>

<style scoped lang="scss">
.filterIco {
  cursor: pointer;
}
.user-list {
  min-height: 95vh;
}
.user-list__header {
  display: grid;
  grid-template-columns: 1fr 3fr 3fr;
  font-weight: bold;
  font-size: 24px;
  color: gray;
}
.user-list__header div {
  align-self: center;
  border: 1px solid gray;
}
.user-list__header_title {
  cursor: pointer;
}
.filerActive {
  color: rgb(15, 15, 180);
}
.filter-line {
  padding: 10px;
  background-color: rgba(32, 32, 95, 0.1);
  box-shadow: 0 1px 2px gray;
  border-radius: 0 0 10px 10px;
}
.user-list__header_filter-inp {
  font-size: 18px;
  line-height: 1.5rem;
  max-width: 100%;
  margin-bottom: 10px;
  border: none;
  outline: none;
}

.user-list__content {
  display: grid;
  grid-template-columns: 1fr 3fr 3fr;
  border-bottom: 1px solid rgba(128, 128, 128, 0.3);
  line-height: 1.5rem;
  justify-items: left;
}
.user-list__content span {
  white-space: pre-wrap;
  word-wrap: break-word;
  padding-left: 4px;
  padding-right: 2px;
}

@media screen and (max-width: 500px) {
  .user-list__header {
    font-size: 16px;
  }
  .user-list__header_filter-inp {
    font-size: 16px;
  }
  .user-list__content {
    font-size: 14px;
  }
}
</style>
