<template>
  <div style="display:flex">
    <svg :width="w*cellSize" :height="h*cellSize">
      <rect :width="w*cellSize" :height="h*cellSize" />
      <template v-for="y in h">
        <template v-for="x in w">
          <rect
            :key="'cell_'+ x + '_' + y"
            :x="(x-1)*cellSize"
            :y="(y-1)*cellSize"
            :width="cellSize"
            :height="cellSize"
            class="cell"
          />
        </template>
      </template>

      <rect
        v-for="(cell, i) in basePieces[fallingPiece.basePiece][fallingPiece.rotation]"
        :key="'piece_' + fallingPiece.id + '_cell_' + i"
        :x="(fallingPiece.x+cell.x)*cellSize"
        :y="(fallingPiece.y+cell.y)*cellSize"
        :width="cellSize"
        :height="cellSize"
        :class="'pieceCell type' + fallingPiece.basePiece"
      />
      <rect
        v-for="(cell,i) in droppedPieces"
        :key="'droppedPiece_' + i"
        :x="(cell.x)*cellSize"
        :y="(cell.y)*cellSize"
        :width="cellSize"
        :height="cellSize"
        :class="'pieceCell type' + cell.basePiece"
      />
    </svg>
    <div>
      <p>Lines: {{ completedLines }}</p>
      <button @click="pause" v-if="intervalId">Pause</button>
      <button @click="resume" v-if="!intervalId">Resume</button>
      <button @click="restart">Restart</button>
      <p>Next: <svg>
        <rect
        v-for="(cell, i) in basePieces[bag[0]][0]"
        :key="'next_piece_' + bag[0] + '_cell_' + i"
        :x="(cell.x)*cellSize"
        :y="(cell.y)*cellSize"
        :width="cellSize"
        :height="cellSize"
        :class="'pieceCell type' + bag[0]"
      />
        </svg>
        </p>
      <pre>{{bag}}</pre>
    </div>
  </div>
</template>

