<template>
  <div>
    <app-header></app-header>
    <h1>Edit Booking Timings</h1>

        <div class="float-container">
            <div class = "float-left">
                <v-date-picker v-model="date" ></v-date-picker>
            </div>
            <div class = "float-right">
                <div class="timeslot">

                <div v-if="dateSelected && availableTime">
                    <button v-on:click="showModal">Add Timeslots</button>
                    <button v-on:click="selectAll()">Select All</button>
                    <h3>Time: Pax</h3>
                    <label class="timing-listing" v-for="(slot, index) in timeslots" :key="index" >
                        {{ slot.time }} : {{slot.pax}}
                        <input type="checkbox" v-bind:value="slot" v-model="selected" />
                        <span class="checkmark"></span>
                    </label>
                    <input v-model.number="pax" placeholder="Input pax limit">
                    <button  v-on:click="update()">Update Pax</button>
                    <button  v-on:click="del()"> Delete Timeslot</button>
                </div>
                <div v-else-if="dateSelected">
                    <button v-on:click="showModal">Add Timeslots</button>
                    <h3>No timeslots available</h3>
                </div>
                <div v-else> Choose a date to start </div>

                <div>
                    <modal 
                        v-show="isModalVisible" 
                        @close="closeModal" 
                        @apply="closeModal(); add()"
                    >
                        <template v-slot:header>
                            New timeslots:
                        </template>
                        <template v-slot:body>
                            <p><input v-model="start" placeholder="Start Time"></p>
                            <p><input v-model="end" placeholder="End Time"></p>
                            <p><input v-model.number="pax" placeholder="Pax limit"></p>
                            <input type="checkbox" v-bind:value=true v-model="applyMonth"/>Apply for the Month
                        </template>
                    </modal>
                </div>



            </div>            
            </div>
        </div>
    </div>
</template>

