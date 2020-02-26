<template>
  <div class="container mx-auto p-8">
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
    <div class="border border-gray-400 p-4 mt-4">
      <h2 class="font-semibold">Key dates for immunisations:</h2>
      <div v-for="(immunisation, idx) in immunisationData" :key="idx" class="border border-gray-800 my-2 p-6">
        <p class="font-bold text-xs">{{ milestoneText(immunisation.milestone) }}</p>
        <h3 class="font-bold text-lg mb-4">{{ immunisation.title }}</h3>
        <p >{{ immunisation.description }}</p>
        <p class="mt-4 text-sm" v-if="hasCalculated">{{ calculateDate(immunisation.milestone) }}</p>
        <button v-if="hasCalculated" type="submit" @click.prevent="handleAddToCalendar(immunisation)" class="border border-gray-800 px-4 py-2 mt-3 rounded text-gray-800 font-medium text-sm">Add to calendar</button>
      </div>
    </div>
    <!-- <div v-if="hasCalculated" class="border border-gray-400 p-4 mt-4">
      <h2 class="font-semibold">Key dates for kindergartens:</h2>
    </div> -->
  </div>
</template>

<script>
import moment from 'moment';
// import ics from 'ics';
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

const kindergartenData = {

};

export default {
  data() {
    return {
      date: null,
      hasCalculated: false,
      immunisationData: [],
      popoverProps: {
        visibility: 'focus',
        placement: 'bottom'
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
  },
  methods: {
    handleClick() {
      this.hasCalculated = true;
    },
    handleInput() {
      this.hasCalculated = false;
    },
    handleAddToCalendar(immunisation) {
      const eventInfo = {
        start: [2020, 2, 28, 9, 0],
        duration: { hours: 1 },
        title: immunisation.title,
        description: immunisation.description,
      };

      const event = ics.createEvent(eventInfo);

      console.warn('event:', event);

      window.open('data:text/calendar;charset=utf8,' + event);
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
    }
  },
}
</script>

<style>
</style>
