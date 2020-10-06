# vuetify-parser-xlsx-csv
A simple Vuetify based module that extracts data from **.csv**, **.xlsx**, **.xls** files in the form of a JSON. It uses the [xlsx](https://www.npmjs.com/package/xlsx) module to parse the file. Be sure to check it out

## Installation
`npm install vuetify-parser-xlsx-csv`

## Working
The module ask for a file input, and then asks you to choose your labels for the columns in the file. After that is done, you will receive the results in the form of a JSON. Check the usage section for a detailed example.

## Usage
This is a simple example using the module to display a demo .xlsx file.

### Data
Here's where the data looks like.

| First Name | Last Name | Email | Phone |
|------------|-----------|-------|-------|
| John | Doe | johndoe@example.com | 123 456 8790 |
| Alan | Shepard | commandershepard@example.com | 456 879 1230 |
| Carl | Johnson | carljohnson@example.com | 789 456 1230 |
| Leon | Kennedy | leonkennedy@example.com | 123 879 4560 |

### Code
And here's the code
```html
<template>
  <v-app>
    <v-main>
      <v-container>
        <h4>File Upload</h4>
        
        <vuetifyXlsxCsvParser
          :columns="columns"
          :hint="help"
          @results="processResults"
        >

        </vuetifyXlsxCsvParser>

        <v-divider></v-divider>
        
        <div v-if="results && results.length">

          <v-simple-table fixed-header height="400px">
            <template v-slot:default>
              <thead>
                <tr>
                  <th class="text-left" v-for="col in columns" :key="col.key">
                    {{col.label}}
                  </th> 
                </tr>
              </thead>
              <tbody>
                <tr v-for="(row, index) in results" :key="index">
                  <td v-for="col in columns" :key="col.key">
                    {{ row[col.key] }}
                  </td>
                </tr>
              </tbody>
            </template>
          </v-simple-table>
        </div>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import vuetifyXlsxCsvParser from 'vuetify-xlsx-csv-parser';

export default {
  name: 'App',

  components: {
    vuetifyXlsxCsvParser,
  },

  data: () => {
    return {
      columns: [
        {
          key: 'firstname',
          label: 'First Name',
          patterns: ['first name']
        },
        {
          key: 'lastname',
          label: 'Last Name',
          patterns: ['last name', 'given name', 'family name', 'surname']
        },
        {
          key: 'email',
          label: 'Email',
          patterns: ['email', 'e-mail', 'e mail', 'mail']
        },
        {
          key: 'mobile',
          label: 'Mobile',
          patterns: ['phone', 'mobile', 'telephone', 'contact', 'number', 'cell']
        },
      ],
      results: null,
      help: 'Columns needed are First Name, Last Name, Email, Mobile',
    }
  },
  methods: {
    processResults(results) {
      this.results = results
      console.log(results)
    }
  },
};
</script>

```
### JSON Format
The JSON is stored in results, and each row of the file is a separate element
```javascript
[
  {
    "firstname": "John",
    "lastname": "Doe",
    "email": "johndoe@example.com",
    "mobile": "123 456 8790"
  },
  {
    "firstname": "Alan",
    "lastname": "Shepard",
    "email": "commandershepard@example.com",
    "mobile": "456 879 1230"
  },
  {
    "firstname": "Carl",
    "lastname": "Johnson",
    "email": "carljohnson@example.com",
    "mobile": "789 456 1230"
  },
  {
    "firstname": "Leon",
    "lastname": "Kennedy",
    "email": "leonkennedy@example.com",
    "mobile": "123 879 4560"
  }
]
```
