<template>
  <div>
      <table>
          <thead>
              <tr>
                  <th>Piece</th>
                  <th>Rotation 1</th>
                  <th>Rotation 2</th>
                  <th>Rotation 3</th>
                  <th>Rotation 4</th>
              </tr>
          </thead>
          <tbody>
              <tr v-for="(piece, i) in basePieces" :key="'piece_' + i">
                  <td>{{i}}</td>
                  <td v-for="(rotation, j) in piece" :key="'rotation_' + j">
                      <svg :width="spread(rotation, a=>a.x)*cellSize" :height="spread(rotation, a=>a.y)*cellSize">
                          <rect
                        v-for="(cell, k) in rotation"
                        :key="'basePiece_' + i + '_rotation_' + j + '_cell' + k"
                        :x="(cell.x)*cellSize"
                        :y="(cell.y)*cellSize"
                        :width="cellSize"
                        :height="cellSize"
                        :class="'pieceCell type' + i"
                    />
                      </svg>
                    
                  </td>
              </tr>
          </tbody>
      </table>
  </div>
</template>

<script lang="ts">
import BasePieces from "./BasePieces"
export default {
    data(){
        return {
            basePieces: BasePieces,
            cellSize: 30,
            w: 50,
            h: 50
        }
    },
    methods:{
        spread(arr, f){
            return Math.max(...arr.map(a=>f(a))) - Math.min(...arr.map(a=>f(a))) + 1
        }
    }
}
</script>

<style>
th, td {
    padding: 8px;
}
table, th, tr, td {
    border: 1px solid black;
}
table{
    border-collapse: collapse;
}
</style>