<template>
  <!--Mur-->
  <div id="mur">
    <div class="bienvenu" v-for="usr in user" :key="usr.userId">
      <h2>
        Bienvenue
        <span>{{usr.username}}</span>!
      </h2>
    </div>
    <div class="getMessag">
      <h3 id="mess">Liste des Messages</h3>
      <div id="messdiv" class="msg" v-for="mess in msg" :key="mess.idMESSAGES">
        <p class="nameus">{{mess.username}}</p>
        <p class="text">{{mess.message}}</p>
        <img :src="mess.image" alt="image">
        <p class="datt">{{moment(mess.created_at).fromNow()}}</p>
        <button
          @click="updatemess(mess.idMESSAGES)"
          v-if="data.username == mess.username || data.status =='admin'"
          type="button"
          class="btn btn-success btn-sm mod"
        >Modifier</button>
        <button
          @click="deletemess(mess.idMESSAGES)"
          v-if="data.username == mess.username || data.status =='admin'"
          type="button"
          class="btn btn-danger btn-sm sup"
        >
          <font-awesome-icon icon="trash" />
        </button>
        <button
          @click="response(mess.idMESSAGES)"
          class="btn btn-info btn-circle text-uppercase bt"
          id="reply"
        >
          <span class="glyphicon glyphicon-share-alt"></span>Repondre
        </button>
        <button
          @click="view(mess.idMESSAGES)"
          class="btn btn-warning btn-circle text-uppercase bt"
          id="voir"
          data-toggle="collapse"
          href="#/viewresponse"
        >
          <span class="glyphicon glyphicon-comment"></span>Voir les réponses
        </button>
      </div>
    </div>
    <h4>Écrire votre nouveau message</h4>
    <form id="formtog" method="POST" class="from-group" @submit.prevent="sendMessage" enctype="multipart/form-data">
      <div class="form-group">
        <label for="message">Message</label>
          <textarea
            class="form-control"
            name="message"
            id="message"
            cols="50"
            rows="5"
            v-model="message">
            </textarea>
      </div>
      <div class="button">
        <input type="file" @change="onSelectedFile($event)" name="image" id="image" accept="image/png, image/jpg, image/jpeg, image/gif">
        <button type="submit" id="envoi" class="btn btn-dark">Envoyer</button>
      </div>
    </form>
  </div>
</template>

<script>

let moment = require("moment");
moment.locale("fr"); //Appel du module moment.js pour affichage à quel moment le message a été écrit

export default {
  name: "mur",
  data() {
    return {
      data: JSON.parse(this.$localStorage.get("user")),
      message: "",
      msg: "",
      date: "",
      moment: moment,
      imess: "",
      update: "",
      user: "",
      image: "",
    };
  },
  mounted() {
    //Appel à API pour affichage de tous les messages
    let token= this.data.token;
    this.$axios
      .get("/getmessages", 
        {
          headers: 
          {
            "Content-type": "application/json",
            Authorization: `Bearer ${token}` //Renvoi du token par API en cas d'authentification
          }
        })
      .then(response => {
        // console.log(response.data);
        this.msg = response.data;
      })
      .catch(error => console.log(error));

    let data = JSON.parse(this.$localStorage.get("user"));
    //Appel à l'Api pour l'affichage des informations utilisateurs
    this.$axios
      .get(`/getoneuser/${data.userId}`, 
      {
        headers: 
        {
          "Content-type": "application/json",
          Authorization: `Bearer ${token}` //Renvoi du token par API en cas d'authentification
        }
      })
      .then(response => {
        // console.log(response.data);
        this.user = response.data;
      })
      .catch(error => console.log(error));
  },
  methods: {
    sendMessage: function() {
      //Fonction qui permet d'envoyer un message
      let token = this.data.token;
      let idUSERS = this.data.userId;
      let username = this.data.username;
      if (this.message.length ===0) {
        alert(
          "Vous n'avez rien écrit: vous ne pouvez pas envoyer un message vide !"
        );
      } else {
        let formData = new FormData();
        formData.append("idUSERS", idUSERS)
        formData.append("username", username)
        formData.append("message", this.message)
        formData.append("image", this.image)
        
        this.$axios
          .post(
            "/postmessage", formData,
            {
              headers: 
              {
                "Content-type": "multipart/form-data",
                Authorization: `Bearer ${token}`
              }
            }
          )
          .then(() => {
            console.log("message envoyé");
            this.message === "";
            alert("votre message a bien été envoyé !");
            location.reload(true);
          })
          .catch(() => {
            console.log("le message n'a pas été envoyé");
          });
      }
    },

    onSelectedFile(event) {
      console.log(event.target.files[0]);
      this.image = event.target.files[0];
    },

    deco: function() {
      //Déconnexion
      if (window.confirm("Voulez-vous vraiment vous déconnecter ?")) 
      {
        this.$session.remove("user");
        window.location.href = "http://localhost:8080//#/";
      }
    },

    deletemess: function(delid) {
      //Fonction permettant à utilisateur de supprimer un message (avec image)
      let token = this.data.token;
      let image = this.msg[0].image;
      let idmess = delid;

      if (
        confirm("Etes-vous sûr de vouloir supprimer ce message ?") &&
        confirm("Attention, cela effacera définitivement le message")
      ) 
      {
        this.$axios
          .post(
            "/deletemessage",
            {
              id: idmess,
              image: image
            },
            {
              headers: {
                "Content-type": "application/json",
                Authorization: `Bearer ${token}`
              }
            }
          )
        .then(() => {
          console.log("message supprimé");
          alert("votre message a bien été supprimé");
          location.reload(true);
        })
        .catch(() => {
          console.log("le message n'a pas été supprimé !");
        });
      }
    },

    updatemess: function(idmess) {
      //Fonction permettant à utilisateur de modifier message
      let imess = idmess;

      window.location.href = `http://localhost:8080//#/res?id=${imess}`;
      location.reload(true);
    },

    response: function(idmess) {
      //Fonction permettant à utilisateur de répondre à message posté
      let irep = idmess;

      window.location.href = `http://localhost:8080//#/reponses?id=${irep}`;
      location.reload(true);
    },

    view: function(idmess) {
      let iview = idmess;

      window.location.href = `http://localhost:8080//#/viewresp?id=${iview}`;
      location.reload(true);
    }
  }
};
</script>

