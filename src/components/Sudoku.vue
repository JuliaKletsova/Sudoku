<template>
    <div class="sudoku">
        <div class="title">Sudoku</div>
        <div class="btn_div">
            <button class="btn" @click="generatePuzzle">Restart</button>
            <button class="btn" @click="resetPuzzle">Reset</button>
        </div>   
        <select v-model="difficulty" @change="generatePuzzle">
            <option v-for="(value, ind) in difficulties" :key="ind" :value="value">{{ value }}</option>
        </select>     
        <div class="grid">
            <div class="row" v-for="(row, rowInd) in puzzle" :key="rowInd">
                <div class="cell" :class="{
                        'right-border': colInd == 2 || colInd == 5,
                        'bottom-border': rowInd === 2 || rowInd === 5,
                        'origin': cell.original,
                        'active': activeRow === rowInd && activeCol === colInd,
                        'invalid': cell.value && isCellInvalid(rowInd, colInd, cell.value)
                    }"
                     v-for="(cell, colInd) in row" :key="colInd"
                     @click="setCellActive(rowInd, colInd, cell.original)"
                    >{{ cell.value }}</div>
            </div>
        </div>
        <div  v-if="showNums" class="row">
            <button
                type="button"
                class="btn"
                v-for="value in numbers" :key="value"
                @click="setCellValue(value)"
            >
                {{ value }}
            </button>
        </div>
    </div>
</template>

<script>
import { sudoku } from "sudoku.js/sudoku.js"

export default {
    name: "sudoku",
    data: () => ({
        difficulty: null,
        difficulties: {
            'easy' :'Easy',
            'medium': "Medium",
            'hard': "Hard",
            'very-hard': 'Very Hard',
            'insane': "Insane",
            'inhuman': "Inhuman"
        },
        puzzle: [],
        activeRow: -1,
        activeCol: -1,
        numbers: [1,2,3,4,5,6,7,8,9],
        showNums: false,
        isCellValid: null
    }),
    methods: {
        getData(difficulty) {
            const boardString = sudoku.generate(difficulty)
            this.puzzle = sudoku.board_string_to_grid(boardString)
            .map(row => {
                return row.map(cell => {
                    return {
                        value: cell !== "." ? parseInt(cell): null,
                        original: cell !== "."
                    }
                })
            })
        },
        generatePuzzle(e) {
            document.getElementsByClassName('sudoku')[0].style.color = "black"
            if(e.type === "change" && e.target.value) {
                this.getData(e.target.value)
            }
            if(this.difficulty && e.type === "click") {
                this.getData(this.difficulty)
            }
        },
        setCellActive(row, col, original) {
            if(original) {
                return
            }
            if(this.activeRow === row && this.activeCol === col) {
                this.activeRow = -1
                this.activeCol = -1
                this.showNums = false
                return
            }
            this.activeRow = row
            this.activeCol = col
            this.showNums = true
            console.log('active cell', row, col)
        },
        setCellValue (value) {
            this.isCellValide = this.isCellInvalid(this.activeRow, this.activeCol, value)
            this.puzzle[this.activeRow][this.activeCol].value = value
            this.activeRow = -1
            this.activeCol = -1
            this.showNums = false
            if (this.isGameComplete()) {
                document.getElementsByClassName('sudoku')[0].style.color = "green"
            }
            
        },
        isCellInvalid (row, col, value) {
            if (!value) {
                return true
            }
            for (let c=0; c<9; c++) {
                if (this.puzzle[row][c].value === value && c !== col) {
                return true
                }
            }
            for (let r=0; r<9; r++) {
                if (this.puzzle[r][col].value === value && r !== row) {
                return true
                }
            }
            const rowStart = Math.floor(row / 3) * 3
            const colStart = Math.floor(col / 3) * 3
            for (let r=rowStart; r<rowStart+3; r++) {
                for (let c=colStart; c<colStart+3; c++) {
                    if (this.puzzle[r][c].value === value && !(r === row && c === col)) {
                        return true
                    }
                }
            }
            return false
        },
        isGameComplete () {
            for (let r=0; r<9; r++) {
                for (let c=0; c<9; c++) {
                    if (this.isCellInvalid(r, c, this.puzzle[r][c].value)) {
                        return false
                    }
                }
            }
            return true
        },
        resetPuzzle() {
            for(let r=0; r<9; r++) {
                for(let c=0; c<9; c++) {
                    if(this.puzzle[r][c].original === false)
                        this.puzzle[r][c].value = ""
                }
            }
            return
        }
    },
    mounted() {
        this.generatePuzzle('easy')
    }
}
</script>

<style scoped>
    .sudoku {
        width: 100vw;
        background: tomato;
        font-family: Arial, Helvetica, sans-serif;
    }
    .title {
        padding-top: 40px;
        width: 100vw;
        text-align: center;
        color: white;
        font-size: 40px;
    }
    .btn_div {
        display: flex;
        justify-content: space-around;
    }
    .btn {
        margin: 15px 0 0 0;
        font-size: 16px;
        width: 150px; 
        height: 30px;
    }
    .grid {
        width: calc(9*40px);
        position: absolute;
        top: 200px;
        left: 185px;
    }
    .row {
        display: flex;
        align-items: center;
        justify-content: center;
        background-color: none;
    }
    .cell {
        display: block;
        width: 40px;
        height: 40px;
        box-sizing: border-box;
        border: 1px solid #bbb;
        line-height: 40px;
        font-size: 24px;
        text-align: center;
        background-color: white;
    }
    .cell.right-border { border-right-width: 3px; }
    .cell.bottom-border { border-bottom-width: 3px; }
    .cell.original { font-weight: bold; }
    .cell:not(.original) { cursor: pointer; }
    .cell.active { border: 1px solid red; }
    .cell.invalid { color: red}
    @media screen and (max-width: 460px) {
        .sudoku {
            height: 600px;
        }

    }
    @media screen and (max-width: 461px) and (max-width: 800px){
        .sudoku {
            height: 900px;
        }
    }
    @media screen and (max-width: 801px) {
        .sudoku {
            height: 1300px;
        }
    }
</style>