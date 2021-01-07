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
        height: 100wh;
        overflow-y: scroll;
        font-family: Arial, Helvetica, sans-serif;
    }
    .cell:not(.original) { color:blue; }
    .cell.active { border: 1px solid red; }
    .cell.invalid { color: red; }
    .success { color: green; }
    .cell.original { font-weight: bold;}
    .cell.right-border { border-right-width: 3px; }
    .cell.bottom-border { border-bottom-width: 3px; }
    .selector {
        float: right;
        border: 1px solid #bbb;
        border-radius: 5px;
    }
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
    .btn_number {
        color:blue;
        font-weight: normal;
        border: 1px solid #bbb;
    }
    .btn,
    .btn_number {
        box-sizing: border-box;
        background: none;
        border: none;
        width: 60px;
        height: 45px;
    }
    .btn:focus,
    .selector:focus {
        outline: none;
        border: 1px solid #bbb;
        border-radius: 5px;
    }
    @media screen and (max-width: 426px) {     /* Стили мобильных устройств */
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
            margin: 2px 0 0 0;
        }
        .selector {
            margin: 15px 15px 0 0;
        }
        .numbers_row {
            margin-top: 380px;
            width: 99%;
            display:flex;
            flex-direction: row;
            justify-content: space-around;
            align-items: center;
        }
        .btn_number {
            width: 1.95rem;
            height: 2rem;
            font-size: 22px;
            margin: 0 0 0 3.55px;
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
    }
    @media screen and (min-width: 427px) and (max-width: 520px) { 
        .grid {
            position: absolute;
            top: 60px;
            left: 2rem;
            right: 2rem;
        }
        .cell {
            width: 50px;
            height: 40px;
            line-height: 40px;
            font-size: 28px;
        }
        .btn:first-of-type {
            margin: 0.5rem 0 0 1.5rem;
        }
        .btn:focus,
        .selector:focus {
            outline: none;
            border: 1px solid #bbb;
            border-radius: 5px;
        }
        .selector {
            height: 2rem;
            margin: 1rem 2rem 0 0;
        }
        .numbers_row {
            margin-top: 380px;
            width: 99%;
            display:flex;
            flex-direction: row;
            justify-content: space-around;
            align-items: center;
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
    }
    @media screen and (min-width: 521px) {
        .grid {
            position: absolute;
            top: 90px;
            left: 2rem;
            right: 2rem;
        }
        .cell {
            width: 60px;
            height: 60px;
            line-height: 60px;
            font-size: 36px;
        }
        .btn {
            font-size: 22px;
            margin-left: 1rem;
        }
        .btn:first-of-type {
            margin: 1rem 0 0 2rem;
        }
        .btn:focus,
        .selector:focus {
            outline: none;
            border: 1px solid #bbb;
            border-radius: 5px;
        }
        .selector {
            height: 2.5rem;
            width: 10rem;
            font-size: 22px;
            margin: 1.5rem 2.2rem 0 0;
        }
        .numbers_row {
            margin-top: 600px;
            width: 99%;
            max-width: 500px;
            display:flex;
            flex-direction: row;
            justify-content: space-around;
            align-self: center;
            align-items: center;
        }
        .btn_number {
            color:blue;
            font-weight: normal;
            border: 1px solid #bbb;
            width: 2.5rem;
            height: 2.5rem;
            font-size: 25px;
            margin: 0 0 0 3.55px;
        }
        .btn_number {
            margin: 0 0 0 2.58%;
        }
        .btn_number:first-child {
            margin-left: 40vw;
        }
        @media screen and (min-width: 521px) and (max-width: 560px)  {
            .btn_number:first-child {
                margin-left: 5vw;
            }
        }
        @media screen and (min-width: 561px) and (max-width: 660px)  {
            .btn_number:first-child {
                margin-left: 15vw;
            }
        }
        @media screen and (min-width: 661px) and (max-width: 760px)  {
            .btn_number:first-child {
                margin-left: 25vw;
            }
        }
        @media screen and (min-width: 761px) and (max-width: 860px)  {
            .btn_number:first-child {
                margin-left: 33vw;
            }
        }
        @media screen and (min-width: 861px) and (max-width: 930px)  {
            .btn_number:first-child {
                margin-left: 43vw;
            }
        }
        @media screen and (min-width: 931px) and (max-width: 1000px)  {
            .btn_number:first-child {
                margin-left: 50vw;
            }
        }
        @media screen and (min-width: 1001px) and (max-width: 1200px)  {
            .btn_number:first-child {
                margin-left: 55vw;
            }
        }
        @media screen and (min-width: 1201px) and (max-width: 1400px)  {
            .btn_number:first-child {
                margin-left: 59vw;
            }
        }
        @media screen and (min-width: 1401px) and (max-width: 1550px)  {
            .btn_number:first-child {
                margin-left: 65vw;
            }
        }
        @media screen and (min-width: 1551px) and (max-width: 1800px)  {
            .btn_number:first-child {
                margin-left: 70vw;
            }
        }
        @media screen and (min-width: 1801px) and (max-width: 2100px) {
            .btn_number:first-child {
                margin-left: 75vw;
            }
        }
        @media screen and (min-width: 2101px) {
            .btn_number:first-child {
                margin-left: 77vw;
            }
        }
    }
</style>