<template>
  <div class="diagram">
    <ul v-if="sortedDeals.length">
      <Deal v-for="deal of sortedDeals" :key="deal._id" :deal="deal" :minDate="minDate" :maxDate="maxDate" :duration="duration"/>
    </ul>
  </div>
</template>

<script>
import { DateTime, Duration } from 'luxon'
import Deal from './Deal.vue'

export default {
  name: 'DealsDiagram',
  props: {
    deals: {
      default: [],
      type: Array,
    },
  },
  components: {
    Deal,
  },
  computed: {
    sortedDeals() {
      return this.deals
      .map(deal => this.transformDealDates(deal))
      .sort(
        (deal1, deal2) => deal2.priority - deal1.priority ||
          deal1.startDate.diff(deal2.startDate).milliseconds,
      )
    },
    dates() {
      let maxDate = 0
      let minDate = Infinity

      for (let deal of this.sortedDeals) {
        let startDate = deal.startDate.valueOf()
        let endDate = deal.endDate.valueOf()

        if (startDate < minDate) {
          minDate = startDate
        }

        if (endDate > maxDate) {
          maxDate = endDate
        }
      }

      return {
        minDate: DateTime.fromMillis(minDate, { zone: 'utc' }),
        maxDate: DateTime.fromMillis(maxDate, { zone: 'utc' })
      }
    },
    minDate() {
      return this.dates.minDate
    },
    maxDate() {
      return this.dates.maxDate
    },
    duration() {
      if (this.sortedDeals.length) {
        return this.maxDate.diff(this.minDate)
      }
      return Duration.fromObject({ days: 1 })
    },
  },
  methods: {
    transformDealDates(deal) {
      const newDeal = Object.assign({}, deal)

      newDeal.startDate = DateTime.fromISO(newDeal.startDate, { zone: 'utc' })
      newDeal.endDate = DateTime.fromISO(newDeal.endDate, { zone: 'utc' })

      if (newDeal.endDateActual) {
        newDeal.endDateActual = DateTime.fromISO(newDeal.endDateActual, { zone: 'utc' })
      }

      return newDeal
    },
  },
}
</script>

<style scoped>
.diagram ul {
  list-style-type: none;
  padding: 0;
}
</style>
