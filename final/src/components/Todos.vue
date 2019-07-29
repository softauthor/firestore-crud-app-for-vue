<template>
    <section>
        <navigation></navigation>
        <h5 class="center-align">To-Dos</h5>
        <ul class="collection with-header">
            <li class="collection-header">
                <div class="row">
                    <div class="input-field col s10">
                        <input id="new_todo" type="text" class="validate" v-model="todo.title" />
                    </div>
                    <div class="input-field col s2">
                        <button class="btn" @click="addTodo">Add</button>
                    </div>
                </div>
            </li>
            <li
                class="collection-item"
                v-for="todo in todos"
                :key="todo.id"
                :class="{ fade: todo.isCompleted }"
            >
                <span class="deleteIcon" @click="deleteToDo(todo.id)">&#10005;</span>
                {{todo.title}}
                <span class="secondary-content">
                    <label>
                        <input
                            type="checkbox"
                            class="filled-in"
                            :checked="todo.isCompleted"
                            @change="updateTodoItem(todo.id, $event)"
                        />
                        <span></span>
                    </label>
                </span>
            </li>
        </ul>
    </section>
</template>

<script>
import navigation from "@/components/NavBar.vue";
import firebase from "firebase";

export default {
    data() {
        return {
            todos: [],
            todo: {
                title: ""
            }
        };
    },
    created() {
        this.getTodos();
    },
    components: { navigation },
    methods: {
        addTodo() {
            firebase
                .firestore()
                .collection("users")
                .doc(firebase.auth().currentUser.uid)
                .collection("todos")
                .add({
                    title: this.todo.title,
                    createdAt: new Date(),
                    isCompleted: false
                });
        },
        async getTodos() {
            var todosRef = await firebase
                .firestore()
                .collection("users")
                .doc(firebase.auth().currentUser.uid)
                .collection("todos");
            todosRef.onSnapshot(snap => {
                this.todos = [];
                snap.forEach(doc => {
                    var todo = doc.data();
                    todo.id = doc.id;
                    this.todos.push(todo);
                });
            });
        },
        updateTodoItem(docId, e) {
            var isChecked = e.target.checked;
            firebase
                .firestore()
                .collection("users")
                .doc(firebase.auth().currentUser.uid)
                .collection("todos")
                .doc(docId)
                .update({
                    isCompleted: isChecked
                });
        },
        deleteToDo(docId) {
            firebase
                .firestore()
                .collection("users")
                .doc(firebase.auth().currentUser.uid)
                .collection("todos")
                .doc(docId)
                .delete();
        }
    }
};
</script>

<style>
.fade {
    opacity: 0.4 !important;
}
li {
    font-size: 1.3em;
}
.collection.with-header {
    max-width: 500px;
    margin: 0 auto;
}
.deleteIcon {
    margin-right: 10px;
    cursor: pointer;
}
.deleteIcon:hover {
    opacity: 0.5;
}
</style>

