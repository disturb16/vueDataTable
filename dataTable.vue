<template>
  <div>
    <div class="row">
      <ul v-for="dd in ddList" :key="dd.id" :id='dd.id' class='dropdown-content'>
        <li @click="selectFilter('')"><a>Todo</a></li>
        <li v-for="item in getDDList(dd.colName)" :key="item" @click="selectFilter(dd.colName, item)" v-if="item">
          <a>{{getValueFormat(dd.colName, item)}}</a></li>
      </ul>
    </div>

    <table>
      <thead>
        <tr>
          <th v-for="extra in extraColumns" :key="extra+randId()">{{extra}}</th>
          <th v-for="dd in ddList"
          :key="dd.id"> <a href="#" class="dropdown-trigger" :data-target="dd.id">{{dd.colName | colName | capitalize}} <i class="material-icons">expand_more</i></a></th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="row in dataFiltered" :key="randId()+row[visibleColumns[0]]" :class="getRowStyle(row)">
          <td v-for="extraCol in extraColumns" :key="extraCol+randId()"  v-html="getExtraValueFormat(extraCol, row)" />
          <td v-for="col in visibleColumns" :key="col"> {{ getValueFormat(col, row[col]) }} </td>
        </tr>
      </tbody>
    </table>
  </div>

</template>

<script>

export default {
  name: 'datatable',
  props: ['dataSource', 'excludedColumns', 
          'rowStyleConditions','formattingRules',
          'extraColumns', 'extraColumnsFormattingRules'],
  data(){
    return{
      activeFilter: ''
    }
  },
  mounted(){
    this.refreshDropDowns()
  },
  updated(){
    this.refreshDropDowns()
  },

  methods:{

    refreshDropDowns(){
      const ddTriggerEls = document.querySelectorAll('.dropdown-trigger');
      const insTriggers = M.Dropdown.init(ddTriggerEls, {constrainWidth: false})

      this.$emit('afterDataLoad')
    },

    validValue(val){
      if (val == null || typeof val == 'undefined' || val == "")
        return false
      return true
    },

    getDDList(property){
      let arr = []
      this.dataSource.forEach(item=>{
        const repeated = arr.find(arrItem=> arrItem === item[property])
        if(!repeated)
          arr.push(item[property])
      })

      return arr
    },
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
      if(!this.validValue(this.formattingRules))
        return value
      
      return this.formattingRules(colName, value)
    },
    getExtraValueFormat(col, row){
      if(!this.validValue(this.extraColumnsFormattingRules))
        return value
      
      return this.extraColumnsFormattingRules(col, row)
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

    ddList(){
      let listOfDD = []
      this.visibleColumns.forEach(col=>{
        listOfDD.push({id: 'dd_'+col, colName: col})
      })
      return listOfDD
    },
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

  filters:{
    colName(value){
      if(typeof value == "string")
        return value.replace(/_/g, " ")
      else
        return value
    }
  }

  
}
</script>
