<template>
<v-app class="bg-lightteal">
  <v-container cols="12" text-center align="center" justify="center" class="pt-0">
    <v-row>
      <!-- header -->
      <v-col cols="8" offset="2">
          <h1>Edit an existing volunteer</h1>
      </v-col>

      <!-- return-to-volunteers-list button -->
      <v-col cols="2">
        <v-btn 
          class="mx-1 my-1" 
          to="/volunteers"
          fab right dark x-small 
          color="teal"
        >
          <v-icon dark>mdi-arrow-left</v-icon>
        </v-btn>
        Back
      </v-col>

    </v-row>

    <v-row>

        <!-- enter first name -->
        <v-col cols="3">
          <v-card class="inputCard">
            <v-card-text class="py-0 px-1 inputCard">
              <v-form>
                <v-text-field 
                  label="First Name" 
                  class="py-0" 
                  v-model="firstName" 
                  autofocus
                ></v-text-field>
              </v-form>
            </v-card-text>
          </v-card>
        </v-col>

        <!-- enter last name -->
        <v-col cols="3">
          <v-card class="inputCard">
            <v-card-text class="py-0 px-1 inputCard">
              <v-form>
                <v-text-field 
                  label="Last Name" 
                  class="py-0" 
                  v-model="lastName" 
                ></v-text-field>
              </v-form>
            </v-card-text>
          </v-card>
        </v-col>

        <!-- enter email -->
        <v-col cols="3">
          <v-card class="inputCard">
            <v-card-text class="py-0 px-1 inputCard">
              <v-form>
                <v-text-field 
                  label="Email" 
                  class="py-0" 
                  v-model="email" 
                >
                </v-text-field>
              </v-form>
            </v-card-text>
          </v-card>
        </v-col>

        <!-- choose image file -->
        <v-col cols="3">
          <v-card class="inputCard">
            <v-card-text class="py-0 px-1 inputCard">
              <v-form class="match-height">
                <v-select
                  label="Choose avatar image." 
                  v-model="image"
                  prepend-icon="mdi-camera"
                  :items="avatars"
                  dense
                ></v-select>
              </v-form>
            </v-card-text>
          </v-card>
        </v-col>
    </v-row>

    <v-row>
      <!-- choose roles & preferred times -->
      <v-col cols="6" class="white">
        <p class="text-left">Choose role:</p>
        <!-- to choose more than one in a future release
                    <checkbox class="ml-2 my-0 list-height" -->
        <v-radio-group class="ml-2"
          v-model="rolesChosen"
        ><v-radio
          class="my-0 list-height"
          v-for="(role, roleIndex) in roles"
          v-bind:key="roleIndex"
          :value="`${role}`"
          :label="`${role}`"
          color="teal"
        ></v-radio>
        </v-radio-group>
      </v-col>

      <v-col cols="6" class="white">
        <p class="text-left">Choose a preferred time slot:</p>
        <v-radio-group class="ml-2"
          v-model="preferredTime"
        ><v-radio
            class = "my-0 list-height"
            v-for="(timeSlot, idx) in timeSlots"
            v-bind:key="idx"
            :label="formatTime(timeSlot)"
            :value="`${idx}`"
            color="teal"
          ></v-radio>
        </v-radio-group>

      </v-col>

    </v-row>

    <v-row>
        <!-- choose availability -->
      <v-col cols="11">
        <v-card class="inputCard">
          <v-card-text class="py-0 px-1 inputCard">


            <v-dialog
              ref="dialog"
              v-model="showNotAvailablePicker"
              persistent
              width="290px"
            >

              <template v-slot:activator="{ on }">
                <v-text-field
                  v-model="badDates"
                  label="Choose dates volunteer is NOT available."
                  readonly
                  v-on="on"
                ></v-text-field>
              </template>

              <v-date-picker
                v-if="showNotAvailablePicker"
                v-model="badDates"
                full-width
                color="teal"
                no-title
                multiple
                :min="today()"
              >
                <v-spacer></v-spacer>
                <v-btn text color="teal" @click="showNotAvailablePicker = false">Cancel</v-btn>
                <v-btn text color="teal" @click="$refs.dialog.save( badDates )">OK</v-btn>
              </v-date-picker>
            </v-dialog>

            <v-spacer></v-spacer>

          </v-card-text>
        </v-card>
      </v-col>

      <!-- save button -->
      <v-col cols="1"> 
        <v-btn class="mx-1 my-1" fab right dark x-small color="teal">
          <!-- <v-icon dark @click="handleSaveNewVolunteer(volunteers);"> mdi-content-save-outline</v-icon> -->
          <v-icon dark @click="handleSaveNewVolunteer();"> mdi-content-save-outline</v-icon>
        </v-btn>
        Save
      </v-col>

    </v-row>


    <!-- scheduling with and not-with saved for a future release -->

    <v-row>
        <!-- choose who to schedule with (or not) -->
 
                   <!-- choose roles & preferred times -->
      <v-col cols="4" class="white">
        <p class="text-left">Choose volunteers to schedule this person WITH (not functional, yet, but will save in database):</p>
        <v-checkbox class="ml-2 my-0 list-height"
          v-for="(volunteer, volIndex) in this.volunteerNames"
          v-bind:key="volIndex"
          v-model="schedWith"
          :value="`${volunteer.id}`"
          :label="`${volunteer.name}`"
          multiple
          color="teal"
        ></v-checkbox>

      </v-col>

      <v-col cols="4" offset="1" class="white">
        <p class="text-left">Choose volunteers to AVOID scheduling this person with (MUST note this for ALL volunteers involved):</p>
        <v-checkbox class="ml-2 my-0 list-height"
          v-for="(volunteer, volIndex) in this.volunteerNames"
          v-bind:key="volIndex"
          v-model="notWith"
          :value="`${volunteer.id}`"
          :label="`${volunteer.name}`"
          multiple
          color="teal"
        ></v-checkbox>

      </v-col>


    </v-row>
