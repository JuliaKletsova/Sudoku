<template>
    <div id="app">
        <Sudoku />
    </div>
</template>

<script>
import Sudoku from './components/Sudoku.vue'

export default {
    name: 'App',
    components: {
        Sudoku
    }
}
</script>

<style scoped>
    #app {
        width: 100%;
        min-width: 345px;
        margin: 0;
        padding: 0;
    }
</style>