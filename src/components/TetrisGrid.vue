<template>
  <div>
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
      <template v-for="piece in pieces">
        <rect
          v-for="(cell, i) in basePieces[piece.basePiece]"
          :key="'piece_' + piece.id + '_cell_' + i"
          :x="(piece.x+cell.x)*cellSize"
          :y="(cell.y+piece.y)*cellSize"
          :width="cellSize"
          :height="cellSize"
          :class="'pieceCell type' + piece.basePiece"
        />
      </template>
    </svg>
  </div>
</template>

<script>
export default {
  props: { w: { type: Number }, h: { type: Number } },
  data() {
    return {
      codes: {
        left: "ArrowLeft",
        right: "ArrowRight",
        down: "ArrowDown",
        instantDrop: "Space"
      },
      delay: 2000,
      cellSize: 30,
      basePieces: [
        [{ x: 0, y: 0 }, { x: 0, y: 1 }, { x: 0, y: 2 }, { x: 0, y: 3 }],
        [{ x: 0, y: 0 }, { x: 1, y: 0 }, { x: 0, y: 1 }, { x: 0, y: 2 }],
        [{ x: 0, y: 0 }, { x: 1, y: 0 }, { x: 0, y: 1 }, { x: 1, y: 1 }],
        [{ x: 0, y: 0 }, { x: 1, y: 0 }, { x: 1, y: 1 }, { x: 1, y: 2 }],
        [{ x: 0, y: 0 }, { x: 1, y: 0 }, { x: 1, y: 1 }, { x: 2, y: 1 }],
        [{ x: 1, y: 0 }, { x: 2, y: 0 }, { x: 0, y: 1 }, { x: 1, y: 1 }],
        [{ x: 1, y: 0 }, { x: 0, y: 1 }, { x: 1, y: 1 }, { x: 2, y: 1 }]
      ],
      activePieceIndex: 0,
      pieces: []
    };
  },
  created() {
    window.addEventListener("keydown", e => this.keyup(e));
    this.spawnNewPiece();
    setInterval(this.moveDown, this.delay);
  },
  computed: {
    activePiece() {
      return this.pieces[this.activePieceIndex];
    },
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
    keyup(key) {
      if (key.code === this.codes.left) {
        this.moveLeft();
      } else if (key.code === this.codes.right) {
        this.moveRight();
      } else if (key.code === this.codes.down) {
        this.moveDown();
      } else if (key.code === this.codes.instantDrop) {
        this.instantDrop();
      }
    },
    spawnNewPiece() {
      this.pieces.push({
        id: new Date().valueOf(),
        basePiece: Math.floor(Math.random() * this.basePieces.length),
        x: 3,
        y: -1
      });
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
        var copy = this.pieceCopy(this.activePiece);
        copy.x--;
        this.pieces.splice(this.activePieceIndex, 1, copy);
      }
    },
    moveRight() {
      if (this.checkRight()) {
        var copy = this.pieceCopy(this.activePiece);
        copy.x++;
        this.pieces.splice(this.activePieceIndex, 1, copy);
      }
    },
    moveDown() {
      if (this.checkDown()) {
        var copy = this.pieceCopy(this.activePiece);
        copy.y++;
        this.pieces.splice(this.activePieceIndex, 1, copy);
      } else {
        this.pieceDropped();
      }
    },
    instantDrop() {
      do {
        this.moveDown();
      } while (this.checkDown());
      this.pieceDropped();
    },
    pieceDropped() {
      this.spawnNewPiece();
      this.activePieceIndex++;
    },
    checkLeft() {
      var pieceCells = this.getCells(this.activePiece);
      var dropped = this.getDroppedCells();
      var obs = dropped.concat(this.leftWall);
      var pieceCellsShiftedBy = this.newCellsShiftedBy(pieceCells, -1);
      return !this.isOverlappingCells(obs, pieceCellsShiftedBy);
    },
    checkRight() {
      var pieceCells = this.getCells(this.activePiece);
      var dropped = this.getDroppedCells();
      var obs = dropped.concat(this.rightWall);
      var pieceCellsShiftedBy = this.newCellsShiftedBy(pieceCells, 1);
      return !this.isOverlappingCells(obs, pieceCellsShiftedBy);
    },
    checkDown() {
      var pieceCells = this.getCells(this.activePiece);
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
      return this.pieces
        .slice(0, this.activePieceIndex)
        .map(p => this.getCells(p))
        .flat();
    },
    getCells(piece) {
      return this.basePieces[piece.basePiece].map(base => {
        return { x: piece.x + base.x, y: piece.y + base.y };
      });
    }
  }
};
</script>

<style scoped>
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
  fill: orange;
}
.type2 {
  fill: yellow;
}
.type3 {
  fill: blue;
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
</style>