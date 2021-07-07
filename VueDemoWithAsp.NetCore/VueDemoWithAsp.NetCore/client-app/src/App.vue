<template>
  <div id="app">
    <ejs-schedule
      height="550px"
      width="100%"
      :selectedDate="selectedDate"
      :eventSettings="eventSettings"
    >
      <e-views>
        <e-view option="Week" startHour="07:00" endHour="15:00"></e-view>
        <e-view option="WorkWeek" startHour="10:00" endHour="18:00"></e-view>
        <e-view option="Month" showWeekend="false"></e-view>
      </e-views>
    </ejs-schedule>
  </div>
</template>
<script>
/* eslint-disable */
import Vue from "vue";
import { scheduleData } from "./datasource.js";
import { extend } from "@syncfusion/ej2-base";
import {
  SchedulePlugin,
  WorkWeek,
  Month,
  Week,
} from "@syncfusion/ej2-vue-schedule";
import axios from "axios";
import * as M from "materialize-css";
import moment from "moment";

Vue.use(SchedulePlugin);
export default {
  name: "app",
  computed: {
    hasEvents() {
      return this.isLoading === false && this.events.length > 0;
    },
    noEvents() {
      return this.isLoading === false && this.events.length === 0;
    },
  },
 data() {
   
   return {
     title: "",
     description: "",
     events: [],
     isLoading: true,
     startDate: "",
     startTime: "",
     endDate: "",
     endTime: "",
     selectedEvent: "",
     selectedEventId: 0,
     selectedDate: new Date(2021, 1, 15),
     eventSettings: { dataSource: extend([], scheduleData, null, true) }   
   };
 },
 methods: {
   addEvent() {
     const event = {
       startDate: this.startDate ? moment( this.startDate ).format( "YYYY-MM-DD" ) : null,
       startTime: this.startTime ? moment( this.startTime ).format( "YYYY-MM-DD HH:mm:00" ) : null,
       endDate: this.endDate ? moment( this.endDate ).format( "YYYY-MM-DD" ) : null,
       endTime: this.endTime ? moment( this.endTime ).format( "YYYY-MM-DD HH:mm:00" ) : null,
       title: this.title,
       description: this.description
     };
     axios
       .post( "/api/events", event )
       .then( () => {
         this.startDate = "";
         this.startTime = "";
         this.endDate = "";
         this.endTime = "";
         this.title = "";
         this.description = "";
         this.loadEvents();
       } )
       .catch( err => {
         this.msg = err.message;
         console.log( err );
       } );
   },
   confirmDeleteEvent( id ) {
     const event = this.events.find( e => e.id === id );
     this.selectedEvent = `'${ event.title }' on ${ event.startDate }${ event.startTime ? ` at ${ event.startTime }` : "" }`;
     this.selectedEventId = event.id;
     const dc = this.$refs.deleteConfirm;
     const modal = M.Modal.init( dc );
     modal.open();
   },
   deleteEvent( id ) {
     axios
       .delete( `/api/events/${ id }` )
       .then( this.loadEvents )
       .catch( err => {
         this.msg = err.message;
         console.log( err );
         this.loadEvents();
       } );
   },
   formatDate( d ) {
     return d ? moment.utc( d ).format( "MMM D, YYYY" ) : "";
   },
   formatTime( t ) {
     return t ? moment( t ).format( "h:mm a" ) : "";
   },
   formatEvents( events ) {
     return events.map( event => {
       return {
         id: event.id,
         title: event.title,
         description: event.description,
         startDate: this.formatDate( event.startDate ),
         startTime: this.formatTime( event.startTime ),
         endDate: this.formatDate( event.endDate ),
         endTime: this.formatTime( event.endTime )
       };
     } );
   },
   loadEvents() {
     axios
       .get( "/api/events" )
       .then( res => {
         this.isLoading = false;
         this.events = this.formatEvents( res.data );
       } )
       .catch( err => {
         this.msg = err.message;
         console.log( err );
       } );
   }
 },
 mounted() {
   return this.loadEvents();
 },
  provide: {
    schedule: [WorkWeek, Month, Week],
  },
};
</script>
<style>
@import "../node_modules/@syncfusion/ej2-base/styles/material.css";
@import "../node_modules/@syncfusion/ej2-buttons/styles/material.css";
@import "../node_modules/@syncfusion/ej2-calendars/styles/material.css";
@import "../node_modules/@syncfusion/ej2-dropdowns/styles/material.css";
@import "../node_modules/@syncfusion/ej2-inputs/styles/material.css";
@import "../node_modules/@syncfusion/ej2-navigations/styles/material.css";
@import "../node_modules/@syncfusion/ej2-popups/styles/material.css";
@import "../node_modules/@syncfusion/ej2-vue-schedule/styles/material.css";
</style>
