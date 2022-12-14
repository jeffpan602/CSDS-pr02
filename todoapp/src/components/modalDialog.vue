<template>
    <div class="modalDialog-background">
        <div class="modalDialog">
            <v-card>
                <!-- panel header -->
                <v-card-title class='headline primary justify-left' primary-title style="color: white">
                    <div v-if="isAddTask" key="add">
                        <span class="fa-solid fa-circle-plus" />Add Task
                    </div>
                    <div v-else key="edit">
                        <span class="fa-solid fa-pen-to-square" /> Edit Task
                    </div>
                </v-card-title>
                <!-- panel body-->
                <v-form ref="form" lazy-validation>
                    <v-layout v-if="isAddTask" class="elements mx-5 mb-0 mt-7">
                        <v-text-field label="Title" outlined v-model="title" required :rules="titleValidation" />
                    </v-layout>

                    <v-layout class=" element mx-5 my-2">
                        <v-text-field label="Description" outlined v-model="description" required
                            :rules="[v => !!v || 'Description is Required!']" />
                    </v-layout>

                    <v-layout justify-center class="elements mx-5 my-0">
                        <v-menu ref="menu" :close-on-content-click="false" transition="scale-transition" offset-y
                            min-width="auto">
                            <template v-slot:activator="{ on, attrs }">
                                <v-text-field v-model="computedDateFormatted" label="Deadline"
                                    append-icon="mdi-calendar" v-bind="attrs" v-on="on" readonly outlined required
                                    :rules="[v => !!v || 'Deadline is Required!']" />
                            </template>
                            <v-date-picker v-model="date" no-title scrollable></v-date-picker>
                        </v-menu>
                    </v-layout>

                    <v-radio-group label="Priority" class="elements mx-6 my-0" v-model="priority">
                        <v-layout align-start row d-flex justify-space-between>
                            <v-radio label="Low" value="low" />
                            <v-radio label="Med" value="med" />
                            <v-radio label="High" value="high" />
                        </v-layout>
                    </v-radio-group>

                    <v-layout justify-end class="elements mx-2 mt-2 mb-2">
                        <div v-if="isAddTask" key="add1">
                            <v-btn color="primary" elevation="2" @click="addTask" class="mr-2">
                                <i class="fa-solid fa-circle-plus"></i> ADD
                            </v-btn>
                        </div>
                        <div v-else key="edit1">
                            <v-btn color="primary" elevation="2" @click="editTask" class="mr-2">
                                <i class="fa-solid fa-pen-to-square"></i> EDIT
                            </v-btn>
                        </div>
                        <v-btn color="error" elevation="2" @click="close"> <span class="fa-solid fa-ban" />
                            CANCEL
                        </v-btn>
                    </v-layout>
                </v-form>
            </v-card>
        </div>
    </div>
</template>

<script>
export default {
    name: 'modalDialog',
    props: {
        isVisible: Boolean,
        isAddTask: Boolean,
        existing_description: String,
        existing_date: String,
        existing_priority: String,
        tasks: Array
    },
    methods: {
        close() {
            this.$emit('close')
            this.clear()
        },
        clear() {
            this.$refs.form.resetValidation()

            this.title = ''
            this.description = ''
            this.date = ''
            this.priority = 'low'
        },
        addTask() {
            if (this.$refs.form.validate()) {
                this.$emit('addTask', this.title, this.description, this.formatDate(this.date), this.priority)
                this.clear()
                this.close()
            }
        },
        editTask() {
            if (this.$refs.form.validate()) {
                this.$emit('editTask', this.description, this.formatDate(this.date), this.priority)
                this.clear()
                this.close()
            }
        },
        formatDate(date) {
            if (!date)
                return null;
            const [year, month, day] = date.split('-')
            return `${month}/${day}/${year}`
        },
        parseDate(date) {
            if (!date) return null

            const [month, day, year] = date.split('/')
            return `${year}-${month.padStart(2, '0')}-${day.padStart(2, '0')}`
        },
        getParentData() {
            this.description = this.existing_description
            this.date = this.parseDate(this.existing_date)
            this.priority = this.existing_priority
        },
        isDistinctTitle(title) {
            return !this.tasks.filter(e => e.title === title).length > 0
        }
    },
    watch: {
        date() {
            this.dateFormatted = this.formatDate(this.date)
        },
        isAddTask(i) {
            this.$refs.form.resetValidation();
            if (!i) {
                this.getParentData();
            }
        }
    },
    data() {
        return {
            title: '',
            description: '',
            deadline: '',
            priority: 'low',
            date: '',
            titleValidation: [
                v => !!v || 'Title is Required!',
                v => this.isDistinctTitle(v) || 'Title must be Distinct'
            ]
        }

    },
    computed: {
        computedDateFormatted() {
            return this.formatDate(this.date)
        },
    }
}
</script>

<style>
.modalDialog-background {
    position: fixed;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    z-index: 99;
    background-color: rgba(0, 0, 0, 0.2);
    display: flex;
    justify-content: center;
    align-items: center;
}

.modalDialog {
    background: white;
    box-shadow: 2px 2px 20px 2px;
    overflow-x: auto;
    display: flex;
    flex-direction: column;
}

.modalDialog {
    background: white;
    box-shadow: 2px 2px 20px 1px;
    overflow-x: auto;
    display: flex;
    flex-direction: column;
}

.element {
    margin: 15px 15px;
    padding: 0px;
}
</style>