<template>
    <div class="container">
        <div class="row mt-5">
            <div class="col-md-12">
                <div class="card">
                    <div class="card-header">
                        <h3 class="card-title">Users Table</h3>

                        <div class="card-tools">
                            <button class="btn btn-success" @click = "newModal"> Add New
                                <i class="fa fa-user-plus"></i>
                            </button>
                        </div>
                    </div>
                    <!-- /.card-header -->
                    <div class="card-body table-responsive p-0">
                        <table class="table table-hover text-nowrap">
                            <thead>
                                <tr>
                                    <th>ID</th>
                                    <th>Name</th>
                                    <th>Email</th>
                                    <th>Type</th>
                                    <th>Registered At</th>
                                    <th>Modify</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr v-for="user in users" :key="user.id">
                                    <td>{{ user.id }}</td>
                                    <td>{{ user.name }}</td>
                                    <td>{{ user.email }}</td>
                                    <td>{{ user.type | upText }}</td>
                                    <td>{{ user.created_at | myDate }}</td>

                                    <td>

                                        <a href="#" @click="editModal(user)">
                                            <i class="fa fa-edit blue"></i>
                                        </a>

                                        <a href="#" @click="deleteUser(user.id)">
                                            <i class="fa fa-trash red"></i>
                                        </a>

                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                    <!-- /.card-body -->
                </div>
                <!-- /.card -->
            </div>
        </div>


        <!-- Modal -->
        <div class="modal fade" id="addNew" tabindex="-1" aria-labelledby="addNewLabel" aria-hidden="true">
            <div class="modal-dialog modal-dialog-ventered">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 v-show="!editMode" class="modal-title" id="addNew">Add New</h5>
                        <h5 v-show="editMode" class="modal-title" id="addNew">Edit User</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>

                    <form @submit.prevent="editMode ? updateUser() : createUser() ">

                        <div class="modal-body">

                            <div class="form-group">
                                <input v-model="form.name" type="text" id="name" name="name" class="form-control" placeholder="Name"
                                    :class="{ 'is-invalid': form.errors.has('name') }">
                                <has-error :form="form" field="name"></has-error>
                            </div>


                            <div class="form-group">
                                <input v-model="form.email" type="email" id="email" name="email" class="form-control" placeholder="Email"
                                    :class="{ 'is-invalid': form.errors.has('email') }">
                                <has-error :form="form" field="email"></has-error>
                            </div>


                            <div class="form-group">
                                <textarea v-model="form.bio" name="bio" id="bio" class="form-control" placeholder="Short bio for user (Optional)"
                                    :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                                <has-error :form="form" field="bio"></has-error>
                            </div>


                            <div class="form-group">
                                <select name="type" v-model="form.type" id="type" class="form-control"
                                    :class="{ 'is-invalid': form.errors.has('type') }">
                                    <option value="">Select User Role</option>
                                    <option value="admin">Admin</option>
                                    <option value="user">Standard User</option>
                                    <option value="Author">Author</option>
                                </select>
                                <has-error :form="form" field="type"></has-error>
                            </div>



                            <div class="form-group">
                                <input type="password" v-model="form.password" name="password" class="form-control" placeholder="Password" id="password"
                                    :class="{ 'is-invalid': form.errors.has('password') }">
                                <has-error :form="form" field="password"></has-error>
                            </div>




                        </div>


                        <div class="modal-footer">
                            <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                            <button v-show="editMode" type="submit" class="btn btn-success">Update</button>
                            <button v-show="!editMode" type="submit" class="btn btn-primary">Create</button>
                        </div>

                    </form>

                </div>
            </div>
        </div>



    </div>
</template>

<script>
    export default {

        data() {
            return {

                editMode : false,
                users : {},
                form: new Form({
                    id : '',
                    name: '',
                    email: '',
                    password: '',
                    type: '',
                    bio: '',
                    photo: '',

                })
            }
        },

        methods: {

            updateUser() {

                this.$Progress.start();

                this.form.put('api/user/'+this.form.id)
                .then( () => {

                    $('#addNew').modal('hide');

                    swal.fire(
                    'Updated!',
                    'User has been updated.',
                    'success'
                    )

                    this.$Progress.finish();



                    Fire.$emit('AfterCreate');


                })
                .catch( () => {
                    this.$Progress.fail();

                });

            },

            editModal(user){

                this.editMode = true;
                this.form.reset();
                $('#addNew').modal('show');
                this.form.fill(user);

            },

            newModal(){

                this.editMode = false;

                this.form.reset();
                $('#addNew').modal('show');

            },

            deleteUser(id){
                swal.fire({
                    title: 'Are you sure?',
                    text: "You won't be able to revert this!",
                    icon: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Yes, delete it!'
                    }).then((result) => {


                    //Send Request to the server
                    if(result.value) {
                        this.form.delete('api/user/'+id).then( () => {

                            swal.fire(
                            'Deleted!',
                            'Your file has been deleted.',
                            'success'
                            )

                            Fire.$emit('AfterCreate');


                            }).catch( () => {
                                swal("Failed!", "There was something wrong. ", "warning");
                            });
                    }

                })
            },

            loadUsers() {
                axios.get("api/user").then(( { data } ) => (this.users = data.data) );
            },

            createUser() {

                this.$Progress.start();
                
                this.form.post('api/user')
                .then(() => {

                    Fire.$emit('AfterCreate');

                    $('#addNew').modal('hide');

                    toast.fire({
                        icon: 'success',
                        title: 'User Created Successfully'
                    });


                })
                .catch(() => {

                });

                this.$Progress.finish();
            }

        },

        created() {
            this.loadUsers();
            // setInterval( () => this.loadUsers() , 3000);
            Fire.$on('AfterCreate', () => {
                this.loadUsers();
            });
        }
    }

</script>
