<template>
    <div class="sudoku">
        <div class="btn_div">
            <button class="btn" @click="generatePuzzle">New<br>Game</button>
            <button class="btn" @click="resetPuzzle">&#8635;<br>Clear</button>
            <select id="selector"
                    class="selector"
                    v-model="difficulty"
            ><option v-for="(value, ind) in difficulties" 
                     :key="ind" 
                     :value="value"
                     :id="ind"
                >{{ value }}</option>
            </select>  
        </div>   
        <div class="grid">
            <div class="row" v-for="(row, rowInd) in puzzle" :key="rowInd">
                <div class="cell" :class="{
                        'right-border': colInd == 2 || colInd == 5,
                        'bottom-border': rowInd === 2 || rowInd === 5,
                        'original': cell.original,
                        'active': activeRow === rowInd && activeCol === colInd,
                        'invalid': cell.value && isCellInvalid(rowInd, colInd, cell.value)
                    }"
                     v-for="(cell, colInd) in row" :key="colInd"
                     @click="setCellActive(rowInd, colInd, cell.original)"
                    >{{ cell.value }}</div>
            </div>
        </div>
        <div v-if="showNums" class="numbers_row">
            <button
                type="button"
                class="btn_number"
                v-for="value in numbers" 
                :key="value"
                :id="value"
                @click="setCellValue(value)"
            >{{ value }}
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
                        original: cell !== "." && cell !== null
                    }
                })
            })
        },
        generatePuzzle(e) {
            document.getElementsByClassName('sudoku')[0].style.color = "black"
            if(e.type === "change" && e.target.value) {
                this.getData(e.target.value)
                return true
            }
            if(this.difficulty && e.type === "click") {
                this.getData(this.difficulty)
                return true
            }
            if(this.difficulties[e]) {
                this.getData(e)
                return true
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
            const rowStart = Math.floor(row / 3) * 3,
                  colStart = Math.floor(col / 3) * 3
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
                        this.puzzle[r][c].original = true;
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
        const diffs = ['easy', 'medium', 'high', 'very-high', 'insane', 'inhuman'],
              random = Math.floor(Math.random() * diffs.length);

        this.generatePuzzle(diffs[random])
    }
}
</script>

<style scoped>
    .sudoku { 
        font-family: Arial, Helvetica, sans-serif; 
    }
    .cell:not(.original) { color:blue; }
    .cell.active { border: 1px solid red; }
    .cell.invalid { color: red; }
    .success { color: green; }
    .cell.original { font-weight: bold;}
    .cell.right-border { border-right-width: 3px; }
    .cell.bottom-border { border-bottom-width: 3px; }

    .title {
        font-size: 40px;
        padding-top: 40px;
        width: 100vw;
        text-align: center;
        color: white;
    }
    .row {
        display: flex;
        align-items: center;
        justify-content: center;
        background-color: none;
    }
    .cell {
        display: block;
        box-sizing: border-box;
        border: 1px solid #bbb;
        text-align: center;
        background-color: white;
    }
    @media screen and (max-width: 426px) {     /* Стили мобильных устройств */
        .sudoku {
            height: 100wh;
            overflow-y: hidden;
        }
        .grid {
            position: absolute;
            top: 60px;
            left: 5px;
            right: 5px;
        }
        .cell {
            width: 40px;
            height: 40px;
            line-height: 40px;
            font-size: 24px;
        }
        .btn,
        .btn_number {
            box-sizing: border-box;
            background: none;
            border: none;
            width: 60px;
            height: 45px;
            margin: 2px 0 0 0;
        }
        .btn:focus,
        .selector:focus {
            outline: none;
            border: 1px solid #bbb;
            border-radius: 5px;
        }
        .selector {
            float: right;
            margin: 15px 15px 0 0;
            border: 1px solid #bbb;
            border-radius: 5px;
        }
        .numbers_row {
            margin-top: 380px;
        }
        .btn_number {
            color:blue;
            font-weight: normal;
            border: 1px solid #bbb;
            width: 1.95rem;
            height: 2rem;
            font-size: 22px;
            margin: 0 0 0 3.55px;
        }
        @media screen and (max-width:320px) {
            .btn_number {
                margin-left: 5px;
            }
        }
        @media screen and (min-width: 321px) and (max-width: 375px) {
            .btn_number {
                margin: 0 0 0 1%;
            }
            .btn_number:first-child {
                margin-left: 5%;
            }
        }
        @media screen and (min-width: 376px) and (max-width: 390px) {
            .btn_number {
                margin: 0 0 0 2.58%;
            }
            .btn_number:first-child {
                margin-left: 2.5%;
            }
        }
        @media screen and (min-width: 391px) and (max-width: 426px) {
            .btn_number {
                margin: 0 0 0 2.4%;
            }
            .btn_number:first-child {
                margin-left: 5%;
            }
        }
        
    }
    
</style>