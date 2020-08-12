<template>
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-12">
                <ul>
                    <li v-for = " x in myVariable">
                        {{ x.email | upText}}
                    </li>

                    <h1 v-if = "check">Yes</h1>
                    <h1 v-else>No</h1>

                </ul>
            </div>
        </div>
    </div>
</template>


<script>
    export default {

        data()
        {
            return {
                myVariable : {},
                check : true,
            }
        },

        methods : {

            loadUsers() {
                axios.get("api/user")
                .then(( { data } ) => (this.myVariable = data.data) );
            },

        },


        mounted:function() {
            this.loadUsers();
            console.log('Component mounted.')
        }
    }
</script>
