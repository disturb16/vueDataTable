# VueDataTable
Vue + Materialize datatable component

# Pre-requisits & Installation
Vuejs 2.0 and Materialize.js are needed. Then paste the dataTable.vue file in your project and import it
To export data to CSV papaparse is needed. `npm install papaparse`

Add component:

```
import dataTable from './path-to-datatable.vue'

new Vue({
  ...
  components:{
    'data-table': dataTable
  }
})

//-- or globally ---

Vue.component('data-table', dataTable)

```

## Basic usage:
```
const jsonArray = [
        {
          name: 'Scarleth J.',
          Age: 24,
          userName: 'scar',
          email:'scar@doe.com'
        },
        {

          name: 'Creg Tyrn',
          Age: 26,
          userName: 'cregT',
          email:'tyrn@doe.com'
        },
        {
          name: 'Viviam St.',
          Age: 21,
          userName: 'vivst',
          email:'viv@doe.com'
        },
        {
          name: 'John R.',
          Age: 25,
          userName: 'johnr',
          email:'jphn@doe.com'
        }
      ]
```
Add it to component:

```
<data-table
  :dataSource="jsonArray" />
  ```

Output:

![](preview.png?raw=true)

###  Options

#### Hiding Columns

`const excludedColumns = ['id', 'department', 'date']`

```
<data-table
  :dataSource="jsonArray"
  :excludedColumns ="excludedColumns" />
  ```
#### Cell Value format

you can use the `cellValueFormat` attribute to stylize your data, it accepts two params: `columnName` and `value`,
with these you can validate which column you want to format.

for example:
```
new Vue({

  ...

  methods:{
    cellValueFormat(colName, value){
      let formatted = ''
      switch(colName){
        case 'date':
          formatted = dateFormat(value)
        break;

        case 'hour':
          formatted = dateToTime(value)
        break;

        case 'name':
          formatted = capitalize(value)
        break;

        default:
          return value
      }

      return formatted
    }
  }
})
```

then pass it to dataTable:

```
<data-table
  :dataSource="jsonArray"
  :excludedColumns ="excludedColumns"
  :cellValueFormat="cellValueFormat" />
  ```

  #### Extra Columns

  You can pass extra columns to datasource

  `const extraColumns = ["Actions"]`

  then you can pass values

  `extraColumnsValues` is a method that receives current row and current column name

```
  extraColumnsValues(colName, row){
      switch(colName){
        case 'Actions':
          const id = row.userId
          return `<a href="#/user-profile/${id}" class="more-details-link">Check more</a>
                  <a href="#" class="btn red">Delete</a>`
        break;
      }
    }
  ```

  ```
  <data-table
  :dataSource="jsonArray"
  :extraColumns="extraColumns"
  :extraColumnsValues="extraColumnsValues" />
  ```

  #### Row Style
  You can use `rowStyleConditions` to give different css classes to table rows, it works using Vuejs Class & style biding

  `rowStyleConditions` is a method which receives current row

  ```
    rowStyleConditions(row){
     const mins = row.late_mins
      return {
        low_late: mins > 0 && mins < 12,
        mid_late: mins >= 12 && mins <= 30,
        high_late: mins > 30
      }
    }
  ```
  Then you can use it as:

  ```
    <data-table
    :dataSource="jsonArray"
    :rowStyleConditions="rowStyleConditions" />
  ```
#### What if  you want to pass rendered components?

You can use the Vuejs slot system, and it only works for extraColumns, here is an example:
```
const data = [...]
const extraColumns = ['Validate_User']

<data-table
  :dataSource="data"
  :extraColumns="extraColumns">

  <template slot-scope="dataRow" slot="Validate_User">
    <primary-button @clicked="AuthUser(dataRow.row.userId">Validate</primary-button>
  </template>

</data-table>
```

Note that the named slot has the same name of your extra-column and to access data of the current row you can use the
`slot-scope` attribute, this has the row data, then you can access `dataRow.row.your-value`