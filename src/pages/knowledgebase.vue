<template>
  <v-app>
    <v-main>
      <v-container>
        <h1 class="text-h4 mb-4">Knowledgebase Upload</h1>
        <v-form @submit.prevent="uploadFiles">
          <v-file-input
            v-model="files"
            accept="*/*"
            label="Select Files"
            multiple
            outlined
            @change="handleFileChange"
          />
          <v-btn color="primary" type="submit">Upload Files</v-btn>
        </v-form>
        <v-alert v-if="successMessage" class="mt-4" type="success">
          {{ successMessage }}
        </v-alert>
        <v-alert v-if="errorMessage" class="mt-4" type="error">
          {{ errorMessage }}
        </v-alert>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
  import { ref } from 'vue'
  import axios from 'axios'

  export default {
    setup () {
      const files = ref([])
      const successMessage = ref('')
      const errorMessage = ref('')

      const handleFileChange = event => {
        files.value = event.target.files
      }

      const uploadFiles = async () => {
        if (files.value.length === 0) {
          errorMessage.value = 'Please select files to upload.'
          return
        }

        const formData = new FormData()
        for (let i = 0; i < files.value.length; i++) {
          formData.append('files', files.value[i])
        }

        try {
          const response = await axios.post('https://puv111.app.n8n.cloud/webhook/uploadinternalfiles', formData, {
            headers: {
              'Content-Type': 'multipart/form-data',
            },
          })

          if (response.status === 200 && response.data.myField === 'Files have been uploaded') {
            successMessage.value = response.data.myField
            errorMessage.value = ''
          } else {
            throw new Error('Unexpected response from the server.')
          }
        } catch (error) {
          console.error('Error uploading files:', error)
          successMessage.value = ''
          errorMessage.value = 'Error uploading files. Please try again later.'
        }
      }

      return {
        files,
        successMessage,
        errorMessage,
        handleFileChange,
        uploadFiles,
      }
    },
  }
</script>

<style>
.v-container {
  margin-top: 20px;
}
</style>
