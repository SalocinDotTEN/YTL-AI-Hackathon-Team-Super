<template>
  <v-container>
    <v-row>
      <v-col cols="12" md="6">
        <v-file-input
          v-model="file1"
          accept="application/pdf"
          hint="Please upload the original PDF to compare against."
          label="Original PDF"
          outlined
          persistent-hint
          @change="loadPdf(1)"
        />
      </v-col>
      <v-col cols="12" md="6">
        <v-file-input
          v-model="file2"
          accept="application/pdf"
          hint="Please upload the updated PDF to compare with the original."
          label="Updated PDF"
          outlined
          persistent-hint
          @change="loadPdf(2)"
        />
      </v-col>
    </v-row>
    <v-btn
      class="mt-4 animate__animated animate__bounceIn"
      color="primary"
      :disabled="!file1 || !file2"
      :loading="loading"
      @click="uploadToEndpoint"
    > Compare PDFs </v-btn>
    <v-card>
      <v-tabs v-model="tab" grow>
        <v-tab value="comparison">Comparison</v-tab>
        <v-tab value="analysis">Analysis</v-tab>
      </v-tabs>
      <v-card-text>
        <v-tabs-window v-model="tab">
          <v-tabs-window-item value="comparison">
            <v-textarea
              label="Overall Summary"
              :model-value="Object.entries(overallSummary).length !== 0 ? overallSummary : ''"
              outlined
              readonly
              rows="5"
            />
            <v-data-table
              item-key="index"
              :items="comparisonResults"
              :loading="loading"
              :no-data-text="loading ? 'Loading...' : 'No data available'"
            />
          </v-tabs-window-item>
          <v-tabs-window-item value="analysis">
            <v-textarea
              label="Overall Analysis Summary"
              :model-value="Object.entries(analysisSummary).length !== 0 ? analysisSummary : ''"
              outlined
              readonly
              rows="5"
            />
            <v-data-table
              item-key="index"
              :items="analysisResults"
              :loading="loading"
              :no-data-text="loading ? 'Loading...' : 'No data available'"
            />
          </v-tabs-window-item>
        </v-tabs-window>
      </v-card-text>
    </v-card>
  </v-container>
</template>
<script>
  import { computed, ref } from 'vue'
  import axios from 'axios'

  export default {
    setup () {
      const file1 = ref(null)
      const file2 = ref(null)
      const loading = ref(false)
      const comparisonResults = ref([])
      const analysisResults = ref([])
      const overallSummary = ref({})
      const analysisSummary = ref({})

      const loadPdf = async fileNumber => {
        const file = fileNumber === 1 ? file1.value : file2.value
        if (!file) return
        comparisonResults.value = []
      }

      const uploadToEndpoint = async () => {
        // if (!file1.value || !file2.value) {
        //   alert('Please upload both PDF files before comparing.')
        //   return
        // }

        // loading.value = true
        // const formData = new FormData()
        // formData.append('Original_File', file1.value)
        // formData.append('Updated_File', file2.value)

        try {
          const response = await axios.get(
            'https://6714d38a690bf212c762a3ff.mockapi.io/tinkerers/comparison',
            {
              headers: {
                'Content-Type': 'text/json',
              },
            }
          )

          const content = response.data[0].message.content.comparison
          comparisonResults.value = content.sections
          overallSummary.value = JSON.stringify(content.overall_summary)

          try {
            const analysisResponse = await axios.get(
              'https://6714d38a690bf212c762a3ff.mockapi.io/tinkerers/analysis',
              {
                headers: {
                  'Content-Type': 'text/json',
                },
              }
            )
            // const analysisContent = analysisResponse.data[0]['Input 1'][0].message.content
            analysisResults.value = analysisResponse.data
            analysisSummary.value = 'Summary of analysis here'
          } catch (error) {
            console.error('Error analyzing:', error)
            alert('Error analyzing. Please try again.')
          } finally {
            loading.value = false
          }
        } catch (error) {
          console.error('Error uploading files:', error)
          alert('Error uploading files. Please try again.')
        } finally {
          loading.value = false
        }
      }

      return {
        file1,
        file2,
        loadPdf,
        loading,
        comparisonResults,
        analysisResults,
        overallSummary,
        analysisSummary,
        uploadToEndpoint,
      }
    },
    data () {
      return {
        tab: 'comparison',
      }
    },
  }
</script>
<style>
/* Add any additional styles here if needed */
</style>
