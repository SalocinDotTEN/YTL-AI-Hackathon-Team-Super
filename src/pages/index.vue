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
    <v-card v-if="comparisonResults.length > 0" class="mt-4 animate__animated animate__fadeInUp">
      <v-card-title class="d-flex align-center">
        <span>Change Analysis Summary</span>
      </v-card-title>
      <v-card-text>
        <v-data-table
          class="elevation-1"
          :headers="headers"
          :items="tableItems"
          :items-per-page="10"
          @click:row="handleRowClick"
        >
          <template #item.changeCount="{ item }">
            {{ item.changeCount }}
          </template>
          <template #item.section="{ item }">
            <div>
              <strong>{{ item.section }}</strong>
              <div class="caption grey--text">Section {{ item.subSection }}</div>
            </div>
          </template>
          <template #expanded-item="{ headers, item }">
            <td class="pa-4" :colspan="headers.length">
              <v-row>
                <v-col cols="6">
                  <strong>Original Version:</strong>
                  <div class="mt-2">{{ item.oldVersion }}</div>
                </v-col>
                <v-col cols="6">
                  <strong>New Version:</strong>
                  <div class="mt-2">{{ item.newVersion }}</div>
                </v-col>
                <v-col class="mt-4" cols="12">
                  <strong>Changes Identified:</strong>
                  <div class="mt-2">{{ item.changesIdentified }}</div>
                </v-col>
              </v-row>
            </td>
          </template>
        </v-data-table>
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

      const headers = [
        {
          title: 'No.',
          align: 'center',
          value: 'changeCount',
          width: '100px',
        },
        {
          title: 'Affected Section',
          align: 'start',
          value: 'section',
          width: '300px',
        },
        {
          title: 'Summary of Impact',
          align: 'start',
          value: 'changesIdentified',
        },
      ]

      const tableItems = computed(() => {
        return comparisonResults.value.map((result, index) => ({
          ...result,
          changeCount: index + 1,
          id: index,
        }))
      })

      const parseResults = content => {
        const sections = content.split('---').filter(section => section.trim())

        return sections.map(section => {
          const lines = section.trim().split('\n')
          const result = {}

          // Parse section info
          const sectionMatch = section.match(/\*\*Section :\*\* (.*?) \*\*/)
          const subSectionMatch = section.match(/\*\* Sub-Section : (.*?)\*\*/)

          result.section = sectionMatch ? sectionMatch[1].trim() : ''
          result.subSection = subSectionMatch ? subSectionMatch[1].trim() : ''

          // Parse versions and changes
          lines.forEach(line => {
            if (line.includes('**Old Version**:')) {
              result.oldVersion = line.split('**Old Version**:')[1].trim()
            }
            if (line.includes('**New Version**:')) {
              result.newVersion = line.split('**New Version**:')[1].trim()
            }
            if (line.includes('**Changes Identified**:')) {
              result.changesIdentified = line.split('**Changes Identified**:')[1].trim()
            }
            if (line.includes('**Significance**:')) {
              result.significance = line.split('**Significance**:')[1].trim()
            }
          })

          if (result.significance === 'No change') {
            result.significance = 'Low'
          }

          return result
        })
      }

      const significanceCounts = computed(() => {
        const counts = {
          High: 0,
          Medium: 0,
          Low: 0,
        }

        comparisonResults.value.forEach(result => {
          if (result.significance) {
            counts[result.significance] = (counts[result.significance] || 0) + 1
          }
        })

        return counts
      })

      const getSignificanceColor = significance => {
        const colors = {
          High: 'error',
          Medium: 'warning',
          Low: 'success',
        }
        return colors[significance] || 'grey'
      }

      const handleRowClick = item => {
        // Handle row click if needed
        console.log('Row clicked:', item)
      }

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
          comparisonResults.value = parseResults(content)
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
        significanceCounts,
        getSignificanceColor,
        loadPdf,
        uploadToEndpoint,
        headers,
        tableItems,
        handleRowClick,
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
