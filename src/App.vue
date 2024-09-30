
<script setup lang="ts">
 import { ref, onMounted, computed } from 'vue';
 import TodoComponent from './components/TodoComponent.vue';
 import { Todo } from './models/Todo'; // Import de l'interface Todo
const monTableau = ref<Todo[]>([]);
 onMounted(async () => {
 const todosRequest = await fetch('http://localhost:3001/todos');
 const todos: Todo[] = await todosRequest.json();
 monTableau.value = [...todos];
 });


// Définir un tableau de tâches avec l'état des cases

// Calculer le nombre de cases cochées
const countChecked = computed(() => {
  return monTableau.value.filter((item) => item.done).length
})

const TaskFinish = computed(() => {
  return monTableau.value.length > 0 && monTableau.value.every((item) => item.done)
})

const newTodo = ref('')

// Fonction pour ajouter un nouveau todo
const addTodo = () => {
  if (newTodo.value.trim() !== '') {
    monTableau.value.push({ todo: newTodo.value.trim(), done: false })
    newTodo.value = '' // Réinitialiser le champ de saisie après ajout
  }
}

const onTodoInput = async (newTodoValue: Todo, index: number) => {
 monTableau.value[index] = newTodoValue;
 // Appel PUT pour mettre à jour le todo côté serveur
 await fetch(`http://localhost:3001/todos/${newTodoValue.id}`, {
 method: 'PUT',
 headers: {
 'Content-Type': 'application/json',
 },
 body: JSON.stringify(newTodoValue),
 });
 console.log('monTableau est mis à jour et la modification est envoyée au serveur');
 };

 const onTodoDelete = async (id: number, index: number) => {
 // Suppression sur le serveur
 await fetch(`http://localhost:3001/todos/${id}`, {
 method: 'DELETE',
 });
 // Mise à jour locale du tableau
 monTableau.value.splice(index, 1);
 console.log('Todo supprimé');
 };
</script>

<template>
  <h1>Todo List</h1>
  <br />

  <span>
    <input v-model="newTodo" placeholder="Ajouter une nouvelle tâche" />
    <button @click="addTodo">Ajouter</button>
  </span>
  <br />

  <TodoComponent
    v-for="(element, index) in monTableau"
    :todo="element"
    v-bind:key="element.id
    "
    @onInput="onTodoInput($event, index)"
  />
  <button @click="onTodoDelete(element.id, index)">Supprimer</button>
  <br />
  <br />
  <br />

  <h3>Nombre de tâches complètes : {{ countChecked }}</h3>
  <p v-if="TaskFinish">Félicitations ! Toutes les tâches sont complètes !</p>
</template>

<style scoped></style>
