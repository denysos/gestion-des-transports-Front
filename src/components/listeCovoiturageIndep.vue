<template lang="">
    <div>
        <h2>Liste des covoiturages pour : </h2>
        <p>  {{$store.state.storeUser.user.prenom}} {{$store.state.storeUser.user.nom}} - 
          <v-chip :color="getHistoryColor(date)">
                          {{date}}
            </v-chip>
            -
            <v-chip :color="getHistoryColor('2000-01-01')">
                          Historique
            </v-chip>
            -
            <v-chip :color="getHistoryColor('9999-01-01')">
                          A venir
            </v-chip>
           </p>
        <v-data-table  :headers="headers" :sort-by="['dateDepart', 'villeDepart']" :sort-desc="[false, false]"
        :items="listecovoiturage" @click:row="afficherDetail" single-select>
          <template v-slot:item.dateDepart="{ item }">
            <v-chip :color="getHistoryColor(item.dateDepart)">
                          {{ formatDateDisplay(item.dateDepart) }}
            </v-chip>
          </template>
         
            <template v-slot>
             <!-- <button @click="afficherDetail(item, row)">Reserver</button> -->
             <button>Reserver</button>
            </template>
            <!-- <template v-slot:item.placesRestantes="{ item }">
              <p>
                {{item.placesDisponibles}}
              </p>
            </template> -->
        </v-data-table> 
          <div v-if="valeursDetail" >
            <v-btn @click="()=>valeursDetail=null">
              <v-icon color="red">
                mdi-close-box
              </v-icon>
            </v-btn>
            <div v-if="isAlreadyParticipant">
            <CovoiturageDetail :covoiturage="valeursDetail" :placesrestantes="placesrestantes"/>
            </div>
            <div v-if="!isAlreadyParticipant">
            <CovoiturageDetail :covoiturage="valeursDetail" resapossible :placesrestantes="placesrestantes"/>
            </div>
            <p></p>
            <CovoiturageParticipants :participants="valeursParticipants" :idcovoiturage="valeursDetail[0].id" :isHistory="isHistory(dateDetail)"/>
          </div>
    </div>
    
</template>
<script>
import CovoiturageDetail from "./CovoiturageDetail.vue";
import CovoiturageParticipants from "./CovoiturageParticipants.vue"
import {dateApp, cleanDate} from "../utils/dateUtils";
export default {
  name: "listeCovoiturageIndep",
  props: {
    listecovoiturage : {},
    date : String,
    
  },
  components: {
    CovoiturageDetail,
    CovoiturageParticipants
  },
  data() {
    return {
      headers: [
        { text: "date/heure départ", value: "dateDepart" },
        { text: "ville départ", value: "villeDepart" },
        { text: "ville arrivée", value: "villeArrivee" },
        { text: "places disponibles", value: "placesDisponibles" },
        // { text: "places restantes", value: "placesRestantes" },
        { text: "statut", value: "status" },
        { text: "actions", value: "detail" },
      ],
      userId: this.$store.state.storeCovoiturage.user.id,
      valeursDetail: null,
      valeursParticipants: null,
      // date: dateApp(),
      dateDetail: "",
      // resapossible: true,
      placesrestantes: 0,
      isAlreadyParticipant: false
      };
  },
  methods: {
    formatDateDisplay(dateTimeString) {
      return dateTimeString.split("T").join("  à ");
    },

    afficherDetail(item, row) {
      this.placesrestantes = item.placesDisponibles
      this.valeursDetail = [item];
      this.valeursParticipants = item.participant;
      row.select(true);
      this.dateDetail = item.dateDepart;
      let userId = this.$store.state.storeUser.user.id;
      this.isAlreadyParticipant = false;
      let tbParticipantUser = item.participant.filter(participant => participant.id == userId);
      console.log("tbParticipantUser : " + tbParticipantUser)
      if(tbParticipantUser.length > 0) {
        this.isAlreadyParticipant = true
      }
      
      
    },
    getHistoryColor(dateparm) {
      if (this.isHistory(dateparm)) {
        return "red"
      }
      if (this.isToday(dateparm)) {
        return "orange"
      }
      return "green";
    },
    isHistory(dateparm) {
      let dateItem = cleanDate(dateparm);
      let dateNow = dateApp();
      if (dateItem < dateNow) {
        return true;
      }
      return false;
    },
    
    isToday(dateparm) {
      let dateItem = dateparm.split("T")[0];
      // let dateNow = dateApp();
      let dateNow = this.$props.date; //pour afficher la date de recherche en orange
      if (dateItem == dateNow) {
        return true
      }
      return false
    },
    
  },

beforeCreate() {
    this.$store.getters.getAllCovoiturageUserId;
  },
};
</script>
<style>
tr.v-data-table__selected {
    background: #BBDEFB !important;
  }
</style>