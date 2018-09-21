<template>
  <div>
    <div class="row">
      <ul v-for="(dd, ddindex) in ddList"
        :key="dd.id"
        :id='dd.id'
        :tabindex="ddindex"
        class='dropdown-content'>
        <li @click="selectFilter('')" tabindex="0"><a href="#!">Todo</a></li>

        <li v-for="(item, index) in getDDList(dd.colName)" 
          :tabindex="index+1"
          :key="item" 
          @click="selectFilter(dd.colName, item)">
          <a href="#">{{getValueFormat(dd.colName, item)}}</a>
        </li>
      </ul>
    </div>
    
    <div v-if="dataSource.length > 0">
      <a href="#!" 
        class="btn-flat btn-small waves-effect waves-light green white-text"
        @click="exportDataToCSV">Export<i class="material-icons right">file_download</i></a>
    </div>

    <table>
      <thead>
        <tr>
          <th v-for="extra in extraColumns" :key="extra+randId()">{{extra}}</th>
          <th v-for="dd in ddList"
          :key="dd.id"> <a href="" class="column-filter" :data-target="dd.id">{{dd.colName | colName | capitalize}} <i class="material-icons">expand_more</i></a></th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="row in dataFiltered" :key="randId()+row[visibleColumns[0]]" :class="getRowStyle(row)">

          <!-- values for extra columns -->
          <td v-for="extraCol in extraColumns" :key="extraCol+randId()">
            <span v-html="getExtraValueFormat(extraCol, row)"/>
            <slot :name="extraCol" v-bind:row="row"></slot>
          </td>

          <!-- values from datasource -->
          <td v-for="col in visibleColumns" :key="col"> {{ getValueFormat(col, row[col]) }} </td>
          
        </tr>
      </tbody>
    </table>
  </div>

</template>

<script>
import Papa from 'papaparse'

export default {
  name: 'datatable',
  props: ['dataSource', 'excludedColumns', 
          'rowStyleConditions','cellValueFormat',
          'extraColumns', 'extraColumnsValues'],
  data(){
    return{
      activeFilter: '',
      firstUpdated: false
    }
  },
  mounted(){
    this.refreshDropDowns()
  },
  
  updated() {
    if(!this.firstUpdated)
      this.refreshDropDowns()
  },

  methods:{

    refreshDropDowns(){
      const elems = document.querySelectorAll('.column-filter')
      const instances = M.Dropdown.init(elems, {
        coverTrigger: false,
        constrainWidth: false
      })
      
    },

    validValue(val){
      if (val == null || typeof val == 'undefined' || val == "")
        return false
      return true
    },

  /**
  * @param {String} property Column name
  */
    getDDList(property){
      let arr = []
      this.dataSource.forEach(item=>{
        const repeated = arr.find(arrItem=> arrItem === item[property])
        if(!repeated)
          arr.push(item[property])
      })

      return arr.sort()
    },

    /**
    * @param {String} param Column name to be filtered
    * @param {String} value value to be filtered
    */

    selectFilter(param, value){
      if (!param || !value)
        this.activeFilter = ''

      const filter = {
        param,
        value
      }
      this.activeFilter = filter
    },

    randId(){
      const max = 100000
      return Math.floor(Math.random() * Math.floor(max))
    },

    getRowStyle(row){
      if(!this.validValue(this.rowStyleConditions))
        return ''
      
      return this.rowStyleConditions(row)
    },

    getValueFormat(colName, value){
      if(!this.validValue(this.cellValueFormat))
        return value
      
      return this.cellValueFormat(colName, value)
    },

    getExtraValueFormat(col, row){
      if(!this.validValue(this.extraColumnsValues))
        return ''
      
      return this.extraColumnsValues(col, row)
    },

    exportDataToCSV(){

      const dataToExport = []

      this.dataFiltered.forEach(row=>{
        const  ob = {}
        this.visibleColumns.forEach(col=>{
          const colName = col.charAt(0).toUpperCase() +col.slice(1)
          ob[colName] = this.getValueFormat(col, row[col])
        })
        dataToExport.push(ob)
      })


      const csv = Papa.unparse(dataToExport)

      const file = new Blob([csv], {type: 'csv'})
      const a = document.createElement("a")
      const url = URL.createObjectURL(file)
      a.href = url
      a.download = `exported-data${this.randId()}.csv`
      document.body.appendChild(a)
      a.click()
      setTimeout(function() {
          document.body.removeChild(a)
          window.URL.revokeObjectURL(url)
      }, 0)
    }
  },
  computed:{

    columns(){
      if(!this.validValue(this.dataSource))
        return []
      if(this.dataSource instanceof Array)
        return Object.keys(this.dataSource[0])
      else
        return []
    },

    /**
     * @returns {Array}
     */
    ddList(){
      let listOfDD = []
      this.visibleColumns.forEach(col=>{
        listOfDD.push({id: 'dd_'+col+this.randId(), colName: col})
      })
      return listOfDD
    },

    /**
     * @returns {Array}
     */
    dataFiltered(){
      if(this.activeFilter == '')
        return this.dataSource
      
      const filterParam = this.activeFilter.param
      const value = this.activeFilter.value

      return this.dataSource.filter(item=> item[filterParam] == value)
    },

    visibleColumns(){
      if(!this.excludedColumns)
        return this.columns

      return this.columns.filter(col=> !this.excludedColumns.includes(col))
    }
  },

  watch:{
    dataSource: function(source){
      this.refreshDropDowns()
    }
  },

  filters:{
    colName(value){
      
      if(typeof value == "string"){
        const arr = value.split("_")
        let header = ""
        arr.forEach(word =>{
          header += word.charAt(0).toUpperCase() + word.slice(1) + " "
        })
        return header.trim()
      }
      else
        return value
    },

    capitalize(value){
    if(typeof value == "string")
      return value.charAt(0).toUpperCase() +value.slice(1)
    else
      return value
    }

  } 

}
</script>
