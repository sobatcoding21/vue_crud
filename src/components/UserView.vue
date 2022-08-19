<template>
    <v-container>

        <v-row class="mb-1 mt-5">
            <v-btn color="primary" dark @click.prevent="tambahUser">
                Tambah Data
            </v-btn>
            <v-dialog v-model="dialog" persistent max-width="600px">
                <v-card>
                    <v-form ref="form">
                        <v-card-title>
                            <span class="text-h5">{{ formTitle }}</span>
                        </v-card-title>
                        <v-card-text>
                            <v-container>
                                <v-alert prominent type="error" v-if="this.errorMessage != ''">
                                    <v-row align="center">
                                        <v-col class="grow">{{ this.errorMessage }}</v-col>
                                    </v-row>
                                </v-alert>
                                <v-row>
                                    <v-col cols="12">
                                        <v-text-field label="Nama Lengkap*" v-model="formData.name" :rules="inputRules">
                                        </v-text-field>
                                    </v-col>
                                    <v-col cols="12">
                                        <v-text-field label="Username*" v-model="formData.username" :rules="inputRules">
                                        </v-text-field>
                                    </v-col>
                                    <v-col cols="12">
                                        <v-text-field v-if="formData.id == ''" label="Password*" type="password"
                                            v-model="formData.password" :rules="inputRules">
                                        </v-text-field>
                                        <v-text-field v-else label="Password*" type="password"
                                            v-model="formData.password">
                                        </v-text-field>
                                    </v-col>
                                    <v-col cols="12">
                                        <v-text-field label="Alamat*" v-model="formData.address" :rules="inputRules">
                                        </v-text-field>
                                    </v-col>
                                    <v-col cols="12">
                                        <v-text-field label="No. Telp*" v-model="formData.phone" :rules="inputRules">
                                        </v-text-field>
                                    </v-col>
                                </v-row>
                            </v-container>
                        </v-card-text>
                        <v-card-actions>
                            <v-spacer></v-spacer>
                            <v-btn @click="dialog = false">
                                Batal
                            </v-btn>
                            <v-btn v-if="formData.id == ''" color="primary" @click.prevent="addUser">
                                Simpan
                            </v-btn>
                            <v-btn v-else color="primary" @click.prevent="updateUser(formData.id)">
                                Update
                            </v-btn>
                        </v-card-actions>
                    </v-form>
                </v-card>
            </v-dialog>
        </v-row>

        <v-simple-table>
            <template v-slot:default>
                <thead>
                    <tr>
                        <th class="text-left">Nama</th>
                        <th class="text-left">Username</th>
                        <th class="text-left">Alamat</th>
                        <th class="text-left">No. Telp</th>
                        <th class="text-left">Aksi</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="item in users" :key="item.name">
                        <td>{{ item.name }}</td>
                        <td>{{ item.username }}</td>
                        <td>{{ item.address }}</td>
                        <td>{{ item.phone }}</td>
                        <td>
                            <v-btn @click.prevent="editUser(item)">
                                Edit
                            </v-btn>
                            <v-btn color="error" @click.prevent="deleteUser(item.id)">
                                Hapus
                            </v-btn>
                        </td>
                    </tr>
                </tbody>
            </template>
        </v-simple-table>
    </v-container>

</template>

<script>
import axios from 'axios';

export default {
    name: 'UserView',
    components: {
        //FormView
    },
    data() {
        return {
            dialog: false,
            formTitle: 'Tambah User',
            users: [],
            errorMessage: '',
            formData: {
                id: '',
                name: '',
                username: '',
                password: '',
                address: '',
                phone: ''
            },
            inputRules: [
                v => v && v.length > 0 || 'Field harus diisi'
            ]
        }
    },
    mounted() {
        this.getDataUsers();
    },
    methods: {
        tambahUser() {
            this.dialog = true;
            this.resetForm();
        },
        resetForm() {
            this.formTitle = 'Tambah User';
            this.formData.id = '';
            this.formData.name = '';
            this.formData.username = '';
            this.formData.address = '';
            this.formData.phone = '';
        },
        getDataUsers() {
            axios.get('http://55.55.55.21/restapi_ci3/users').then(res => {
                console.log(res);
                this.users = res.data.data //respon dari rest api dimasukan ke users
            }).catch((error) => {
                console.log(error);
                this.errorMessage = error.response.data.message;
            })
        },
        addUser() {
            if (this.$refs.form.validate()) {
                var postData = new FormData();
                postData.append("name", this.formData.name);
                postData.append("username", this.formData.username);
                postData.append("password", this.formData.password);
                postData.append("address", this.formData.address);

                axios.post('http://55.55.55.21/restapi_ci3/users/save', postData
                ).then(res => {
                    console.log(res);
                    this.dialog = false;
                    this.resetForm();
                    this.getDataUsers();
                }, (error) => {
                    console.log(error.response.data);
                    this.errorMessage = error.response.data.message;
                });

                /*var formdata=new FormData();
                    formdata.append("name", this.formData.name);
                    formdata.append("username", this.formData.username);
                    formdata.append("password", this.formData.password);
                    formdata.append("address", this.formData.address);
                axios({
                    method: 'POST',
                    url: 'http://55.55.55.21/restapi_ci3/users/save',
                    data: formdata,
                })
                    .then((response) => {
                        console.log(response.data);


                    }, (error) => {
                        console.log(error.response.data);
                        this.errorMessage = error.response.data.message;
                    });*/
            }

        },
        editUser(item) {
            this.dialog = true;
            this.formTitle = 'Edit User';
            this.formData.id = item.id;
            this.formData.name = item.name;
            this.formData.username = item.username;
            this.formData.address = item.address;
            this.formData.phone = item.phone;
        },
        updateUser(id) {
            var postData = new FormData();
            postData.append("name", this.formData.name);
            postData.append("username", this.formData.username);
            postData.append("password", this.formData.password);
            postData.append("address", this.formData.address);

            axios.post('http://55.55.55.21/restapi_ci3/users/' + id, postData
            ).then(res => {
                console.log(res);
                this.dialog = false;
                this.resetForm();
                this.getDataUsers();

                this.$swal({
                    text: res.data.message,
                    icon: res.data.success == true ? 'success' : 'warning',
                    confirmButtonColor: '#3085d6',
                })

            }, (error) => {
                console.log(error.response.data);
                this.errorMessage = error.response.data.message;
            });
        },
        deleteUser(id) {
            this.$swal({
                title: 'Kamu Yakin?',
                text: "Tindakan ini akan menghapus secara permanent!",
                type: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#3085d6',
                cancelButtonColor: '#d33',
                confirmButtonText: 'Iya, Lanjutkan!'
            }).then((result) => {
                if (result.value) {
                    axios.get('http://55.55.55.21/restapi_ci3/users/delete/' + id).then(res => {
                        console.log(res);
                        this.getDataUsers();

                        this.$swal({
                            text: res.data.message,
                            icon: res.data.success == true ? 'success' : 'warning',
                            confirmButtonColor: '#3085d6',
                        })
                    }).catch((error) => {
                        console.log(error);
                        this.errorMessage = error.response.data.message;
                    })
                }
            })
        }
    }
}
</script>