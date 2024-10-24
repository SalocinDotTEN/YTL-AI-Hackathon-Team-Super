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
            <div v-html="comparisonResults" />
          </v-tabs-window-item>
          <v-tabs-window-item value="analysis">
            {{ analysisSummary }}
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

      function markdownToHtmlTable (markdownString) {
        // Split the markdown into lines
        const lines = markdownString.trim().split('\n')

        // Start building the HTML table
        let htmlTable = '<table>\n'

        // Process each line
        lines.forEach((line, index) => {
          // Remove leading and trailing pipe characters and split by pipes
          const cells = line.replace(/^\||\|$/g, '').split('|').map(cell => cell.trim())

          // Determine if it's a header row
          const isHeader = index === 0

          // Start a new row
          htmlTable += '  <tr>\n'

          // Process each cell
          cells.forEach(cell => {
            // Determine if it's a header cell
            const cellType = isHeader ? 'th' : 'td'

            // Convert bold markdown to <strong> tags
            cell = cell.replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')

            // Add the cell to the row
            htmlTable += `    <${cellType}>${cell}</${cellType}>\n`
          })

          // Close the row
          htmlTable += '  </tr>\n'

          // Add a horizontal rule after the header row
          if (isHeader) {
            htmlTable += '  <tr><td colspan="' + cells.length + '"><hr></td></tr>\n'
          }
        })

        // Close the table
        htmlTable += '</table>'

        return htmlTable
      }

      const uploadToEndpoint = async () => {
        if (!file1.value || !file2.value) {
          alert('Please upload both PDF files before comparing.')
          return
        }

        loading.value = true
        // const formData = new FormData()
        // formData.append('Original_File', file1.value)
        // formData.append('Updated_File', file2.value)

        try {
          const response = await axios.get(
            // 'https://6714d38a690bf212c762a3ff.mockapi.io/tinkerers/comparison',
            `https://puv111.app.n8n.cloud/webhook-test/upload2compareoai?Old%20doc=${file1.value.name}&New%20doc=${file2.value.name}`,
            {
              headers: {
                'Access-Control-Allow-Origin': '*',
                'Content-Type': 'application/json',
              },
            }
          )

          let content = response.data[0].response.text
          content = content.replace(/markdown/g, '')
          content = content.replace(/```/g, '')
          // content = JSON.parse(content)
          comparisonResults.value = markdownToHtmlTable(content)
          // changedSections.value = content['Changed Sections']
          // overallSummary.value = JSON.stringify(content.overall_summary)

          try {
            const analysisResponse = await axios.get(
              // 'https://6714d38a690bf212c762a3ff.mockapi.io/tinkerers/analysis',
              `https://puv111.app.n8n.cloud/webhook-test/analysis?Olddoc=${file1.value.name}&Newdoc=${file2.value.name}`,
              {
                headers: {
                  'Access-Control-Allow-Origin': '*',
                  'Content-Type': 'application/json',
                },
              }
            )
            // const analysisContent = analysisResponse.data[0]['Input 1'][0].message.content
            // analysisResults.value = analysisResponse.data[0].response.text
            analysisSummary.value = analysisResponse.data[0].response.text
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
