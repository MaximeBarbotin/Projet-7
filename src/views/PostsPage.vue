<template>
  <div id="app">
    <div class="PostTemplateBody">
      <!--Aside Left-->
      <aside>
        <div class="AsideMenu">
          <img
            src="../assets/images/icon-left-font-monochrome-white.png"
            alt=""
          />
          <ul>
            <a class="leftMenu" href="#"><i class="fa fa-home" aria-hidden="true"></i><li>Accueil</li></a>
            <a class="leftMenu" href="#"><i class="fa fa-search" aria-hidden="true"></i><li>Explorer</li></a>
            <a class="leftMenu" href="#"><i class="fa fa-bell" aria-hidden="true"></i><li>Notifications</li></a>
            <a class="leftMenu" href="#"><i class="fa fa-envelope" aria-hidden="true"></i><li>Messages</li></a>
            <a class="leftMenu" href="/profile"><i class="fa fa-user" aria-hidden="true"></i><li>Mon profil</li></a>
            <a class="leftMenu" href="#" @click="logout($event)"><i class="fa fa-power-off" aria-hidden="true"></i><li>Déconnexion</li></a>
          </ul>
        </div>
      </aside>
      <!--Post content-->
      <main>
        <div class="LeftLine">
          <div class="RightLine">
            <div class="container">
              <form class="post" @submit="createPost($event)">
                <h2>Publier</h2>
                <input
                  type="text"
                  name="title"
                  id="titleArea"
                  placeholder="Votre titre"
                  required
                />
                <textarea
                  id="PostArea"
                  name="description"
                  rows="5"
                  placeholder="Ecrivez ici..."
                ></textarea>
                <label for="AddIMGG" id="AddIMG">Ajouter un média</label>
                <input type="file" id="AddIMGG" name="image" />
                <button type="submit" id="publier">Publier</button>
              </form>
            </div>
            <div v-for="post in posts" :key="post.id">
              <PostComponent
                :post="post"
                :addLike="addLike"
                :deletePost="deletePost"
                :modifyPostParent="modifyPost"
              />
            </div>
          </div>
        </div>
      </main>
      <!--Aside RIGHT-->
      <aside>
        <div class="AsideFriend">
          <div class="FriendAsideContent">
            <h2 class="RelationTitle">Relations</h2>
            <ul v-for="friend in friends" :key="friend.name">
              <FriendComponent :friend="friend" />
            </ul>
          </div>
        </div>
      </aside>
    </div>
  </div>
</template>

<script>
import PostComponent from "../components/PostComponent.vue";
import FriendComponent from "../components/FriendComponent.vue";

import axios from "axios";

