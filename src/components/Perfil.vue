<template >
    <div>
        <!-- User image and username -->
        <div class="Header">
            <!-- @click="imageClicked()  -->
            <v-avatar class="avatar" @click.stop="dialog = true" :key="forcerender"
                      size="140"
            >
                <img
                        :src="this.posts[0].userimage"
                        alt="user image"
                >
            </v-avatar>
            <p style="font-size: 40px" >{{this.posts[0].username}}</p>
        </div>

        <!-- Pop up image upload-->
        <div>
            <v-row justify="center">
                <v-dialog
                        v-model="dialog"
                        max-width="500"
                        max-height="500"

                >
                    <v-card>
                        <v-card-title class="headline">Choose a new profile image</v-card-title>

                        <br>
                        <br>
                        <div >
                            <input style="display: none" type="file" @change="onFileSelected" ref="fileInput" >
                            <v-btn color="success" style="margin-left: 200px;" @click="$refs.fileInput.click()">Pick a photo</v-btn>
                            <br> <br>
                            <p v-if="imageSelected" style="margin-left: 220px" >Photo: {{this.imageName}}</p>
                            <br>
                            <v-btn color="success" style="margin-left: 185px" class="dialogChange" v-if="imageSelected" @click="upload" >Change Image {{this.percentage}}</v-btn>
                            <br><br>
                            <div v-if="showAnimation" id="app" style="padding-left: 250px">
                                <half-circle-spinner
                                        :animation-duration="1000"
                                        :size="60"
                                        :color="'#ff1d5e'"
                                />
                            </div>
                        </div>
                        <br>
                        <br>

                    </v-card>
                </v-dialog>
            </v-row>
        </div>

        <!-- POSTS -->
         <div class="posts grid">
             <div v-for="post in this.posts" class="posts">
                 <v-card
                         class="mx-auto"
                 >
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

                <div >
                    <router-link to="/dashboard">
                        <v-btn class="ma-2" title outlined color="black" >
                            <v-icon left >mdi-plus</v-icon>New Post
                        </v-btn>
                    </router-link>
                </div>

                <div >
                    <router-link to="/posts">
                        <v-btn class="ma-2" title outlined color="black" @click.prevent="logout()">
                            <v-icon left >mdi-logout</v-icon>Logout
                        </v-btn>
                    </router-link>
                </div>
            </v-app-bar>
        </v-app>

    </div>
</template>

<script>
    import axios from "axios";
    import {HalfCircleSpinner} from 'epic-spinners'

    export default {
        name: "Perfil",
        components: {
            HalfCircleSpinner
        },
        data: () => ({
            posts:[],
            logado:false,
            userimageClicked: false,
            imageName:"",
            imageSelected:false,
            percentage:"",
            imageLink:"",
            dialog:"",
            forcerender:1,
            showAnimation: false
        }),
        created() {
            if (sessionStorage.getItem("IDuser") === null) {
                this.$router.push("/posts")
            }else{
               // login feito
            }
        },
        mounted: function () {
            this.getPosts()
        },
        methods:{
            onFileSelected(event){
                this.selectedFile = event.target.files[0];
                this.imageName = this.selectedFile.name;
                this.imageSelected = true;
            },
            logout(){
                this.logado = false;
                sessionStorage.removeItem("IDuser");
                this.$router.push("/posts");
            },
            async getPosts(){
                const url = "http://localhost:3000/api/getAllPostsByUser" + "?" + "IDuser=" + sessionStorage.getItem("IDuser");
                await axios.get(url)
                    .then(response=>{
                        this.posts = response.data.results;

                        var i;
                        for (i = 0; i < this.posts.length; i++) {
                            if(this.posts[i].userimage === null || this.posts[i].userimage === "" || this.posts[i].userimage === undefined ){
                                this.posts[i].userimage = "https://i.imgur.com/23kxlWn.png";
                            }
                        }
                    })
                    .catch(error =>{
                        console.log("Error perfil: " + error)
                    })
            },
            imageClicked(){
                this.userimageClicked = true;
            },
            async upload(){
                this.showAnimation = true;
                const fd = new FormData();
                fd.append('image',this.selectedFile, this.selectedFile.name);
                const url = "https://api.imgur.com/3/image";
                const ClientID = "11a345230020f89";
                await axios.post(url,fd,{ headers: { Authorization: `Client-ID ${ClientID}`} ,onUploadProgress: uploadEvent =>{
                        this.showAnimation = true;
                        console.log("Upload progress: " +  Math.round(uploadEvent.loaded / uploadEvent.total * 100) + "%" );
                        this.percentage = Math.round(uploadEvent.loaded / uploadEvent.total * 100) + "%";
                    }})
                    .then(res =>{
                        this.imageLink = res.data.data.link;
                        this.changeUserImage();
                    })
                    .catch(err =>{
                        console.log(err);
                    })
            },
            async changeUserImage(){

                const url = "http://localhost:3000/api/insereUserImage";
                var dados = {
                    userimage:this.imageLink,
                    IDuser: sessionStorage.getItem("IDuser")
                };
                await axios.put(url,dados
                ).then(response=>{
                    this.showAnimation = false;
                    this.dialog = false;
                    location.reload(); // reload pagina
                }).catch(error=>{
                    console.log("Erro insirir BD: " + error);
                })
            }
        }
    }
</script>

<style lang="css" scoped>
    .posts{
        margin-bottom: 70px;
        padding-right: 35px;
    }
    .grid {
        display: grid;
        grid-gap: 1px;
        grid-template-columns: repeat(3, 1fr);
    }
    .avatar{
        cursor: pointer;
    }

    a {  text-decoration: none;}
</style>