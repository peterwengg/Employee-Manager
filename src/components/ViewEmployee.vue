<template>
    <div id="view-employee">
        <ul class="collection with-header">
            <li class="collection-header"><h4>{{ name }}</h4></li>
            <li class="collection-item">Employee ID: {{ employee_id }}</li>
            <li class="collection-item">Department: {{ dept }}</li>
            <li class="collection-item">Position: {{ position }}</li>
        </ul>
        <router-link to="/" class="btn grey">Back</router-link>
        <button @click="deleteEmployee" class="btn red">Delete</button>
        <div class="fixed-action-btn">
            <router-link v-bind:to="{name: 'edit-employee', params: {employee_id: employee_id}}" 
            class="btn-floating btn-large red">
                <i class="fa fa-pencil"></i>
            </router-link>
        </div>
    </div>
</template>

<script>
    import db from './firebaseInit'
    export default {
        name: 'view-employee',
        data (){
            return {
                employee_id: null,
                name: null,
                dept: null,
                position: null
            }
        },

        //Needed to fetch the data
        beforeRouteEnter (to, from, next){
            db.collection('employees').where('employee_id', '==', to.params.employee_id).get()
            .then(querySnapshot => {
                querySnapshot.forEach(doc => {

                    //Have to use vm to access instance's components
                    //This is done because this navigation guard is called before the component is created.   

                    next(vm => { //Assigning all the fields that will be added to the template
                        vm.employee_id = doc.data().employee_id
                        vm.name = doc.data().name
                        vm.dept = doc.data().dept
                        vm.position = doc.data().position
                    })
                })
            })
        },

        //We can use the watch option to trigger / change something whenever a reactive property changes
        //For this case, after beforeRouteEnter is used to fetch data, page does not refresh and the employee_id will
        //not change in the component state. Thus need watcher to look out for route param change and then change ID
        watch: {
            '$route' : 'fetchData' //Setting route to a method called fetchData
        },

        methods: {
            fetchData(){
                db.collection('employees').where
                ('employee_id', '==', this.$route.params.employee_id).get() //$ sign mean public instance property
                .then(querySnapshot => {
                    querySnapshot.forEach(doc => {
                        this.employee_id = doc.data().employee_id
                        this.name = doc.data().name
                        this.dept = doc.data().dept
                        this.position = doc.data().position
                    })
                })
            },

            deleteEmployee(){
                if(confirm('Are you sure?')){
                    db.collection('employees').where
                    ('employee_id', '==', this.$route.params.employee_id).get()
                    .then(querySnapshot => {
                        querySnapshot.forEach(doc => {
                            doc.ref.delete();
                            this.$router.push('/') //To navigate to a different URL
                    })
                })
                }
            }
        }
    }
</script>