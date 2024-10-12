<template>
  <v-app>
    <v-main>
      <v-container>
        <h1 class="text-h4 mb-4">PDF Diff Analyzer</h1>
        <v-row>
          <v-col cols="12" md="6">
            <v-file-input
              v-model="file1"
              accept="application/pdf"
              label="PDF Document 1"
              outlined
              @change="loadPdf(1)"
            />
          </v-col>
          <v-col cols="12" md="6">
            <v-file-input
              v-model="file2"
              accept="application/pdf"
              label="PDF Document 2"
              outlined
              @change="loadPdf(2)"
            />
          </v-col>
        </v-row>
        <v-btn class="mt-4" color="primary" :disabled="!file1 || !file2" @click="comparePdfs"> Compare PDFs </v-btn>
        <v-card v-if="differences" class="mt-4">
          <v-card-title>Differences:</v-card-title>
          <v-card-text>
            <div v-html="differences" />
          </v-card-text>
        </v-card>
        <v-card v-if="aiAnalysis" class="mt-4">
          <v-card-title>AI Analysis:</v-card-title>
          <v-card-text>
            <p>{{ aiAnalysis }}</p>
          </v-card-text>
        </v-card>
      </v-container>
    </v-main>
  </v-app>
</template>
<script>
  import { ref } from 'vue'
  import { diff_match_patch } from 'diff-match-patch'
  import axios from 'axios'
  import * as pdfjsLib from 'pdfjs-dist'

  pdfjsLib.GlobalWorkerOptions.workerSrc = `//cdnjs.cloudflare.com/ajax/libs/pdf.js/4.7.76/pdf.worker.min.mjs`

  export default {
    setup () {
      const file1 = ref(null)
      const file2 = ref(null)
      const text1 = ref('')
      const text2 = ref('')
      const differences = ref('')
      const aiAnalysis = ref('')

      const dmp = new diff_match_patch()

      const loadPdf = async fileNumber => {
        const file = fileNumber === 1 ? file1.value : file2.value
        if (!file) return

        try {
          const arrayBuffer = await file.arrayBuffer()
          const pdf = await pdfjsLib.getDocument({ data: arrayBuffer }).promise
          let fullText = ''

          for (let i = 1; i <= pdf.numPages; i++) {
            const page = await pdf.getPage(i)
            const textContent = await page.getTextContent()
            const pageText = textContent.items.map(item => item.str).join(' ')
            fullText += pageText + ' '
          }

          if (fileNumber === 1) {
            text1.value = fullText
          } else {
            text2.value = fullText
          }
        } catch (error) {
          console.error('Error parsing PDF:', error)
          alert(`Error parsing PDF ${fileNumber}. Please try another file.`)
        }
      }

      const comparePdfs = async () => {
        if (!text1.value || !text2.value) {
          alert('Please upload and parse both PDF files before comparing.')
          return
        }

        const diff = dmp.diff_main(text1.value, text2.value)
        dmp.diff_cleanupSemantic(diff)

        let html = ''
        for (const [op, text] of diff) {
          if (op === 0) {
            html += `<span>${text}</span>`
          } else if (op === -1) {
            html += `<span class="red lighten-4 text-decoration-line-through">${text}</span>`
          } else {
            html += `<span class="green lighten-4">${text}</span>`
          }
        }
        differences.value = html

        try {
          const response = await axios.post('https://api.example.com/analyze-diff', {
            diff,
          })
          aiAnalysis.value = response.data.analysis
        } catch (error) {
          console.error('Error calling AI service:', error)
          aiAnalysis.value = 'Unable to perform AI analysis at this time.'
        }
      }

      return {
        file1,
        file2,
        differences,
        aiAnalysis,
        loadPdf,
        comparePdfs,
      }
    },
  }
</script>
<style>
/* Add any additional styles here if needed */
</style>
