<template>
    <div>
        <div v-for="post in this.posts" class="posts">
            <v-card

                    class="mx-auto"
            >
                <v-list-item>
                    <v-list-item-avatar >
                        <v-img
                                :src="post.userimage"
                        />
                    </v-list-item-avatar>
                    <v-list-item-content>
                        <v-list-item-title style="display: flex" class="headline">{{post.username}}</v-list-item-title>
                    </v-list-item-content>
                </v-list-item>
                <v-img
                        height="800"
                        width="500"
                        class="white--text align-end"
                        :src="post.postimage"
                >
                    <v-card-title>{{post.description}}</v-card-title>
                </v-img>
            </v-card>
        </div>


        <v-app>
            <v-app-bar
                    app
                    color="white"
                    dark
            >
                <router-link to="/posts">
                    <v-avatar :tile="true">
                        <img src="https://image.flaticon.com/icons/png/512/87/87390.png" alt="logo">
                    </v-avatar>
                </router-link>

                <v-spacer/>

                <div v-if="logado">
                    <router-link to="/perfil">
                        <v-btn  class="ma-2" title outlined color="black" >
                            <v-icon left >mdi-account</v-icon>Perfil
                        </v-btn>
                    </router-link>
                </div>

                <div v-if="logado">
                    <router-link to="/dashboard">
                        <v-btn class="ma-2" title outlined color="black" >
                            <v-icon left >mdi-plus</v-icon>New Post
                        </v-btn>
                    </router-link>
                </div>

                <div v-if="logado">
                    <router-link to="/posts">
                        <v-btn class="ma-2" title outlined color="black" @click.prevent="logout()">
                            <v-icon left >mdi-logout</v-icon>Logout
                        </v-btn>
                    </router-link>
                </div>

                <div v-if="!logado">
                    <router-link to="/login">
                        <v-btn class="ma-2" title outlined color="black" >
                            <v-icon left >mdi-login</v-icon>Login
                        </v-btn>
                    </router-link>
                </div>

                <div v-if="!logado">
                    <router-link to="/signup">
                        <v-btn class="ma-2" title outlined color="black" >
                            <v-icon left >mdi-account-plus</v-icon>Sign Up
                        </v-btn>
                    </router-link>
                </div>

            </v-app-bar>
        </v-app>
    </div>
</template>

<script>
    import axios from 'axios'
    export default {
        name: "Posts",
        data: () => ({
            posts:[],
            logado:false
        }),
        created() {
            if (sessionStorage.getItem("IDuser") === null) {

            }else{

                this.logado = true;
            }
        },
        mounted: function () {
            this.getPosts()
        },
        methods:{
            async getPosts(){
                const url = "http://localhost:3000/api/getAllPosts";
                await axios.get(url)
                    .then(response=>{
                        this.posts = response.data.results;
                        // Se user não tem imagem vai ser colocada uma default
                        var i;

                        for (i = 0; i < this.posts.length; i++) {
                            if(this.posts[i].userimage === null || this.posts[i].userimage === "" || this.posts[i].userimage === undefined){
                                this.posts[i].userimage = "https://i.imgur.com/23kxlWn.png";
                            }
                        }
                    })
                    .catch(error =>{
                        console.log("Error Posts: " + error)
                    })
            }, // END GET POSTS
            logout(){

                this.logado = false;
                sessionStorage.removeItem("IDuser");
            }
        }
    }
</script>

<style scoped>
    .posts{
        padding-bottom: 100px;
    }

    a {  text-decoration: none;}
</style>