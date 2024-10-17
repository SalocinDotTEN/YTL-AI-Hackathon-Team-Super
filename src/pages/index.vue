<template>
  <v-container>
    <v-row>
      <v-col cols="12" md="6">
        <v-file-input
          v-model="file1"
          accept="application/pdf"
          class="animate__animated animate__fadeInLeft"
          label="Original PDF"
          outlined
          @change="loadPdf(1)"
        />
      </v-col>
      <v-col cols="12" md="6">
        <v-file-input
          v-model="file2"
          accept="application/pdf"
          class="animate__animated animate__fadeInRight"
          label="Updated PDF"
          outlined
          @change="loadPdf(2)"
        />
      </v-col>
    </v-row>
    <v-btn
      class="mt-4 animate__animated animate__bounceIn"
      color="primary"
      :disabled="!file1 || !file2"
      @click="comparePdfs"
    > Compare PDFs </v-btn>
    <v-card v-if="differences" class="mt-4 animate__animated animate__fadeInUp">
      <v-card-title>Differences (red is removed, green is added):</v-card-title>
      <v-card-text>
        <div v-html="differences" />
      </v-card-text>
    </v-card>
    <v-card v-if="aiAnalysis" class="mt-4 animate__animated animate__fadeInUp">
      <v-card-title>AI Analysis:</v-card-title>
      <v-card-text>
        <p>{{ aiAnalysis }}</p>
      </v-card-text>
    </v-card>
  </v-container>
</template>
<script>
  import { ref } from 'vue'
  import { diff_match_patch } from 'diff-match-patch'
  import axios from 'axios'
  import * as pdfjsLib from 'pdfjs-dist'
  import { gapi } from 'gapi-script'

  pdfjsLib.GlobalWorkerOptions.workerSrc = `//cdnjs.cloudflare.com/ajax/libs/pdf.js/4.7.76/pdf.worker.min.mjs`

  export default {
    setup () {
      const file1 = ref(null)
      const file2 = ref(null)
      const text1 = ref('')
      const text2 = ref('')
      const html1 = ref('')
      const html2 = ref('')
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
          let fullHtml = ''

          for (let i = 1; i <= pdf.numPages; i++) {
            const page = await pdf.getPage(i)
            const textContent = await page.getTextContent()
            const pageText = textContent.items.map(item => item.str).join(' ')
            const pageHtml = textContent.items.map(item => `<span>${item.str}</span>`).join(' ')
            fullText += pageText + ' '
            fullHtml += `<div>${pageHtml}</div>`
          }

          if (fileNumber === 1) {
            text1.value = fullText
            html1.value = fullHtml
            await uploadToGoogleDrive(file, '1IDWV99Em-6pKCuaxObBlNsyvbAnTpNME')
          } else {
            text2.value = fullText
            html2.value = fullHtml
            await uploadToGoogleDrive(file, '1w_h_qpSHDxfWQCVGDvGPe6VhH4d6GgcE')
          }
        } catch (error) {
          console.error('Error parsing PDF:', error)
          alert(`Error parsing PDF ${fileNumber}. Please try another file.`)
        }
      }

      const uploadToGoogleDrive = async (file, folderId) => {
        const CLIENT_ID = '857888640400-6khe7mul45dgamd58r58pgvgcgioor7o.apps.googleusercontent.com'
        const API_KEY = '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'
        const SCOPES = 'https://www.googleapis.com/auth/drive.file'

        gapi.load('client:auth2', () => {
          gapi.client.init({
            apiKey: API_KEY,
            clientId: CLIENT_ID,
            discoveryDocs: ['https://www.googleapis.com/discovery/v1/apis/drive/v3/rest'],
            scope: SCOPES,
          }).then(() => {
            gapi.auth2.getAuthInstance().signIn().then(() => {
              const accessToken = gapi.auth.getToken().access_token
              const form = new FormData()
              form.append('metadata', new Blob([JSON.stringify({
                name: file.name,
                mimeType: file.type,
                parents: [folderId],
              })], { type: 'application/json' }))
              form.append('file', file)

              fetch('https://www.googleapis.com/upload/drive/v3/files?uploadType=multipart', {
                method: 'POST',
                headers: new Headers({ Authorization: 'Bearer ' + accessToken }),
                body: form,
              }).then(response => response.json()).then(data => {
                console.log('File uploaded to Google Drive:', data)
              }).catch(error => {
                console.error('Error uploading file to Google Drive:', error)
              })
            })
          })
        })
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
            html += `<span class="bg-red lighten-4">${text}</span>`
          } else {
            html += `<span class="bg-green lighten-4">${text}</span>`
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
        text1,
        text2,
        html1,
        html2,
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