export default {
  name: "PostsPage",
  components: {
    PostComponent,
    FriendComponent,
  },
  data: function () {
    return {
      userConnection: false,

      posts: [],
      friends: [
        //liste d'amis data
        {
          name: "Laure Delacour",
          image: "ProfilPictures/dwfscgcdhfvjhk.jpg",
        },
        {
          name: "Remi Michel",
          image: "ProfilPictures/fgdrthfygjuhh.jpg",
        },
        {
          name: "Anne Delaune",
          image: "ProfilPictures/michelle-prince-profile-img.png",
        },
        {
          name: "Maxime Barbotin",
          image: "ProfilPictures/PPP.jpg",
        },
        {
          name: "Christophe Lemaitre",
          image: "ProfilPictures/profile-img4.jpg",
        },
        {
          name: "Yann Dufour",
          image: "ProfilPictures/vfsdghkjhk.jpg",
        },
        {
          name: "Antoine Peltier",
          image: "ProfilPictures/photo-profil_301783868.jpg",
        },
        {
          name: "Nicolas Charpentier",
          image: "ProfilPictures/fssdghf.jpg",
        },
        {
          name: "Alexandre Chemon",
          image: "ProfilPictures/csfdgdfhtjk.jpg",
        },
      ],
      counter: 0, //compteur de like
    };
  },
  created() {
    const token = localStorage.getItem("groupomania_token");
    axios
      .get("http://localhost:3000/api/posts", {
        headers: { Authorization: "Bearer " + token },
      })
      .then((d) => {
        this.posts = d.data;
      });

    //TODO: Envoyer une requete à l'API pour récupérer le listing de posts et le listing friends
    //api.getPosts().then(d => this.posts = d)
    //api.getFriends().then(d => this.friends = d)
  },
  methods: {
    addLike(postId) {
      const token = localStorage.getItem("groupomania_token");
      axios
        .post(
          "http://localhost:3000/api/posts/" + postId + "/like",
          {},
          {
            headers: { Authorization: "Bearer " + token },
          }
        )
        .then((d) => {
          this.posts = this.posts.map((post) => {
            if (post.id === postId) {
              if (d.status === 201) {
                post.likes += 1;
                post.liked = 1;
              } else {
                post.likes -= 1;
                post.liked = 0;
              }
            }
            return post;
          });
        });
    },
    //Création d'un POST
    createPost(event) {
      event.preventDefault();
      var bodyFormData = new FormData();
      bodyFormData.append("title", event.target.title.value);
      bodyFormData.append("description", event.target.description.value);
      bodyFormData.append("image", event.target.image.files[0]);

      const token = localStorage.getItem("groupomania_token");
      axios
        .post("http://localhost:3000/api/posts", bodyFormData, {
          headers: {
            Authorization: "Bearer " + token,
            "Content-Type": "multipart/form-data",
          },
        })
        .then((d) => {
          this.posts = [d.data.post].concat(this.posts);
          event.target.title.value = "";
          event.target.description.value = "";
          event.target.image.value = null;
        });
    },
    modifyPost(pid, post) {
      const token = localStorage.getItem("groupomania_token");
      axios
        .put("http://localhost:3000/api/posts/" + pid, post, {
          headers: { Authorization: "Bearer " + token },
        })
        .then((d) => {
          this.posts = this.posts.map((p) => {
            if (pid === p.id) {
              p["image"] = d.data.filename;
            }

            return p;
          });
        });
    },
    deletePost(id) {
      if (confirm("Voulez-vous vraiment supprimer ce post ?")) {
        const token = localStorage.getItem("groupomania_token");
        axios
          .delete("http://localhost:3000/api/posts/" + id, {
            headers: { Authorization: "Bearer " + token },
          })
          .then(() => {
            this.posts = this.posts.filter((post) => post.id !== id);
          });
      }
    },
    logout(event) {
      event.preventDefault();
      localStorage.removeItem("groupomania_token");
      this.$router.push("/login");
    },
  },
};
</script>

<style scoped>
body,
html,
#app {
  padding: 0;
  margin: 0;
  width: 100%;
  height: 100%;
  font-family: "Montserrat", sans-serif;
  color: rgb(228, 230, 235);
}

input[type="file"] {
  display: none;
}

body {
  background-color: #18191a;
}

.PostTemplateBody {
  padding-top: 30px;
}

main {
  position: relative;
  width: 50%;
  margin: auto;
  background-color: #18191a;
}

h2 {
  color: rgb(228, 230, 235);
}

.RightLine {
  background: url("../assets/images/ligne-rouge-saumon.png") repeat-y right;
}

.post {
  color: rgb(228, 230, 235);
  display: flex;
  flex-direction: column;
  width: 60%;
  margin: auto;
  margin-bottom: 40px;
  padding: 0px 30px 5px 30px;
  background-color: #242526;
  border-radius: 10px;
}

.LeftLine {
  width: 100%;
  justify-content: center;
  margin: auto;
  background: url("../assets/images/ligne-rouge-saumon.png") repeat-y left;
}

