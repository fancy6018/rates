<script>
import axios from 'axios';
import { reactive } from '@vue/reactivity';
import { onMounted } from '@vue/runtime-core';

export default {  
  setup(){
    const tableSetting = reactive({
      data:{},
      datafinal:{},
      dataTotal:0,
      rowSel:5,
      rowArr:[5,25,50,100],      
      pages:1,
      pageTotal:1,
      idxStart:1,
      idxEnd:1,
      asc:true,
      sortData: ()=>{      
        if(tableSetting.asc){
          tableSetting.data = Object.values(tableSetting.data).sort((a,b)=>{ return a['value'] - b['value'] })
        }else{
          tableSetting.data = Object.values(tableSetting.data).sort((a,b)=>{ return b['value'] - a['value'] })
        }
        tableSetting.asc = !tableSetting.asc;

        tableSetting.pageIdx();
      },
      pagination: ()=>{
        tableSetting.pages = 1;
        tableSetting.dataTotal = Object.keys(tableSetting.data).length
        tableSetting.pageTotal = Math.ceil(tableSetting.dataTotal / tableSetting.rowSel);
              
        tableSetting.pageIdx();   
      },
      pageIdx:()=>{
        tableSetting.idxStart = tableSetting.pages * tableSetting.rowSel - tableSetting.rowSel + 1; 
        tableSetting.idxEnd = tableSetting.pages * tableSetting.rowSel;
        if(tableSetting.idxEnd > tableSetting.dataTotal) tableSetting.idxEnd = tableSetting.dataTotal; 
        
        tableSetting.datafinal = Object.keys(tableSetting.data).filter((obj)=>{
          if(obj >= tableSetting.idxStart - 1 && obj <= tableSetting.idxEnd - 1){            
            return tableSetting.data[obj]
          }                   
        }).map((obj)=>{
          return tableSetting.data[obj]
        })        
      },
      prevPage:()=>{        
        if(tableSetting.pages > 1) tableSetting.pages--;
        tableSetting.pageIdx();
      },
      nextPage:()=>{
        if(tableSetting.pages < tableSetting.pageTotal) tableSetting.pages++;
        tableSetting.pageIdx();
      },
    });

    const getData = ()=>{
      var url = 'https://api.coingecko.com/api/v3/exchange_rates';
      axios
      .get(url)
      .then((res)=>{
        tableSetting.data = res.data.rates;
        
        tableSetting.sortData();
        tableSetting.pagination()
      })
      .catch((error)=>{
        console.log(error)
      })
    }

    onMounted(()=>{
      getData();
    })

    return { tableSetting };
  }
}
</script>

<template>
  <div class="rate">
    <table>
      <thead>
        <th>Name</th>
        <th>Type</th>
        <th>Unit</th>
        <th @click="tableSetting.sortData">Value <i :class="['fas',  tableSetting.asc ? 'fa-caret-down' : 'fa-caret-up']"></i></th>
      </thead>
      <tbody>
        <tr v-for="(item,index) in tableSetting.datafinal" :key="item.name">
          <td>{{item.name}}</td>
          <td>{{item.type}}</td>
          <td>{{item.unit}}</td>
          <td>{{item.value}}</td>
        </tr>
      </tbody>
      <tfoot>
        <tr><td colspan="4" style="text-align:right">
          Row count:
          <select v-model="tableSetting.rowSel" @change="tableSetting.pagination">
            <option v-for="item in tableSetting.rowArr" :value="item" :key="item">{{item}}</option>
          </select>
          &nbsp;
          {{tableSetting.idxStart}}-{{tableSetting.idxEnd}} of {{tableSetting.dataTotal}}
          <i :class="['fas', 'fa-chevron-left' , { 'opacity' : tableSetting.pages==1 }]" @click="tableSetting.prevPage"></i>
          <i :class="['fas', 'fa-chevron-right' , { 'opacity' : tableSetting.pages==tableSetting.pageTotal }]" @click="tableSetting.nextPage"></i>
        </td></tr>        
      </tfoot>
    </table>
  </div>
</template>

<style scoped lang="scss">
   @import url(https://pro.fontawesome.com/releases/v5.10.0/css/all.css);

   .rate table{
      width:100%;
      min-width: 300px;     
      border-collapse:collapse;
   }
   th:first-child,tr td:first-child{
     text-align: left;
   }
   th:not(:first-child),tr td:not(:first-child){
     text-align: right;
   }
   .rate th{
     background: #212529;
     color:#fff;
   }
   .rate th,.rate td{
     padding:10px 5px;
   }
   .fas{
     padding-left: 10px;
   }
   .rate table tbody tr:nth-child(odd){
     background-color: rgba(0, 0, 0, 0.05);
   }
   .opacity{
     filter: opacity(0.2);
   }
</style>
