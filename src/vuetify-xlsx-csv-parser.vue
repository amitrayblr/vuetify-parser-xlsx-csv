<template>
  <div>
    <v-file-input
      v-model="file"
      accept=".csv, .xlsx, .xls"
      label="Upload Spreadsheet"
      :hint="hint"
      :loading="loading"
      loader-height="5"
      @change="upload()"
    />

    <div v-if="error">
      {{ error }}
    </div>

    <v-dialog
      v-if="filedata"
      v-model="dialog"
      max-width="1000"
      scrollable
    >
      <v-card>
        <v-toolbar flat>
          <v-toolbar-title>
            Review Columns or fields
          </v-toolbar-title>
          <v-spacer />
          <v-btn
            icon
            @click="dialog = false"
          >
            <v-icon>mdi-close</v-icon>
          </v-btn>
        </v-toolbar>
        <div v-if="filedata && filedata.length">
          <v-simple-table
            fixed-header
            height="400px"
          >
            <template v-slot:default>
              <thead>
                <tr>
                  <th
                    v-for="col in usercolumns"
                    :key="col"
                    class="text-left"
                  >
                    {{ col }}
                    <v-select
                      v-model="colmap[col]"
                      :items="columns"
                      item-text="label"
                      item-value="key"
                      clearable
                      dense
                    />
                  </th>
                </tr>
              </thead>
              <tbody>
                <tr
                  v-for="(row, index) in rows"
                  :key="index"
                >
                  <td
                    v-for="(cell, c) in row"
                    :key="c"
                  >
                    {{ cell }}
                  </td>
                </tr>
              </tbody>
            </template>
          </v-simple-table>
        </div>
        <v-card-actions>
          <v-spacer />
          <v-btn
            text
            @click="dialog = false"
          >
            Done
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>

import XLSX from 'xlsx'
import { 
  VFileInput,
  VDialog,
  VCard,
  VToolbar,
  VToolbarTitle,
  VSpacer,
  VBtn,
  VIcon,
  VSimpleTable,
  VSelect,
  VCardActions,
} from 'vuetify/lib'


export default {
  name: 'UploadSpreadsheet',
  components: {
    VFileInput,
    VDialog,
    VCard,
    VToolbar,
    VToolbarTitle,
    VSpacer,
    VBtn,
    VIcon,
    VSimpleTable,
    VSelect,
    VCardActions,
  },
  props: {
    columns: {
      type: Array,
      default: () => [],
    },
    hint: {
      type: String,
      default: ''
    },
  },
  data () {
    return {
      error: null,
      loading: false,
      file: null,
      filedata: null,
      dialog: false,
      colmap: {},
      usercolumns: [],
      results: [],
    }
  },
  computed: {
    rows () {
      if (this.filedata && this.filedata.length > 1) {
        return this.filedata.slice(1, (this.filedata.length))
      } else {
        return this.filedata
      }
    }
  },
  watch: {
    filedata (data) {
      if (data) {
        //  populate results
        if (this.columns && this.columns.length) {
          this.dialog = true
          this.mapCols(data)
        } else {
          this.emitResults()
        }
      }
    },
    dialog (to) {
      if (!to) {
        // if closing then check results and emit.
        this.emitResults()
        this.loading = false
        this.file = null
        this.error = null
      }
    }
  },
  methods: {
    upload () {
      this.loading = true
      //  clear results if existing
      this.$emit('results', [])
      this.parseFile()
        .then((filedata) => {
          this.error = null
          this.filedata = filedata
          //  console.log(' --- results ', results)
          this.loading = false
        })
        .catch((error) => {
          this.error = error
          this.file = null
          this.loading = false
        })
    },
    parseFile () {
      return new Promise((resolve, reject) => {
        const reader = new FileReader()
        reader.onload = (event) => {
          this.parseXlsx(event.target.result)
            .then(result => {
              return resolve(result.worksheet)
            })
            .catch(error => {
              return reject(error)
            })
        }

        if (this.file.type === 'text/csv') {
          reader.readAsText(this.file)
        } else {
          reader.readAsBinaryString(this.file)
        }
      })
    },
    parseXlsx (filedata) {
      const workbook = typeof window === 'undefined' ? XLSX.readFile(filedata) : XLSX.read(filedata, { type: 'binary' })
      const sheetNames = workbook.SheetNames
      if (sheetNames.length === 0) {
        return Promise.reject(new Error('Error in sheet'))
      }
      const sheets = workbook.Sheets
      const json = XLSX.utils.sheet_to_json(sheets[sheetNames[0]], {
        header: 1,
        defVal: '',
        blankrows: false,
      })

      if (json.length === 0) {
        return Promise.reject(new Error('Empty sheet'))
      }

      return Promise.resolve({ worksheet: json })
    },

    //  column mapping methods
    mapCols (fileData) {
      if (!fileData) {
        return
      }
      var cols = []
      this.usercolumns = []
      if (fileData && fileData.length) {
        fileData[0].forEach((name) => {
          cols.push({ name })
          this.usercolumns.push(name)
        })
      }

      var colmap = {}
      this.columns.forEach((item) => {
        //  first try without breaking the header into words
        //  if the key is already processed then don't try it again.
        if (!colmap[item.key]) {
          cols.forEach(function (col, i) {
            if (item.patterns) {
              item.patterns.forEach((pattern) => {
                if (!colmap[item.key]) {
                  if (pattern === col.name.trim().toLowerCase()) {
                    colmap[col.name] = item.key
                    // move the item out of the cols
                    cols.splice(i, 1)
                  }
                }
              })
            }
          })
        }

        //  check each of the pattern item to match within col field
        if (!colmap[item.key]) {
          cols.forEach((col, i) => {
            if (item.patterns) {
              item.patterns.forEach((pattern) => {
                if (!colmap[item.key]) {
                  if (col.name.trim().toLowerCase().indexOf(pattern) !== -1) {
                    colmap[col.name] = item.key
                    // move the item out of the cols
                    cols.splice(i, 1)
                  }
                }
              })
            }
          })
        }
        //  if no match then do regex match
        if (!colmap[item.key]) {
          cols.forEach((col, i) => {
            //  regex check with the patterns
            var re = new RegExp(item.patterns.join('|'), 'i')
            if (re.test(col.name)) {
              colmap[col.name] = item.key
              // move the item out of the cols
              cols.splice(i, 1)
            }
          })
        }
      })

      this.colmap = colmap
    },
    emitResults () {
      var results = []
      //  iterate and populate
      if (this.filedata && this.filedata.length > 1) {
        var headers = this.filedata[0]
        this.filedata.slice(1, (this.filedata.length)).forEach((row) => {
          var obj = {}
          headers.forEach((name, i) => {
            if (this.colmap[name] !== undefined) {
              obj[this.colmap[name]] = row[i]
            }
          })
          results.push(obj)
        })
      }
      if (results && results.length) {
        this.$emit('results', results)
      }
    }
  }
}

</script>

<style scoped>

</style>
