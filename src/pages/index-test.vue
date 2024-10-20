<template>
  <v-container>
    <v-card class="mt-4 animate__animated animate__fadeInUp">
      <v-card-title>AI Analysis:</v-card-title>
      <v-card-text>
        <v-data-table
          :headers="tableHeaders"
          item-key="id"
          :items="formattedAiAnalysis"
        />
      </v-card-text>
    </v-card>
  </v-container>
</template>
<script>
  import { computed, ref } from 'vue'
  import sampleOutput from '../assets/output.json'

  export default {
    setup () {
      const aiAnalysis = ref([])

      aiAnalysis.value = sampleOutput[0].message.content.changes.map((change, index) => ({
        id: index + 1,
        section: change.section,
        impact: change.impact,
      }))

      const tableHeaders = [
        { title: 'No', align: 'start', key: 'id' },
        { title: 'Affected Section', align: 'start', key: 'section' },
        { title: 'Summary of Impact', align: 'start', key: 'impact' },
      ]

      const formattedAiAnalysis = computed(() => {
        return aiAnalysis.value.map((change, index) => ({
          id: index + 1,
          section: change.section,
          impact: change.impact,
        }))
      })

      return {
        tableHeaders,
        formattedAiAnalysis,
      }
    },
  }
</script>
<style>
/* Add any additional styles here if needed */
</style>
