<template>
  <div class="favourites list">
    <app-header></app-header>
    <h1>Favourites</h1>
    <ul class="favourites-list">
      <li v-for="(listing, index) in list" :key="index">
        <!-- picture of cafe -->
        <img id="main-pic" v-bind:src="listing.photoURL1" />
        <div id="name">
          <b> {{ listing.name }} </b>
        </div>
        <!-- Button to cancel bookmark -->
        <button
          class="button"
          v-bind:id="listing.id"
          v-on:click="remove($event)"
        >
          Remove
        </button>
      </li>
    </ul>
  </div>
</template>

<script>
import firebase from "firebase";
import database from "../firebase.js";
import Header from "./UI/Header.vue";

export default {
  data() {
    return {
      list: [],
      idList: [],
    };
  },

  components: {
    "app-header": Header,
  },

  created() {
    this.fetchItems();
  },

  methods: {
    getListing: async function () {
      var arr = [];
      await database
        .collection("listings")
        .get()
        .then((querySnapShot) => {
          let item = {};
          querySnapShot.docs.forEach((doc) => {
            item = { ...doc.data(), ["id"]: doc.id };
            // console.log(item)
            arr.push(item);
          });
        });
      return arr;
    },

    getUser: async function () {
      var arr = [];
      await database
        .collection("users")
        .get()
        .then((querySnapShot) => {
          let item = {};
          querySnapShot.docs.forEach((doc) => {
            item = { ...doc.data(), ["id"]: doc.id };
            
            // console.log(item)
            arr.push(item);
          });
        });
      return arr;
    },

    fetchItems: async function () {
      var listArr = await this.getListing();

      var user = firebase.auth().currentUser;

        await database.collection('users').doc(user.uid).get().then(snapshot => {
            const data = snapshot.data();
            this.idList = data.favourites;

        })

      for (var j = 0; j < listArr.length; j++) {

        if (this.idList.includes(listArr[j].id)) {
            this.list.push(listArr[j]);
        }
      }

      console.log(this.list);

 
    },

    remove: function (event) {
      let doc_id = event.target.getAttribute("id");
      console.log(doc_id);
      var newList = [];
      for (var k = 0; k < this.idList.length; k++) {
          if (doc_id != this.idList[k]) {
              newList.push(this.idList[k]);
          }
      }
      console.log(newList);

      this.idList = newList;

      let user = firebase.auth().currentUser;
      database
        .collection("users")
        .doc(user.uid)
        .update({ favourites: this.idList })
        .then(() => {location.reload()});
    },
  },
};
</script>

<style scoped>
ul {
  display: flex;
  flex-wrap: wrap;
  list-style-type: none;
  padding: 30%;
  padding-top: 3%;
  margin: auto;
}

li {
  display: flex;
  flex-wrap: wrap;
  width: 1000px;
  position: relative;
  /*  flex-basis: 300px; */
  text-align: center;
  padding: 10px;
  border: 3px solid #ed7a78;
  margin: 10px;
  margin-top: 5px;
  border-radius: 25px;
  font-family: "Ubuntu", sans-serif;
  margin: 0 0 10 0;
}

.button {
  position: relative;
  left: 90px;
  height: 50px;
  width: 90px;
}

#main-pic {
  position: relative;
  /* left: 1px; */
  border-radius: 15px;
  width: 200px;
  height: 150px;
}

#name {
  position: relative;
  left: 30px;
  top: 15px;
  font-family: Verdana, Geneva, Tahoma, sans-serif;
  font-size: 30px;
  font-weight: 1000;
}
</style>