<script>
import BasePieces from "./BasePieces";
export default {
  props: { w: { type: Number }, h: { type: Number } },
  data() {
    return {
      codes: {
        left: "ArrowLeft",
        right: "ArrowRight",
        down: "ArrowDown",
        rotate: "ArrowUp",
        instantDrop: "Space"
      },
      delay: 1000,
      intervalId: 0,
      cellSize: 40,
      basePieces: BasePieces,
      fallingPiece: null,
      droppedPieces: [],
      completedLines: 0,
      bag: []
    };
  },
  created() {
    window.addEventListener("keydown", e => {
      this.keyup(e);
    });
    this.spawnNextPiece();
    this.resume();
  },
  computed: {
    leftWall() {
      var result = [];
      for (var y = -4; y < this.h + 4; y++) {
        result.push({ x: -1, y });
      }
      return result;
    },
    rightWall() {
      var result = [];
      for (var y = -4; y < this.h + 4; y++) {
        result.push({ x: this.w, y });
      }
      return result;
    },
    floor() {
      var result = [];
      for (var x = 0; x < this.w; x++) {
        result.push({ x, y: this.h });
      }
      return result;
    }
  },
  methods: {
    keyup(e) {
      if (e.code === this.codes.left) {
        e.preventDefault();
        this.moveLeft();
      } else if (e.code === this.codes.right) {
        e.preventDefault();
        this.moveRight();
      } else if (e.code === this.codes.down) {
        e.preventDefault();
        this.moveDown();
      } else if (e.code === this.codes.instantDrop) {
        e.preventDefault();
        this.instantDrop();
      } else if (e.code === this.codes.rotate) {
        e.preventDefault();
        this.rotate();
      }
    },
    pause() {
      clearInterval(this.intervalId);
      this.intervalId = null;
    },
    resume() {
      this.intervalId = setInterval(this.moveDown, this.delay);
    },
    restart() {
      this.droppedPieces = [];
      this.getNewBag();
      this.fallingPiece = null;
      this.completedLines = 0;

      this.spawnNextPiece()
      if(!this.intervalId){
        this.resume()
      }
    },
    getNewBag(){
      this.bag = [0,1,2,3,4,5,6].map(x=> {return {x, r: Math.random()}}).sort((a,b) => {return a.r <= b.r ? -1 : 1 }).map(o => o.x)
    },
    spawnNextPiece(){
      if(this.bag.length == 0){
        this.getNewBag()
      }
      this.spawnNewPiece(this.bag.shift())
    },
    spawnNewPiece(basePiece) {
      this.fallingPiece = {
        id: new Date().valueOf(),
        basePiece,
        x: 3,
        y: -1,
        rotation: 0
      };
    },
    pieceCopy(piece) {
      return {
        id: piece.id,
        basePiece: piece.basePiece,
        x: piece.x,
        y: piece.y
      };
    },
    moveLeft() {
      if (this.checkLeft()) {
        this.fallingPiece.x--;
      }
    },
    moveRight() {
      if (this.checkRight()) {
        this.fallingPiece.x++;
      }
    },
    moveDown() {
      if (this.checkDown()) {
        this.fallingPiece.y++;
      } else {
        this.pieceDropped();
      }
    },
    rotate() {
      if (this.checkRotate()) {
        this.fallingPiece.rotation = (this.fallingPiece.rotation + 1) % 4;
      }
    },
    checkRotate() {
      return true;
    },
    instantDrop() {
      do {
        this.moveDown();
      } while (this.checkDown());
      this.pieceDropped();
    },
    pieceDropped() {
      var fallingCells = this.getCells(this.fallingPiece);
      for (var i = 0; i < fallingCells.length; i++) {
        this.droppedPieces.push({
          x: fallingCells[i].x,
          y: fallingCells[i].y,
          basePiece: this.fallingPiece.basePiece
        });
      }
      this.checkCompletedRow();
      this.spawnNextPiece();
    },
    checkCompletedRow() {
      for (var y = this.h - 1; y >= 0; y--) {
        var piecesDroppedOnline = this.droppedPieces.filter(p => p.y === y);
        if (piecesDroppedOnline.length === this.w) {
          this.completedLines++;
          // this.droppedPieces = this.droppedPieces.map(p => {
          //   if (p.y === y) {
          //     p.basePiece = 7;
          //   }
          //   return p;
          // });
          this.droppedPieces = this.droppedPieces.filter(p => p.y !== y);
          this.droppedPieces = this.droppedPieces.map(p => {
            if (p.y < y) {
              p.y++;
              y++;
            }
            return p;
          });
        }
      }
    },
    checkLeft() {
      var pieceCells = this.getCells(this.fallingPiece);
      var dropped = this.getDroppedCells();
      var obs = dropped.concat(this.leftWall);
      var pieceCellsShiftedBy = this.newCellsShiftedBy(pieceCells, -1);
      return !this.isOverlappingCells(obs, pieceCellsShiftedBy);
    },
    checkRight() {
      var pieceCells = this.getCells(this.fallingPiece);
      var dropped = this.getDroppedCells();
      var obs = dropped.concat(this.rightWall);
      var pieceCellsShiftedBy = this.newCellsShiftedBy(pieceCells, 1);
      return !this.isOverlappingCells(obs, pieceCellsShiftedBy);
    },
    checkDown() {
      var pieceCells = this.getCells(this.fallingPiece);
      var dropped = this.getDroppedCells();
      var obs = dropped.concat(this.floor);
      var pieceCellsDroppedBy = this.newCellsDroppedBy(pieceCells, 1);
      return !this.isOverlappingCells(obs, pieceCellsDroppedBy);
    },
    newCellsDroppedBy(cells, n) {
      return cells.map(cell => {
        return { x: cell.x, y: cell.y + n };
      });
    },
    newCellsShiftedBy(cells, n) {
      return cells.map(cell => {
        return { x: cell.x + n, y: cell.y };
      });
    },
    distinct(v, i, s) {
      return s.indexOf(v) === i;
    },
    isOverlapping(cell1, cell2) {
      return cell1.x === cell2.x && cell1.y === cell2.y;
    },
    isOverlappingCells(cells1, cells2) {
      for (var i = 0; i < cells1.length; i++) {
        for (var j = 0; j < cells2.length; j++) {
          if (this.isOverlapping(cells1[i], cells2[j])) {
            return true;
          }
        }
      }
      return false;
    },
    getDroppedCells() {
      return this.droppedPieces;
    },
    getCells(piece) {
      return this.basePieces[piece.basePiece][piece.rotation].map(base => {
        return { x: piece.x + base.x, y: piece.y + base.y };
      });
    }
  }
};
</script>

<style>
rect {
  fill: rgba(1, 1, 46, 0.8);
}
.cell {
  stroke-width: 1px;
  stroke: aliceblue;
  stroke-opacity: 0.15;
}
.pieceCell {
  stroke: #eee;
  opacity: 0.8;
}
.type0 {
  fill: aqua;
}
.type1 {
  fill: blue;
}
.type2 {
  fill: yellow;
}
.type3 {
  fill: orange;
}
.type4 {
  fill: red;
}
.type5 {
  fill: green;
}
.type6 {
  fill: violet;
}
.type7 {
  fill: gray;
}
</style>