.row {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

textarea {
  background: #18191a;
  color: rgb(228, 230, 235);
  border: 1px solid #18191a;
  font-family: "Montserrat", sans-serif;
  outline: none;
  overflow: auto;
  border-radius: 10px;
}

#titleArea {
  background: #18191a;
  color: rgb(228, 230, 235);
  border: 1px solid #18191a;
  font-family: "Montserrat", sans-serif;
  outline: none;
  border-radius: 10px;
  margin-bottom: 20px;
  width: 75%;
  height: 40px;
}

#AddIMG {
  margin-bottom: 30px;
  background: #18191a;
  color: rgb(228, 230, 235);
  outline: none;
  border: 2px #27292b solid;
  border-radius: 10px;
  width: 40%;
  padding: 10px;
  margin: 20px 0px 20px 0px;
  font-size: 1em;
  text-align: center;
  font-weight: bold;
}

#AddIMG:hover {
  background: #27292b;
}

#publier {
  margin-bottom: 10px;
  background: #18191a;
  color: rgb(228, 230, 235);
  outline: none;
  border: 2px #d1515a solid;
  border-radius: 10px;
  width: 50%;
  padding: 10px;
  font-size: 1.2em;
  font-weight: bold;
}

#publier:hover {
  transition-duration: 0.4s;
  background-color: #d1515a;
  color: white;
}

#publier:active {
  transition-duration: 0s;
  background-color: #d1515a8f;
  color: white;
}

#PostArea::placeholder,
input::placeholder {
  color: rgb(228, 230, 235);
}

/*aside Left*/

aside ul {
  display: flex;
  flex-direction: column;
  float: right;
}

aside ul,
header li {
  list-style: none;
  display: flex;
  padding-right: 50px;
  font-size: 1.25em;
}

aside a:visited {
  color: rgb(228, 230, 235);
}

aside a:hover {
  color: #d1515a;
}

a {
  text-decoration: none;
  color: rgb(228, 230, 235);
}

.AsideMenu {
  background-color: #18191a;
  position: absolute;
  right: 75%;
  width: 25%;
  height: 100%;
  font-size: 1.3em;
  position: fixed;
  top: 30px;
}

.leftMenu {
  display: flex;
  flex-direction: row;
  margin-bottom: 40px;
}

.leftMenu i {
  margin-right: 10px;
}

.AsideMenu img {
  width: 60%;
  margin-bottom: 40px;
  margin-left: 20%;
}

/*AsideRight*/
.AsideFriend {
  background-color: #18191a;
  width: 25%;
  height: 100%;
  position: fixed;
  top: 0px;
  left: 75%;
}

.FriendAsideContent ul {
  float: left;
}

.RelationTitle {
  margin-left: 50px;
  margin-top: 30px;
}

/*responsive !!!*/

@media screen and (max-width: 1430px) {
  .AsideFriend {
    display: none;
  }

  main {
    width: 75%;
    float: right;
  }

  .RightLine {
    background: none;
  }
}

@media screen and (max-width: 1150px) {
  .AsideMenu {
    position: fixed;
    height: 100px;
    right: 0;
    padding-top: 0px;
    margin-top: -30px;
    z-index: 4;
    display: flex;
    flex-direction: row;
    width: 100%;
    background: #242526;
    box-shadow: 0px 0px 10px #242526;
  }

  .AsideMenu img {
    display: none;
  }

   .AsideMenu ul a li {
    display: none;
  }

  .AsideMenu i {
    margin-top: 40px;
    padding-right: 20px;
    padding-left: 20px;

  }


  .AsideMenu ul {
    margin: auto;
    display: flex;
    flex-direction: row;
    height: 100px;
    line-height: 400%;
  }

  .AsideMenu li {
    margin-right: 20px;
    flex-direction: row;
  }

  main {
    width: 100%;
    position: relative;
    margin: auto;
    margin-top: 100px;
  }

  .LeftLine {
    background: none;
  }
}

@media screen and (max-width: 700px) {
  .AsideMenu {
    font-size: 1em;
  }

  .AsideMenu i {
    margin-top: 40px;
    padding-right: 10px;
    padding-left: 10px;
  }

  .post {
    width: 90%;
  }
  
}
</style>