<script>
    import Header from "../UI/Header.vue";
    import firebase from 'firebase';
    import database from "../../firebase.js";
    import DatePicker from "v-calendar/lib/components/date-picker.umd";
    import modal from "../Modal.vue";



    export default {
        data() {
            return {
                bizID: '5iBl58sV6uv7riUzCQzn',//change according to user
                timeslots: [],
                selected: new Array(),//timeslot selected
                paxDisplayed: 0,
                date: "",
                fulldate: "",
                pax: null,
                start: "",
                end: "",
                dateSelected: false,
                availableTime: false,
                isModalVisible: false,
                applyMonth: false
            }
        },

        components: {
            "app-header": Header,
            "v-date-picker": DatePicker,
            modal: modal,


        },

        watch: {
            date: async function() {
                this.dateSelected = true;
                await this.fetch();
            }
        },

        methods: {
            selectAll: function() {
                this.selected = this.timeslots;
            },

            showModal: function() {
                this.isModalVisible = true;
            },

            closeModal: function() {
                this.isModalVisible = false;
            },

            fetch: function() {//fetch the timeslots info
                if (this.date == undefined) {
                    return;
                }

                var date = this.date.getDate();
                if (date < 10) {
                    date = "0" + date;
                }
                var month = this.date.getMonth() + 1;
                if (month < 10) {
                    month = "0" + month;
                }
                var year = this.date.getYear() % 100;

                this.fulldate = date + month + year;
                console.log(this.fulldate);

                //this.bizID = this.$route.params.id;
                console.log("id: "+this.bizID);
                //change after fetch
                
                database
                    .collection('listings')
                    .doc(this.bizID)
                    .collection('timeslots')
                    .doc(this.fulldate)
                    .get()
                    .then((snapshot) => {

                    if (snapshot.exists) {
                        const data = snapshot.data();

                        this.timeslots = [];

                        Object.keys(data).map((time) => {
                            this.timeslots.push({"time": time, "pax": data[time]});//[{time: xx, pax: xx}, {}]
                            this.availableTime = true;
                        });

                        this.timeslots.sort((a, b) => { 
                            return parseInt(a["time"].substring(0, 4)) - parseInt(b["time"].substring(0, 4))
                        });//fix order

                    } else {
                        this.availableTime = false;
                        this.timeslots = [];

                }});
                
               


            },

            add: async function() {//release new timeslots
                if (this.start == "") {
                    alert("Please input a start time.")
                } else if (this.end == "") {
                    alert("Please input an end time")
                } else if (this.pax == null) {
                    alert("Please input a pax limit.")
                }

                var startHour = Math.floor(parseInt(this.start)/100);
                var startMin = (parseInt(this.start) - startHour * 100).toString();
                if (startMin == "0") {
                    startMin = "00";
                }
                var endHour = Math.floor(parseInt(this.end)/100);
                var endMin = (parseInt(this.end) - endHour * 100).toString();
                if (endMin == "0") {
                    endMin = "00";
                }

                if (startHour >= endHour) {
                    alert("Start Time cannot exceed End Time.")
                }

                        var newSlots = [];
                        while (startHour < endHour) {
                            var endSlot = startHour + 1;
                            var startStr = startHour.toString() + startMin;
                            if (startHour < 10) {
                                startStr = "0" + startStr;
                            }
                            var endStr = endSlot.toString() + endMin
                            if (endSlot < 10) {
                                endStr = "0" + endStr;
                            }//create string representation of the time slots
                            newSlots.push({[startStr + " - " + endStr] : this.pax});//collect the new timeslots
                            startHour = endSlot;
                            console.log(newSlots);
                        }//end of while loop

                if (this.applyMonth) {//add timeslots for the whole month
                    var month = this.date.getMonth() + 1;
                    var year = this.date.getYear() % 100;  
                    var numDay = 30;
                    var day = 1;
                    var longer = [1, 3, 5, 7, 8, 10, 12]
                    if (longer.includes(month)) {
                        numDay = 31;
                    } else if (month == 2) {
                        if (this.date.getYear() % 4 == 0) {
                            numDay = 29;
                        } else {
                            numDay = 28;
                        }
                    }//calculate the number of days of the month
                    if (month < 10) {
                        month = "0" + month;
                    } 

                    while (day <= numDay) {//loop through the days of the month
                        var date = day;//change after each loop
                        if (date < 10) {
                            date = "0" + day;
                        }
                        var fulldate = date + month + year;//date of that specific day
                        console.log(fulldate);
                        
                        var docRef = database.collection('listings')
                            .doc(this.bizID)
                            .collection('timeslots')
                            .doc(fulldate);//hard code works

                            await docRef
                            .get()
                            .then((snapshot) => {
                                if (snapshot.exists) {//alr have existing timeslots
                                    for (var j=0; j< newSlots.length; j++) {
                                        console.log("update" + newSlots[j]);
                                        docRef.update(newSlots[j]);
                                    }
                                } else {//empty doc
                                    console.log("set" + newSlots[0]);
                                    docRef.set(newSlots[0]);
                                    for (var k=1; k< newSlots.length; k++) {
                                        docRef.update(newSlots[k]);
                                    }
                                    
                                }
                            });
                        day ++;
                    }

                    this.applyMonth = false;
                    location.reload();
                } else {//add timeslot for the day
                        var docRefDate = database.collection('listings')
                            .doc(this.bizID)
                            .collection('timeslots')
                            .doc(this.fulldate);

                            await docRefDate
                            .get()
                            .then((snapshot) => {
                                if (snapshot.exists) {//alr have existing timeslots
                                    for (var j=0; j< newSlots.length; j++) {
                                        docRefDate.update( newSlots[j]);
                                    }
                                } else {//empty doc
                                    docRefDate.set(newSlots[0]);
                                    for (var k=1; k< newSlots.length; k++) {
                                        docRefDate.update(newSlots[k]);
                                    }
                                    
                                }
                            });
                        location.reload();
                }
            },

            update: function() {//update pax limit
                if (this.selected.length == 0) {
                    alert("Please select at least 1 timeslot");
                    return;
                }

                var docRef = database.collection('listings')
                    .doc(this.bizID)
                    .collection('timeslots')
                    .doc(this.fulldate);

                for (var i = 0; i < this.selected.length; i++) {
                    var time = this.selected[i]["time"];
                        console.log("update: "+time)
                        var updatedValue = this.pax;
                        docRef
                        .update({
                            [time]: updatedValue,//update the firebase
                        })
                        .then(() => location.reload());                    
                }
            },

            del: function() {//delete timeslots

                if (this.selected.length == 0) {
                    alert("Please select at least 1 timeslot");
                    return;
                }

                const FieldValue = firebase.firestore.FieldValue;
                var docRef = database.collection('listings')
                    .doc(this.bizID)
                    .collection('timeslots')
                    .doc(this.fulldate);

                for (var i = 0; i < this.selected.length; i++) {
                    var time = this.selected[i]["time"]
                        console.log("delete: "+time)
                        docRef.update({
                            [time]: FieldValue.delete()//delete field in firebase
                        })
                        .then(() => location.reload());
                }                
            },
    }
}
</script>


<style scoped>
/* Customize the label (the container) */
.timing-listing {
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
  width: 50%;
}

/* Hide the browser's default checkbox */
.timing-listing input {
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
.timing-listing:hover input ~ .checkmark {
  background-color: #ccc;
}

/* When the checkbox is checked, add a blue background */
.timing-listing input:checked ~ .checkmark {
  background-color: #2196f3;
}

/* Create the checkmark/indicator (hidden when not checked) */
.checkmark:after {
  content: "";
  position: absolute;
  display: none;
}

/* Show the checkmark when checked */
.timing-listing input:checked ~ .checkmark:after {
  display: block;
}

/* Style the checkmark/indicator */
.timing-listing .checkmark:after {
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
  margin: auto;
}

.float-left,
.float-right {
  flex: 1;
  height: 100%;
  width: 50%;
  overflow: auto;
}

.float-left {
    width: 50%;
    float: left;
    padding: 20px;
    border: 2px ;
}

.float-right {
    width: 50%;
    float: right;
    padding: 20px;
    border: 2px ;
}

.timeslot {
  width: 100%;
  margin: auto;
  font-size: 18px;
}

</style>
