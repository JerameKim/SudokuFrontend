<template>
  <div class="sudoku">
    <div class="topRow">
      <h1>Sudoku</h1>
      <select v-model="difficulty" name="variations" id="variations" @change="grabPuzzle(difficulty)">
        <option v-for="level in levels" :value="level" :key="level">{{level}}</option>
      </select>
      <button @click="verifySolution()">Verify Puzzle</button>
    </div>
    
    <div class="buttonRow">
      <button @click="clearCell(selectedRow, selectedCol)" :disabled="btnDisabled">Erase Cell</button>
      <button class="buttonValue" @click="markCell(buttonVal, selectedRow, selectedCol)" v-for="(buttonVal, idx) in buttonValues" v-bind:key="idx" :disabled="btnDisabled">
        {{ buttonVal }}</button>
    </div>
    <div class="grid">
      <div class="row" v-for="(row,rowIdx) in puzzle" v-bind:key="rowIdx">
        <div class="val" 
        :class="{
          'border-bottom': rowIdx === 2 || rowIdx === 5, 
          'border-right': colIdx === 2 || colIdx === 5,
          'is-selected': selectedRow === rowIdx && selectedCol === colIdx,
          }"
        v-for="(cellObj, colIdx) in row" v-bind:key="colIdx"
        @click="selectSquare(cellObj, rowIdx, colIdx)">
          {{ cellObj.value }}
        </div>
      </div>
    </div>
    <div class="result" v-if="showResult">
      <h3 v-if="result">Your solution is correct :)</h3>
      <h3 v-else>Your solution is wrong</h3>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Grid', 
    data() { 
      return {
        puzzle: [], 
        rawPuzzle: [],
        refPuzzle: [],
        buttonValues: [1,2,3,4,5,6,7,8,9],
        selectedRow: -1, 
        selectedCol: -1,
        btnDisabled: true,
        showResult: false,
        result: true,
        difficulty: "Very Easy",
        levels: ["Very Easy", "Medium", "Nearly Impossible"],
      }
    }, 
    mounted() {  // When app loads, make a puzzle
      this.grabPuzzle(this.difficulty)
    },
    methods: {
      selectSquare(cellObj, rowIdx, colIdx) { // highlight selected cell
        console.log(cellObj.value, cellObj.editable, rowIdx, colIdx)
        if (cellObj.editable === true) { // empty square, is able to be changed
          cellObj.isSelected = true 
          this.selectedRow = rowIdx 
          this.selectedCol = colIdx
          this.btnDisabled = false
        }
        else { 
          this.btnDisabled = true
        }
      }, 
      grabPuzzle(difficulty){ // fetch request and format puzzle 
        this.showResult = false  // Every time we need a new puzzle, hide the solution 
        if(difficulty == "Very Easy") {
          fetch("http://127.0.0.1:5000/puzzle")
          .then(response => response.json())
          .then( data => 
          { 
            this.rawPuzzle = data
            for(var row = 0; row < 9; row += 1) { 
              var myRow = []
              for(var col = 0; col < 9; col += 1 ) {
                var newValue = this.rawPuzzle.shift()
                myRow.push(newValue)
              }
              this.rawPuzzle.push(myRow)
            }
            this.refPuzzle = this.rawPuzzle
            console.log(this.rawPuzzle)
            this.makePuzzle()
          })
        }
        if(difficulty == "Medium") {
          fetch("http://127.0.0.1:5000/puzzle2")
          .then(response => response.json())
          .then( data => 
          { 
            this.rawPuzzle = data
            for(var row = 0; row < 9; row += 1) { 
              var myRow = []
              for(var col = 0; col < 9; col += 1 ) {
                var newValue = this.rawPuzzle.shift()
                myRow.push(newValue)
              }
              this.rawPuzzle.push(myRow)
            }
            this.refPuzzle = this.rawPuzzle
            console.log(this.rawPuzzle)
            this.makePuzzle()
          })
        }
        if(difficulty == "Nearly Impossible") {
          fetch("http://127.0.0.1:5000/puzzle3")
          .then(response => response.json())
          .then( data => 
          { 
            this.rawPuzzle = data
            for(var row = 0; row < 9; row += 1) { 
              var myRow = []
              for(var col = 0; col < 9; col += 1 ) {
                var newValue = this.rawPuzzle.shift()
                myRow.push(newValue)
              }
              this.rawPuzzle.push(myRow)
            }
            this.refPuzzle = this.rawPuzzle
            console.log(this.rawPuzzle)
            this.makePuzzle()
          })
        }
      },
      makePuzzle(){  // Format the puzzle to be made 
        var myMatrix = []
        for(var row = 0; row < 9; row +=1){ 
          var myRow = []
          for(var col = 0; col< 9; col += 1){
            const squareValue = this.refPuzzle[row][col]
            var isEditable = false
            if(squareValue === ""){
              isEditable = true
            }
            const obj = { 
              value: squareValue, 
              editable: isEditable, 
              isSelected: false
            }
            myRow.push(obj)
          }
          myMatrix.push(myRow)
        } 
        this.puzzle = myMatrix
      },
      markCell(buttonVal, rowIdx, colIdx){  // update the cell at rowIdx,colIdx with the assmed value
        this.puzzle[rowIdx][colIdx].value = buttonVal // update the cell
      }, 
      clearCell(rowIdx, colIdx){  // sets an editable cell to empty
        this.puzzle[rowIdx][colIdx].value = "" 
      }, 
      verifySolution() { // makes a POST request to verify solution
        var board_total = []
            // access cells in grid using this.puzzle[row][col]
            for(let r = 0; r < 9; r += 1){
                for(let c = 0; c<9; c += 1){
                    if (this.puzzle[r][c].value === null) { 
                        board_total.push(0)
                    }
                    else {
                        board_total.push(this.puzzle[r][c].value)
                    }
                }
            }
            // Post data for verification
            const requestOptions = { 
                method: "POST", 
                headers: {"Content-Type": "application/json"},
                body: JSON.stringify({values: board_total})
            }; 
            fetch("http://127.0.0.1:5000/puzzle", requestOptions)
                .then(response => response.json())
                .then(data => {
                  this.solutionAlert(data)
                  this.showResult = true})
      },
      solutionAlert(result) {  // alert/change solution
        console.log(result)
        this.result = result
      },
    }, 
}
</script>

<style scoped>
button { 
  background-color: #ffd3b6
}
select { 
  background-color: #ffd3b6
}
.sudoku{
  width: 100%; max-width: 420px; 
  margin: 0.5rem auto;
  font-family: Arial, Helvetica, sans-serif; 
  background-color: #f4eeff
}
.topRow { 
  display: flex; 
  align-items: center; 
  justify-content: space-between; 
}
.buttonRow { 
  display: flex;
  align-items: center; 
  justify-content: space-between; 
  padding-bottom: 14px;
}
.buttonValue{ 
  height: 30px;
  font-weight: bolder;
  font-size: 16pt;
}
.row { 
  display: flex; 
  justify-content: center;
}
.val { 
  cursor: pointer;
  display: flex; 
  font-weight: bold;
  padding: 10px;
  width: 20px;
  justify-content: center;
  border: 2px solid #ccc; 
}
.val.border-bottom{ 
  border-bottom-color: black;
  border-bottom-width: 3px;
}
.val.border-right{ 
  border-right-color: black;
  border-right-width: 3px;
}

.val.is-selected{
  background-color: #a6b1e1;
} 
</style>
