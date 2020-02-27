<template>
  <div class="container mx-auto p-8">
    <h1 class="text-4xl mb-8 leading-none">Add to Calendar Demo</h1>

    <div class="flex mb-4">
      <button @click="setView('immunisations')" class="border border-gray-600 rounded px-2 py-2 text-xs w-1/2 mr-1">Immunisations</button>
      <button @click="setView('kindergartens')" class="border border-gray-600 rounded px-2 py-2 text-xs w-1/2">Kindergartens</button>
    </div>

    <form class="border border-gray-400 p-4">
      <label class="block mb-2" for="date-of-birth">Enter your child's date of birth:</label>
      <div class="date-picker-container relative md:w-1/4 hover:cursor-pointer">
        <v-date-picker
          v-model="date"
          color="gray"
          :popover="popoverProps"
          :input-props="inputProps"
          :is-required="true"
          :available-dates="availableDates"
          @input="handleInput"
        />
        <svg class="absolute top-0 right-0 w-5 h-full mr-2 fill-current text-gray-700 pointer-events-none" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512"><path d="M452 40h-24V0h-40v40H124V0H84v40H60C26.916 40 0 66.916 0 100v352c0 33.084 26.916 60 60 60h392c33.084 0 60-26.916 60-60V100c0-33.084-26.916-60-60-60zm20 412c0 11.028-8.972 20-20 20H60c-11.028 0-20-8.972-20-20V188h432v264zm0-304H40v-48c0-11.028 8.972-20 20-20h24v40h40V80h264v40h40V80h24c11.028 0 20 8.972 20 20v48z"/><path d="M76 230h40v40H76zM156 230h40v40h-40zM236 230h40v40h-40zM316 230h40v40h-40zM396 230h40v40h-40zM76 310h40v40H76zM156 310h40v40h-40zM236 310h40v40h-40zM316 310h40v40h-40zM76 390h40v40H76zM156 390h40v40h-40zM236 390h40v40h-40zM316 390h40v40h-40zM396 310h40v40h-40z"/></svg>
      </div>
      <button type="submit" :disabled="!date" @click.prevent="handleClick" class="disabled:opacity-75 disabled:cursor-not-allowed bg-gray-800 px-4 py-2 mt-3 rounded text-white font-medium">Calculate dates</button>
    </form>

    <div v-show="currentView === 'immunisations'" class="border border-gray-400 p-4 mt-4">
      <h2 class="font-semibold">Key dates for immunisations:</h2>
      <div v-for="(immunisation, idx) in immunisationData" :key="idx" class="border border-gray-800 my-2 p-6">
        <p class="font-bold text-xs">{{ milestoneText(immunisation.milestone) }}</p>
        <h3 class="font-bold text-lg mb-4">{{ immunisation.title }}</h3>
        <p >{{ immunisation.description }}</p>
        <p class="mt-4 text-sm" v-if="hasCalculated">{{ calculateDate(immunisation.milestone) }}</p>

        <div v-show="hasCalculated" title="Add to Calendar" class="addeventatc mt-3">
          Add to calendar
          <span class="start">{{ startDate(immunisation.milestone) }}</span>
          <span class="timezone">Australia/Melbourne</span>
          <span class="all_day_event">true</span>
          <span class="title">{{ immunisation.title }}</span>
          <span class="description">{{ immunisation.description }}</span>
          <span class="alarm_reminder">60</span>
        </div>
      </div>
    </div>

    <div v-show="currentView === 'kindergartens'" class="border border-gray-400 p-4 mt-4">
      <h2 class="font-semibold">Key dates for kindergartens:</h2>
      <div class="border border-gray-800 my-2 p-6">
        <p class="font-bold text-xs"><span v-if="hasCalculated">{{ openDate }}</span></p>
        <h3 class="font-bold text-lg mb-4">Applications Open</h3>
        <p>You can apply after your child turns 2. The data you enrol does not secure your place on a wait list or give you the higher priority, so children with later birthdays are not disadvantaged.</p>

        <div v-show="hasCalculated" title="Add to Calendar" class="addeventatc mt-3">
          Add to calendar
          <span class="start">{{ openDate }}</span>
          <span class="timezone">Australia/Melbourne</span>
          <span class="all_day_event">true</span>
          <span class="title">Applications Open 📚</span>
          <span class="description">You can apply after your child turns 2. The data you enrol does not secure your place on a wait list or give you the higher priority, so children with later birthdays are not disadvantaged. <a href="https://google.com.au">Google</a></span>
        </div>
      </div>

      <div class="border border-gray-800 my-2 p-6">
        <p class="font-bold text-xs">May <span v-if="hasCalculated">{{ openDaysYear }}</span></p>
        <h3 class="font-bold text-lg mb-4">Attend Kindergarten Open Days</h3>
        <p>While you can arrange a tour with a kindergarten throughout the year, BKCES kindergartens hold Open days in late May.</p>

        <div v-show="hasCalculated" title="Add to Calendar" class="addeventatc mt-3">
          Add to calendar
          <span class="start">{{ `05/01/${openDaysYear}` }}</span>
          <span class="timezone">Australia/Melbourne</span>
          <span class="all_day_event">true</span>
          <span class="title">Attend Kindergarten Open Days 🍎</span>
          <span class="description">While you can arrange a tour with a kindergarten throughout the year, BKCES kindergartens hold Open days in late May.</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import moment from 'moment';
