<script setup>
import { computed, onBeforeMount, provide, ref } from "vue";
import ListItem from "./components/ListItem.vue";
import ListItemCreate from "./components/ListItemCreate.vue";
// Создаем пустой массив
const ListItems = ref([]);
// Провайдим массив записей (будет иджектится в ListItem.vue)
provide("ListItems", ListItems);
// На этапе до отрисовки списка проверяем есть ли у польвателя сохраненный список, если есть загружем, если нет создаем дефолтный
onBeforeMount(() => {
  localStorage.getItem("ListItems")
    ? (ListItems.value = JSON.parse(localStorage.getItem("ListItems")))
    : localStorage.setItem(
        "ListItems",
        JSON.stringify([
          { id: 0, text: "Сделать дело №1", status: false },
          { id: 1, text: "Сделать дело №2", status: true },
          { id: 2, text: "Сделать дело №3", status: false },
        ])
      );
  ListItems.value = JSON.parse(localStorage.getItem("ListItems"));
});
// Создаем реактивную переменную для фильтрации
const active = ref("allTask");
// Создаем компьютед переменную для определения текущего id
const currentId = computed(
  () => ListItems.value[ListItems.value.length - 1]?.id || 0
);
// Функция, создает новую задачу и добавляет ее в localStorage
const createHandler = (text) => {
  ListItems.value.push({
    id: currentId.value + 1,
    text: text,
    status: false,
  });
  localStorage.setItem("ListItems", JSON.stringify(ListItems.value));
};
// Функция, принимает id и удаляет задачу из массива задач и из хранилища
const deleteHandler = (id) => {
  ListItems.value = ListItems.value.filter((item) => item.id !== id);
  localStorage.setItem("ListItems", JSON.stringify(ListItems.value));
};
//Функция обработчик, устанавливает активный фильтр, в разметке на выбранном фильтре активируется класс 'active'
const filterHandler = (e) => {
  if (e.target.id) active.value = e.target.id;
};
// Компьютед переменная в зависимости от значения активного фильтра возвращает список отфильтрованных задач
const filteredListItems = computed(() => {
  if (active.value === "allTask") {
    return ListItems.value;
  } else {
    return ListItems.value.filter(
      (item) => (active.value === "compliteTask") === item.status
    );
  }
});
</script>

<template>
  <div class="wrapped">
    <div class="wrap">
      <header class="header">
        <h1>To-do-list</h1>
      </header>
    </div>
    <div class="wrap">
      <div class="flex">
        <p>Фильтровать задачи</p>
        <div class="filters" @click="filterHandler">
          <button id="allTask" :class="active === 'allTask' && 'active'">
            Все задачи
          </button>
          <button
            id="compliteTask"
            :class="active === 'compliteTask' && 'active'"
          >
            Выполнены
          </button>
          <button id="inWorkTask" :class="active === 'inWorkTask' && 'active'">
            В работе
          </button>
        </div>
      </div>
      <ul class="list">
        <ListItem
          v-for="item in filteredListItems"
          :key="item.id"
          :item="item"
          @delete-item="deleteHandler(item.id)"
        />
        <ListItemCreate @create-task="createHandler" />
      </ul>
    </div>
  </div>
</template>

<style>
.wrapped {
  display: flex;
  justify-content: flex-start;
  flex-direction: column;
  text-align: center;
}
.wrap {
  width: 1210px;
  margin: 0 auto;
}

@media (width < 1210px) {
  .wrap {
    width: 768px;
  }
}
@media (width < 768px) {
  .wrap {
    width: 310px;
  }
  p {
    align-self: flex-start;
    padding-left: 10px;
  }
  .filters {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }
}

.flex {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.header {
  width: 100%;
}
.filters {
  padding: 10px;
}
.active {
  background-color: #ddffd6;
}
.list {
  padding: 0;
  list-style-position: inside;
}
.footer {
  padding: 20px;
}
button {
  margin-left: 10px;
  border-radius: 5px;
  padding: 5px 20px;
  outline: none;
  border: 1px solid gray;
  background-color: #fff;
  user-select: none;
}
button:hover {
  border: 1px solid gray;
  background-color: #e2e2e2;
}
</style>
