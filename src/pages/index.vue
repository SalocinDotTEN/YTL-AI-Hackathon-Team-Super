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
      <v-tabs
        v-model="tab"
        grow
      >
        <v-tab value="comparison">Comparison</v-tab>
        <v-tab value="analysis">Analysis</v-tab>
      </v-tabs>
      <v-card-text>
        <v-tabs-window v-model="tab">
          <v-tabs-window-item value="comparison">
            <v-textarea
              label="Overall Summary"
              :model-value="overallSummary ? overallSummary.summary : ''"
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
            Analysis Table
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
      const overallSummary = ref({})

      const tableItems = computed(() => {
        return comparisonResults.value.map((result, index) => ({
          ...result,
          changeCount: index + 1,
          id: index,
        }))
      })

      const loadPdf = async fileNumber => {
        const file = fileNumber === 1 ? file1.value : file2.value
        if (!file) return
        comparisonResults.value = []
      }

      const uploadToEndpoint = async () => {
        if (!file1.value || !file2.value) {
          alert('Please upload both PDF files before comparing.')
          return
        }

        loading.value = true
        const formData = new FormData()
        formData.append('Original_File', file1.value)
        formData.append('Updated_File', file2.value)

        try {
          const response = await axios.post(
            'https://puv111.app.n8n.cloud/webhook-test/ai-tinkerers-kl-hackathon',
            formData,
            {
              headers: {
                'Content-Type': 'multipart/form-data',
              },
            }
          )

          const content = response.data['Input 1'][0].message.content
          comparisonResults.value = content.comparison
          overallSummary.value = content.overall_summary
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
        loading,
        comparisonResults,
        overallSummary,
        loadPdf,
        uploadToEndpoint,
        tableItems,
      }
    },
    data () {
      return {
        tab: null,
      }
    },
  }
</script>
<style>
.v-data-table>>>.v-data-table__wrapper {
  overflow-x: auto;
}

.v-chip {
  margin-right: 8px;
}

.caption {
  font-size: 0.75rem !important;
}

.expanded-content {
  background-color: #f5f5f5;
}
</style>