import { saveAs } from 'file-saver';
const ics = require('ics');

const mockImmunisationData = [
  {
    title: 'Infanrix hexa, Prevenar 1, Rotaris',
    description: 'Disease: Diphtheria, tetanus, pertussis, hepatitis B, poliomyelitis, Haemophilus influenzae type b; Pneumococcal, Rotavirus',
    milestone: {
      type: 'range',
      lower: {
        value: 6,
        period: 'weeks'
      },
      upper: {
        value: 8,
        period: 'weeks'
      }
    }
  },
  {
    title: 'Infanrix hexa, Prevenar 13, Rotarix',
    description: 'Disease: Diphtheria, tetanus, pertussis, hepatitis B, poliomyelitis, Haemophilus influenzae type b; Pneumococcal, Rotavirus',
    milestone: {
      type: 'exact',
      value: 4,
      period: 'months'
    }
  },
  {
    title: 'Infanrix hexa',
    description: 'Disease: Diphtheria, tetanus, pertussis, hepatitis B, poliomyelitis, Haemophilus influenzae type b; Pneumococcal, Rotavirus',
    milestone: {
      type: 'exact',
      value: 6,
      period: 'months'
    }
  },
  {
    title: 'M-M-R II/Priorix, Nimenrix, Prevenar 13',
    description: 'Disease: Diphtheria, tetanus, pertussis, hepatitis B, poliomyelitis, Haemophilus influenzae type b; Pneumococcal, Rotavirus',
    milestone: {
      type: 'exact',
      value: 12,
      period: 'months'
    }
  },
  {
    title: 'Priorix-Tetra/ProQuad, Infanrix/Tripacel, ActHIB',
    description: 'Disease: Diphtheria, tetanus, pertussis, hepatitis B, poliomyelitis, Haemophilus influenzae type b; Pneumococcal, Rotavirus',
    milestone: {
      type: 'exact',
      value: 18,
      period: 'months'
    }
  },
  {
    title: 'nfranrix IPV/Quadracel',
    description: 'Disease: Diphtheria, tetanus, pertussis, hepatitis B, poliomyelitis, Haemophilus influenzae type b; Pneumococcal, Rotavirus',
    milestone: {
      type: 'exact',
      value: 4,
      period: 'years'
    }
  },
];

const mockKindergartenData = [

];

export default {
  data() {
    return {
      date: null,
      hasCalculated: false,
      currentView: 'immunisations',
      immunisationData: [],
      kindargartenData: [],
      popoverProps: {
        visibility: 'focus',
        placement: 'top'
      },
      inputProps: {
        id: 'date-of-birth',
        placeholder: 'DD/MM/YYYY',
        class: 'block w-full px-4 py-2 bg-gray-200 placeholder-gray-700'
      },
      availableDates: {
        start: moment().subtract(5, 'years').toDate(),
        end: null
      }
    }
  },
  mounted() {
    this.immunisationData = mockImmunisationData;
    this.kindargartenData = mockKindergartenData;
  },
  methods: {
    setView(newView) {
      this.currentView = newView;
    },
    handleClick() {
      this.hasCalculated = true;
    },
    handleInput() {
      this.hasCalculated = false;
    },
    milestoneText(milestoneObj) {
      if (milestoneObj.type === 'exact') {
        return `${milestoneObj.value} ${milestoneObj.period}`
      } else if (milestoneObj.type === 'range') {
        return `${milestoneObj.lower.value} ${milestoneObj.lower.period}-${milestoneObj.upper.value} ${milestoneObj.upper.period}`
      }
    },
    calculateDate(milestoneObj) {
      if (milestoneObj.type === 'exact') {
        return moment(this.date).add(milestoneObj.value, milestoneObj.period).format('Do MMMM YYYY');
      } else if (milestoneObj.type === 'range') {
        return moment(this.date).add(milestoneObj.upper.value, milestoneObj.upper.period).format('Do MMMM YYYY');
      }
    },
    startDate(milestoneObj) {
      if (milestoneObj.type === 'exact') {
        return moment(this.date).add(milestoneObj.value, milestoneObj.period).format('DD-MM-YYYY');
      } else if (milestoneObj.type === 'range') {
        return moment(this.date).add(milestoneObj.upper.value, milestoneObj.upper.period).format('DD-MM-YYYY');
      }
    }
  },
  computed: {
    openDate() {
      return moment(this.date).add(2, 'years').format('Do MMMM YYYY');
    },
    openDaysYear() {
      return moment(this.date).add(2, 'years').format('YYYY');
    }
  }
}
</script>

<style>

/* Hide AddEvent.com branding */
.addeventatc .addeventatc_dropdown {
  padding: 6px 0;
}

/* Hide AddEvent.com branding */
.addeventatc .addeventatc_dropdown .copyx {
  display: none;
}

/* Hide icons */
.addeventatc .addeventatc_dropdown > span {
  background: none;
  padding-left: 10px;
  text-align: center;
}
</style>