<style lang="scss" scoped>
@media screen and (min-width: 300px) and (max-width: 992px) {
  .btn-group {
    position: relative;
    margin-left: auto;
    margin-right: auto;
  }
}

h2 {
  position: relative;
  top: 30px;
  display: inline;
}

span {
  text-transform: uppercase;
}

img {
  height: 80px;
}

.form-control {
  width: 50%;
  margin: 0 auto;
  margin-top: 60px;
}
.form-group {
  margin-top: 100px;
  height: 20px;
}

.text,
.datt {
  color: #fff;
}

.btn {
  color:black;
}

.btn-dark {
  color:white;
}

.bienvuenu {
  position: relative;
}

.msg {
  border: 1px solid lightgray;
  width: 50%;
  line-height: 15px;
  height: 180px;
  position: relative;
  top: 40px;
  margin-right: auto;
  background-color: #392546;
  border-radius: 5px;
  margin-left: auto;
  margin-top: 20px;
  margin-bottom: 100px;
  @media screen and (min-width: 320px) and (max-width: 830px) {
    width: 75%;
  }
}

#mess {
  position: relative;
  top: 50px;
}

.nameus {
  font-size: 0.8em;
  text-transform: uppercase;
  font-weight: bolder;
  margin-top: 10px;
  color: rgb(240, 210, 136);
}

.datt {
  font-size: 0.7em;
  position: relative;
  text-transform: lowercase;
}

.text {
  position: relative;
  font-size: 0.9em;
  font-weight: 700;
}

h4 {
  position: relative;
  top: 80px;
}

h5 {
  margin-top: 40px;
  position: relative;
  top: 70px;
  margin-bottom: 50px;
}

#message {
  position: relative;
  bottom: 100px;
}

#envoi {
  position: relative;
  top: 30px;
  height: 50px;
  margin-top: 60px;
}

#deco {
  text-decoration: none;
}

.btn-circle {
  font-weight: bold;
  font-size: 12px;
  border-radius: 10px;
  height: 30%;
  margin-right: 10px;
  position: relative;
  margin-bottom: 20px;
  top: 20px;
}
.btn-circle span {
  padding-right: 0px;
}
.embed-responsive {
  margin-bottom: 10px;
}

h5 {
  margin-top: 100px;
}

.sup {
  position: absolute;
  left: 570px;
  bottom: 10px;
  @media screen and (min-width: 320px) and (max-width: 500px) {
    position: relative;
    left: 165px;
    bottom: 40px;
  }
}

.mod {
  position: relative;
  bottom: 50px;
  right: 120px;
  @media screen and (min-width: 320px) and (max-width: 500px) {
    position: relative;
    right: 70px;
    bottom: 40px;
  }
}

.button {
  display: flex;
  justify-content: center;
  margin-bottom: 150px;
}

#image {
  margin-top: 90px;
  margin-left: 10px;
}

@media screen and (min-width: 320px) and (max-width: 500px) {
  #reply {
    position: relative;
    top: 40px;
    right: 50px;
    margin-bottom: 10px;
  }

  #voir {
    position: relative;
    margin-top: 15px;
    margin-bottom: 10px;
  }
}
</style>