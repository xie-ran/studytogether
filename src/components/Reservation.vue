<template>
  <div>
    <app-header></app-header>

    <div>
      <p class="title">Make A Reservation</p>
    </div>
    <div class="float-container">
      <form class="form">
        <div class="float-left">
          <br />
          <label for="">PAX: </label>
          <select v-model="pax">
            <option v-for="num in 5" v-bind:key="num">{{ num }}</option>
          </select>
          <br />
          <br />
          <div v-if="paxSelected">
            <label for="date">Date </label>
            <br />
            <v-date-picker
              v-model="date"
              :min-date="new Date()"
            ></v-date-picker>
            <br />
            <br />
            <button class="submit" type="submit" v-on:click="submit">Submit</button>
            <br />
            <br />
          </div>
        </div>

        <div class="float-right">
          <div class="timeslot">
            <br />
            <label for="time">Timeslot: </label>
            <br />
            <br />
            <div v-if="dateSelected && availableTime">
            <label class="container" v-for="item in timeslot" :key="item"
              >{{ item }}
              <input type="checkbox" v-bind:value="item" v-model="selected" />
              <span class="checkmark"></span>
            </label>
            <br/>
            </div>
          </div>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
import Header from "./UI/Header.vue";
import database from "../firebase.js";
import firebase from "firebase";
import DatePicker from "v-calendar/lib/components/date-picker.umd";

export default {
  data() {
    return {
      pax: new Number(),
      date: "",
      timeslot: new Array(),
      selected: new Array(),
      paxSelected: false,
      dateSelected: false,
      timeSelected: false,
      availableTime: false,
    };
  },
  components: {
    "app-header": Header,
    "v-date-picker": DatePicker,
  },

  methods: {
    fetchTime: function () {
      if (this.date == undefined) {
        this.timeslot = [];
        return;
      }

      var date = this.date.getDate();
      if (date < 10) {
        date = "0" + date;
        console.log(date);
      }
      var month = this.date.getMonth() + 1;
      if (month < 10) {
        month = "0" + month;
      }
      var year = this.date.getYear() % 100;

      var fulldate = date + month + year;
      console.log(fulldate);

      database
        .collection("listings")
        .doc(this.$route.params.id)
        .collection("timeslots")
        .doc(fulldate)
        .get()
        .then((snapshot) => {
          console.log(snapshot.data());

          if (snapshot.exists) {
            const data = snapshot.data();

            this.timeslot = [];

            Object.keys(data).map((time) => {
              console.log(time);

              var now = new Date();

              if (data[time] >= this.pax) {
                console.log("test1");

                var startTime = Number(time.substring(0, 2));
                if (
                  new Date(
                    this.date.getFullYear(),
                    this.date.getMonth(),
                    this.date.getDate(),
                    startTime,
                    0,
                    0,
                    0
                  ) > now
                ) {
                  console.log("test2");
                  this.availableTime = true;
                  this.timeslot.push(time);
                }
              }
            });

            this.timeslot = this.timeslot.filter((element) => {
              return element !== undefined;
            });

            console.log(this.timeslot);

            this.timeslot.sort();
          } else {
            const arrayResult = [];

            this.availableTime = false;

            arrayResult.sort();

            this.timeslot = arrayResult;
          }
        });
    },

    submit: function (e) {
      e.preventDefault();

      if (this.selected.length == 0) {
        alert("Please select at least 1 timeslot");
        return;
      }

      var locationID = this.$route.params.id;

      var user = firebase.auth().currentUser;

      var data = {
        location: locationID,
        userid: user.uid,
        name: user.displayName,
        pax: this.pax,
        date: this.date,
        time: this.selected,
      };

      var date = this.date.getDate();
      if (date < 10) {
        date = "0" + date;
      }
      var month = this.date.getMonth() + 1;
      if (month < 10) {
        month = "0" + month;
      }
      var year = this.date.getYear() % 100;

      var fulldate = date + month + year;

      database
        .collection("listings")
        .doc(this.$route.params.id)
        .collection("timeslots")
        .doc(fulldate)
        .get()
        .then((snapshot) => {
          const data = snapshot.data();

          for (var i = 0; i < this.selected.length; i++) {
            var time = this.selected[i];
            var updatedValue = data[time] - this.pax;

            database
              .collection("listings")
              .doc(locationID)
              .collection("timeslots")
              .doc(fulldate)
              .update({
                [time]: updatedValue,
              });
          }
        });

      database.collection("bookings").add(data);

      alert("Booking successful!");
      this.$router.push({ path: "/listings" });
    },
  },

  watch: {
    date: async function () {
      this.dateSelected = true;
      await this.fetchTime();
    },

    pax: async function () {
      this.paxSelected = true;
      if (this.dateSelected) {
        await this.fetchTime();
      }
    },
  },
};
</script>