</v-container>
</v-app>


</template>



<script>
import axios from 'axios';
import moment from 'moment';
import {getAvatars} from '../js/fcns.js';


export default {
  name: "EditExistingVolunteer",
  data: function() {
    return {

      volunteer: {},
      _id: "",
      firstName: "",
      lastName: "",
      email: "",
      image: "",
      prefTimes: {},


      date: null,
      menu2: false,
      badDates: [],
      showNotAvailablePicker: false,
      timeSlots: [],
      badDates: [],

      eventTimes: [],
      roles: [],
      rolesChosen: [],
      preferredTime: -1,
      avatars: [],
      volunteerNames: [],
      schedWith: [],
      notWith: []

    }
  },


  created() {
    // get list of avatars
    this.avatars = getAvatars();
  },


  methods: {

    formatTime: function(timeSlot) {
      const day = timeSlot.day;
      // add random date to time so we can use moment to format it
      let time = new Date("March 16, 2020 " + timeSlot.time);
      time = moment(time).format("hh:mm a").toString();
      return `${day} at ${time}`;
    },

    // schedule start date has to be before (or on) today
    today: function() {
      const t = new Date().toJSON().slice(0,10);
      return t;
    },

    // save dates not available
    save: function (date) {
      var index = this.badDates.findIndex(x => x===date)

      if (index === -1){
          this.badDates.push(moment(date).format("YYYY-MM-DD").toString());
      }
      else {
        this.badDates.splice(index,1);
      }
    },

    // route to return to volunteers list
    handleReturnToVolunteerList: function() {
      this.$router.push({ name: 'Volunteers' })
    },

    // build object and insert in volunteers table
    // handleSaveNewVolunteer: function(volunteers) {
    handleSaveNewVolunteer: function() {

      this.image = this.image.toString();

      this.volunteer = {
        _id: this._id,
        firstName: this.firstName,
        lastName: this.lastName,
        email: this.email,
        image: this.image,
        roles: [this.rolesChosen],
        prefTimes: {
          day: this.timeSlots[this.preferredTime].day,
          time: this.timeSlots[this.preferredTime].time,
          percentPreferred: 1
        },
        notAvailable: this.badDates,
        with: this.schedWith,
        notWith: this.notWith
      };

      // update in volunteers table
      this.updateVolunteer(this.volunteer);
      // update volunteers array
      this.volunteers.push(this.volunteer);
      // return to volunteers list
      this.handleReturnToVolunteerList();
    },

    // axios call to update volunteer in volunteers table
    updateVolunteer: function(volunteer) {
      axios.post('/api/volunteers/volunteer', volunteer)
      .then(response => {
        console.log(response)
      })
      .catch(err => {
        console.log(err)
      });
    },

  },

  // created() {
  mounted() {
    // get data from parameters
    const id = this.$route.params.id;
    this.roles = this.$route.params.roles;
    this.timeSlots = this.$route.params.timeSlots;

    // this is passed in --- duplicates should already have been removed.
    
    this.volunteers = this.$route.params.volunteers;

    // find where in the volunteers array this volunteer is
    let index = this.volunteers.findIndex(volunteer => volunteer._id === id);
    // get that volunteer to edit
    let editVolunteer = this.volunteers[index];

    // update state
    this.rolesChosen = editVolunteer.roles[0]; // only one for this release

    this._id = this.volunteers[index]._id;
    this.firstName = this.volunteers[index].firstName;
    this.lastName = this.volunteers[index].lastName;
    this.email = this.volunteers[index].email;
    this.image = this.volunteers[index].image;

    this.badDates = this.volunteers[index].notAvailable;
    this.prefTimes = this.volunteers[index].prefTimes;

    this.preferredTime = this.timeSlots.findIndex(
      timeSlot => timeSlot.day === this.prefTimes[0].day &&
                  timeSlot.time === this.prefTimes[0].time
    );

    // It needs to be a string to match the v-model and turn the radio button on
    this.preferredTime = this.preferredTime.toString();

    this.schedWith = this.volunteers[index].with;
    this.notWith = this.volunteers[index].notWith;

    // drop old dates (before today)
    this.badDates = this.badDates.filter(badDate => moment(badDate) >= moment(Date.now()));

    // Get all volunteerNames
    this.volunteerNames = [];
    this.volunteers.forEach(volunteer => {
      this.volunteerNames.push({
        "id": volunteer._id,
        "name": volunteer.firstName + " " + volunteer.lastName,
      });
    });

  },

};

</script>

<style scoped>
/* tweak colors and spacing */
.bg-lightteal {
  background-color: #c4fff9 !important;
}

.list-height {
  height: 1.5em;
}

.volunteer-name {
  color:blue;
  font-size: 1.25em;
  padding: 20px 0 0;
  text-align: left;
}

.role {
  font-size: 16px;
  background-color:  #c4fff9 !important;
  height: 20px;
}

/* avatar selection same height as rest of row */
.match-height {
  height: 53.98px;
}

</style>