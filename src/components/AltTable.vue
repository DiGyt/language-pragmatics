<template>
  <table class="table" id="myTable" v-if="content">
    <tr>
      <th v-for="(name, idx) in content.head" @click="sortTable(idx)">
        {{ name }}
      </th>
    </tr>
    <tr v-for="roww in content.data">
      <td v-if="roww[0] === 'Exampleton FC'" v-for="cell in roww" style="color:#ff0000;font-weight: bold;">
        {{ cell }}
      </td>
      <td v-if="roww[0] != 'Exampleton FC'" v-for="cell in roww">
        {{ cell }}
      </td>
    </tr>
  </table>
</template>


<script>
import Vue from 'vue';


export default {
  name: 'AltTable',
  props: ['content'],
  methods: {
    sortTable(n) {
      // A function to vertically sort a table (ascending or descending) via clicking
      // If row data can be numbers, it's sorted numerically, else alphabetically
      // based on https://www.w3schools.com/howto/howto_js_sort_table.asp

      var table, rows, switching, i, x, y, shouldSwitch, dir, switchcount = 0;
      table = document.getElementById("myTable");
      switching = true;
      //Set the sorting direction to ascending:
      dir = "asc";
      /*Make a loop that will continue until
      no switching has been done:*/
      while (switching) {
        //start by saying: no switching is done:
        switching = false;
        rows = table.rows;
        /*Loop through all table rows (except the
        first, which contains table headers):*/
        for (i = 1; i < (rows.length - 1); i++) {
          //start by saying there should be no switching:
          shouldSwitch = false;
          /*Get the two elements you want to compare,
          one from current row and one from the next:*/
          x = rows[i].getElementsByTagName("TD")[n].innerHTML.toLowerCase();
          y = rows[i + 1].getElementsByTagName("TD")[n].innerHTML.toLowerCase();
          // convert to numbers if possible
          var xArg = (isNaN(x)) ?  x : parseInt(x);
          var yArg = (isNaN(y)) ?  y : parseInt(y);
          /*check if the two rows should switch place,
          based on the direction, asc or desc:*/
          if (dir == "asc") {
            if (xArg > yArg) {
              //if so, mark as a switch and break the loop:
              shouldSwitch= true;
              break;
            }
          } else if (dir == "desc") {
            if (xArg < yArg) {
              //if so, mark as a switch and break the loop:
              shouldSwitch = true;
              break;
            }
          }
        }
        if (shouldSwitch) {
          /*If a switch has been marked, make the switch
          and mark that a switch has been done:*/
          rows[i].parentNode.insertBefore(rows[i + 1], rows[i]);
          switching = true;
          //Each time a switch is done, increase this count by 1:
          switchcount ++;
        } else {
          /*If no switching has been done AND the direction is "asc",
          set the direction to "desc" and run the while loop again.*/
          if (switchcount == 0 && dir == "asc") {
            dir = "desc";
            switching = true;
          }
        }
      }
    }
  }

};

</script>
<style>
.table {
  margin-left: auto;
  margin-right: auto;
  font-size: small;
  border-spacing: 0 0px;
  //border-collapse: separate;
}

th {
  background-color: #5187ba;
  color: white;
}

th,
td {
  //width: 150px;
  //text-align: center;
  border: 1px solid black;
  //padding: 0px;
  margin-bottom: 0;
  //display: block;
}


</style>