<style scoped>
/* Customize the label (the container) */
.container {
  display: block;
  margin: auto;
  position: relative;
  padding-left: 35px;
  margin-bottom: 12px;
  cursor: pointer;
  font-size: 22px;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  width: 10%;
}

/* Hide the browser's default checkbox */
.container input {
  position: absolute;
  opacity: 0;
  cursor: pointer;
  height: 0;
  width: 0;
}

/* Create a custom checkbox */
.checkmark {
  position: absolute;
  top: 0;
  left: 0;
  height: 25px;
  width: 25px;
  background-color: #eee;
}

/* On mouse-over, add a grey background color */
.container:hover input ~ .checkmark {
  background-color: #ccc;
}

/* When the checkbox is checked, add a blue background */
.container input:checked ~ .checkmark {
  background-color: #2196f3;
}

/* Create the checkmark/indicator (hidden when not checked) */
.checkmark:after {
  content: "";
  position: absolute;
  display: none;
}

/* Show the checkmark when checked */
.container input:checked ~ .checkmark:after {
  display: block;
}

/* Style the checkmark/indicator */
.container .checkmark:after {
  left: 9px;
  top: 5px;
  width: 5px;
  height: 10px;
  border: solid white;
  border-width: 0 3px 3px 0;
  -webkit-transform: rotate(45deg);
  -ms-transform: rotate(45deg);
  transform: rotate(45deg);
}

select {
  height: 25px;
  width: 40%;
  font-size: 15px;
}

.float-container {
  display: flex;
  align-items: center;
  width: 60%;
  margin: auto;
}

.float-left,
.float-right {
  flex: 1;
  height: 100%;
  overflow: auto;
}

.float-left {
  float: left;
  align-items: left;
  background-color: #FFFFFF;
  border-radius: 20px;
  margin-left: 5px;
  box-shadow: 0px 0px 10px 10px lightgrey;
}

.float-right {
    margin-left: 30px;
  float: right;
  align-items: left;
  background-color: #FFFFFF;
  border-radius: 20px;
  box-shadow: 0px 0px 10px 10px lightgrey;
}

.timeslot {
  width: 100%;
  margin: auto;
  font-size: 18px;
}

.container {
  width: 30%;
  text-align: left;
  margin-left: auto;
}

form {
  display: flex;
  margin: auto;
  width: 100%;
}

.submit{
    display:inline-block;
    padding:0.5em 3em;
    border:0.16em solid darkgray;
    margin:0 0.3em 0.3em 0;
    box-sizing: border-box;
    text-decoration:none;
    text-transform:uppercase;
    font-family:'Roboto',sans-serif;
    font-weight:400;
    color:darkgray;
    text-align:center;
    transition: all 0.2s;
    cursor: pointer;
}

.submit:hover{
    color:#DDDDDD;
    border-color:#DDDDDD;
}
.submit:active{
    color:#BBBBBB;
    border-color:#BBBBBB;
}
</style>