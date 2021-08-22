<template>
    <div class="aligned center aligned grid mt-5">
        <div class="home pb-5">
          <v-row>
            <v-card max-height="60%" class="mx-auto my-auto pa-10" width="600px" elevation="12">
                <v-file-input v-model="files" label="File input" prepend-icon="mdi-file" @change="uploadFile"></v-file-input>
                <v-card-actions><v-btn color="primary" primary large @click="deleteCollection()" block :loading="loading" :disabled="loading">Save To Internet</v-btn></v-card-actions>
            </v-card>
          </v-row>
        </div>
        <v-snackbar v-model="snackbar" absolute bottom color="success" outlined right> Saved Successfully </v-snackbar>
    </div>
</template>
<script>
import XLSX from 'xlsx'
import axios from 'axios'
export default {
  data () {
    return {
      desserts: [],
      snackbar: false,
      loading: false,
      files: null
    }
  },
  methods: {
    async uploadFile () {
      const file = this.files
      console.log(file);
      const types = file.name.split('.')[1]
      const fileType = ['xlsx', 'xls'].some((item) => item === types)
      if (!fileType) {
        return this.$snackbar().showError('File type error, please select again')
      }
      let fileList = []
      const tabJson = await this.file2Xce(file)
      if (tabJson && tabJson.length > 0) {
        this.xlsxJson = tabJson
        fileList = this.xlsxJson[0].sheet
        this.desserts = JSON.parse(JSON.stringify(fileList))
        console.log(this.desserts)
      }
    },
    file2Xce (file) {
      return new Promise((resolve) => {
        const reader = new FileReader()
        reader.onload = function (e) {
          const data = e.target.result
          this.wb = XLSX.read(data, { type: 'binary' })
          const result = []
          const sheet2JSONOpts = { defval: '' }
          this.wb.SheetNames.forEach((sheetName) => {
            result.push({
              sheetName: sheetName,
              sheet: XLSX.utils.sheet_to_json(this.wb.Sheets[sheetName], sheet2JSONOpts)
            })
          })
          resolve(result)
        }
        reader.readAsBinaryString(file)
      })
    },
    saveToFB () {
      this.desserts.forEach(x => {
        axios.post('https://traineesapi.firebaseio.com/zkjxdchskdj.json', JSON.stringify(x))
      })
      this.snackbar = true
      this.loading = false
    },
    deleteCollection () {
      this.loading = true
      axios.delete('https://traineesapi.firebaseio.com/zkjxdchskdj.json').then(() => {
        this.saveToFB()
      })
    }
  }
}
</script>
