<template>
    <section class="content">
        <div class="container">
            <br>
            <h1 class="display-4 text-muted">Usuarios</h1>

            <div class="row">

                <div class="col-12">

                    <div v-if="$gate.isAdmin()">
                        <div class="text-right pull-right">
                            <b-btn variant="outline-secondary" @click="newModal">
                                <b-icon icon="plus"></b-icon>
                                nuevo usuarios
                            </b-btn>
                        </div>

                        <table class="table table-hover">
                            <thead>
                                <tr>
                                    <th>ID</th>
                                    <th>Tipo</th>
                                    <th>Nombre</th>
                                    <th>Email</th>
                                    <th>Email de verificacion?</th>
                                    <th>Creado</th>
                                    <th>Action</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr v-for="user in users.data" :key="user.id">

                                    <td>{{ user.id }}</td>
                                    <td class="text-capitalize">{{ user.type }}</td>
                                    <td class="text-capitalize">{{ user.name }}</td>
                                    <td>{{ user.email }}</td>
                                    <td :inner-html.prop="user.email_verified_at | yesno"></td>
                                    <td>{{ user.created_at }}</td>

                                    <td>

                                        <span @click="editModal(user)" class="optionBtn">
                                            <b-icon icon="pencil-square"></b-icon>
                                        </span>
                                        |
                                        <span @click="deleteUser(user.id)" class="optionBtn">
                                            <b-icon icon="trash"></b-icon>
                                        </span>

                                    </td>
                                </tr>
                            </tbody>
                        </table>

                        <!-- /.card-body -->
                        <!-- <div class="card-footer">
                  <pagination :data="users" @pagination-change-page="getResults"></pagination>
              </div> -->
                    </div>
                    <!-- /.card -->
                </div>
            </div>


            <div v-if="!$gate.isAdmin()">
                <not-found></not-found>
            </div>

            <!-- Modal -->
            <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="addNew" aria-hidden="true">
                <div class="modal-dialog" role="document">
                    <div class="modal-content">
                        <div class="modal-header">
                            <h5 class="modal-title" v-show="!editmode">Crear nuevo usuario</h5>
                            <h5 class="modal-title" v-show="editmode">
                                actualizar la informacion del usuario</h5>
                            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                                <span aria-hidden="true">&times;</span>
                            </button>
                        </div>

                        <!-- <form @submit.prevent="createUser"> -->

                        <form @submit.prevent="editmode ? updateUser() : createUser()">
                            <div class="modal-body">
                                <div class="form-group">
                                    <label>Nombre</label>
                                    <input v-model="form.name" type="text" name="name" class="form-control"
                                        :class="{ 'is-invalid': form.errors.has('name') }">
                                    <has-error :form="form" field="name"></has-error>
                                </div>
                                <div class="form-group">
                                    <label>Email</label>
                                    <input v-model="form.email" type="text" name="email" class="form-control"
                                        :class="{ 'is-invalid': form.errors.has('email') }">
                                    <has-error :form="form" field="email"></has-error>
                                </div>

                                <div class="form-group">
                                    <label>Password</label>
                                    <input v-model="form.password" type="password" name="password" class="form-control"
                                        :class="{ 'is-invalid': form.errors.has('password') }" autocomplete="false">
                                    <has-error :form="form" field="password"></has-error>
                                </div>

                                <div class="form-group">
                                    <label>Rol de ususario</label>
                                    <select name="type" v-model="form.type" id="type" class="form-control"
                                        :class="{ 'is-invalid': form.errors.has('type') }">
                                        <option value="">Seleccionar rol de usuario</option>
                                        <option value="admin">Admin</option>
                                        <option value="user">usuario estándar</option>
                                    </select>
                                    <has-error :form="form" field="type"></has-error>
                                </div>
                            </div>
                            <div class="modal-footer">
                                <button type="button" class="btn btn-secondary" data-dismiss="modal">Cerrar</button>
                                <button v-show="editmode" type="submit" class="btn btn-success">Actualizar</button>
                                <button v-show="!editmode" type="submit" class="btn btn-primary">Crear</button>
                            </div>
                        </form>
                    </div>
                </div>
            </div>
        </div>
    </section>
</template>

<script>
export default {
    data() {
        return {
            editmode: false,
            users: {},
            form: new Form({
                id: '',
                type: '',
                name: '',
                email: '',
                password: '',
                email_verified_at: '',
            })
        }
    },
    methods: {

        getResults(page = 1) {

            this.$Progress.start();

            axios.get('api/user?page=' + page).then(({ data }) => (this.users = data.data));

            this.$Progress.finish();
        },
        updateUser() {
            this.$Progress.start();
            // console.log('Editing data');
            this.form.put('api/user/' + this.form.id)
                .then((response) => {
                    // success
                    $('#addNew').modal('hide');
                    Toast.fire({
                        icon: 'success',
                        title: response.data.message
                    });
                    this.$Progress.finish();
                    //  Fire.$emit('AfterCreate');

                    this.loadUsers();
                })
                .catch(() => {
                    this.$Progress.fail();
                });

        },
        editModal(user) {
            this.editmode = true;
            this.form.reset();
            $('#addNew').modal('show');
            this.form.fill(user);
        },
        newModal() {
            this.editmode = false;
            this.form.reset();
            $('#addNew').modal('show');
        },
        deleteUser(id) {
            Swal.fire({
                title: 'Are you sure?',
                text: "You won't be able to revert this!",
                showCancelButton: true,
                confirmButtonColor: '#d33',
                cancelButtonColor: '#3085d6',
                confirmButtonText: 'Yes, delete it!'
            }).then((result) => {

                // Send request to the server
                if (result.value) {
                    this.form.delete('api/user/' + id).then(() => {
                        Swal.fire(
                            'Deleted!',
                            'Your file has been deleted.',
                            'success'
                        );
                        // Fire.$emit('AfterCreate');
                        this.loadUsers();
                    }).catch((data) => {
                        Swal.fire("Failed!", data.message, "warning");
                    });
                }
            })
        },
        loadUsers() {
            this.$Progress.start();

            if (this.$gate.isAdmin()) {
                axios.get("api/user").then(({ data }) => (this.users = data.data));
            }

            this.$Progress.finish();
        },

        createUser() {

            this.form.post('api/user')
                .then((response) => {
                    $('#addNew').modal('hide');

                    Toast.fire({
                        icon: 'success',
                        title: response.data.message
                    });

                    this.$Progress.finish();
                    this.loadUsers();

                })
                .catch(() => {

                    Toast.fire({
                        icon: 'error',
                        title: 'Some error occured! Please try again'
                    });
                })
        }

    },
    mounted() {
        console.log('User Component mounted.')
    },
    created() {

        this.$Progress.start();
        this.loadUsers();
        this.$Progress.finish();
    }
}
</script>
<style scoped>
.content {
    font-family: Arial, Helvetica, sans-serif;
}

.optionBtn:hover {
    cursor: pointer;
}
</style>
