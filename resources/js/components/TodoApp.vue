<template>
<div class="container">

<h1>TODO List</h1>

<div class="add-task">
    <input v-model="newTask" placeholder="Nová úloha">
    <button @click="addTask">Pridať</button>
</div>

<ul class="task-list">

<li v-for="task in tasks" :key="task.id" class="task">

    <!-- EDIT MODE -->
    <div v-if="editingTask === task.id" class="edit-mode">

    <input v-model="editTitle">

    <div class="edit-actions">
    <button @click="updateTask(task)">Uložiť</button>
    <button @click="cancelEdit">Zrušiť</button>
    </div>

    </div>

    <!-- NORMAL MODE -->
    <div v-else>

        <span :class="{done: task.completed}">
            {{ task.title }}
        </span>

        <div class="actions">
            <button @click="startEdit(task)">Edit</button>
            <button class="complete" @click="completeTask(task)">✓</button>
            <button class="delete" @click="deleteTask(task)">✕</button>
        </div>

    </div>

</li>

</ul>

</div>
</template>

<script>
export default {

data(){
    return{
        tasks:[],
        newTask:'',

        editingTask:null,
        editTitle:''
    }
},

mounted(){
    this.fetchTasks()
},

methods:{

    async fetchTasks(){

    const res = await fetch('/api/tasks')
    const data = await res.json()

    this.tasks=data

    },

    async addTask(){

        if(!this.newTask) return

        await fetch('/api/tasks',{
            method:'POST',
            headers:{
            'Content-Type':'application/json',
            'Accept':'application/json'
            },
            body:JSON.stringify({
                title:this.newTask
            })
        })

        this.newTask=''
        this.fetchTasks()

    },

    startEdit(task){

        this.editingTask = task.id
        this.editTitle = task.title

    },

    cancelEdit(){

        this.editingTask = null
        this.editTitle = ''

    },

    async updateTask(task){

        await fetch('/api/tasks/'+task.id,{
            method:'PUT',
            headers:{
                'Content-Type':'application/json'
            },
            body:JSON.stringify({
            title:this.editTitle
            })
        })

        this.editingTask=null
        this.editTitle=''

        this.fetchTasks()

    },

    async deleteTask(task){

        await fetch('/api/tasks/'+task.id,{
            method:'DELETE'
        })

        this.fetchTasks()

    },

    async completeTask(task){

        await fetch('/api/tasks/'+task.id+'/complete',{
        method:'PATCH'
        })

        this.fetchTasks()

    }
}
}
</script>