<template>
    <div class="wrapper wrapper-full-page">
        <div class="full-page login-page" filter-color="black" data-image="images/login/background.jpg">
            <div class="content">
                <div class="container">
                    <div class="row">
                        <div class="spacer"></div>
                        <div class="col-md-4 col-sm-6 col-md-offset-4 col-sm-offset-3">
                            <form>
                                <div class="card card-login card-hidden">
                                    <div v-if="isLocalServer">
                                        <div class="card-header text-center" data-background-color="rose">
                                            <img src="images/login/rfid.svg" alt="rfid logo">
                                            <h4 class="card-title">Scan your matric card to login</h4>
                                        </div>
                                        <p class="category text-center" v-if="!matric_uuid">
                                            Or Be Classical
                                        </p>
                                    </div>
                                    <div class="content">
                                        <div class="input-group" v-if="!matric_uuid">
                                            <span class="input-group-addon">
                                                <i class="material-icons">email</i>
                                            </span>
                                            <div class="form-group label-floating">
                                                <label class="control-label">Email address</label>
                                                <input type="email" v-model="form.email" class="form-control">
                                            </div>
                                        </div>
    
                                        <div class="alert alert-info" v-if="matric_uuid">
                                            Card Id: {{ matric_uuid }}
                                        </div>
                                        
                                        <div class="input-group">
                                            <span class="input-group-addon">
                                                <i class="material-icons">lock_outline</i>
                                            </span>
                                            <div class="form-group label-floating">
                                                <label class="control-label">Password</label>
                                                <input type="password" v-model="form.password" class="form-control">
                                            </div>
                                        </div>
                                    </div>
                                    <div class="footer text-center">
    
                                        <div class="alert alert-danger" v-if="form.errors.any()" style="margin: 0 10px;" data-notify="container">
                                            <span data-notify="message">
                                                Opps, Please check your credentials..
                                            </span>
                                        </div>
                                        
                                        <button @click.prevent="submit()" class="btn btn-rose btn-wd btn-lg" :disabled="!this.form.valid()">Login</button>
                                        <a href="/register" class="btn btn-simple btn-wd btn-lg">Or SignUp</a>
                                    </div>
                                </div>
                            </form>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <loader v-if="isLoading"></loader>
    </div>
</template>

<script>
    import loader from '../components/Loader.vue'
    import Form from '../core/Form'
    import Echo from 'laravel-echo'

    export default {
        data(){
            return {
                form: new Form({
                    "email" : "",
                    "password" : ""
                }),
                isLoading: false,
                matric_uuid: null,
                url: "/login"
            }
        },
        props: ['server'],
        computed: {
            isLocalServer() {
                return this.server == "local";
            },
        },
        methods: {
            submit() {
                if( ! this.form.valid() ) return;
                
                this.isLoading = true;
                this.form.post(this.url).then(response => {
                    this.isLoading = false;
                    window.location.replace("/portal");
                }).catch(() => {
                    alert("Opps, something went wrong!");
                    this.isLoading = false;
                });
            },
            validateMatricUUID(matric_uuid) {
                this.isLoading = true;
                axios.post('/login/validate/matric_uuid', {
                    'matric_uuid' : matric_uuid
                }).then(response => {
                    this.form.email = response.data.data;
                    this.isLoading = false;
                }).catch(error => {
                    this.isLoading = false;
                });
            }
        },
        mounted() {
            if(this.isLocalServer){
                const io = window.io = require('socket.io-client');
                let echo = new Echo({
                    broadcaster: 'socket.io',
                    host: window.location.hostname + ':6001'
                });

                echo.channel('cards')
                    .listen('NewCard', (e) => {
                        this.matric_uuid = e.card;
                        this.validateMatricUUID(e.card);
                    });
            }
        },
        components: {
            loader
        }
    }
</script>