<template>
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
</template>
<script>
export default {
    data: () => ({
        showNums: true,
        numbers: [1,2,3,4,5,6,7,8,9]
    })
}
</script>

<style>
    .numbers_row {
        width: 400px;
        display:flex;
        flex-direction: row;
        justify-content: space-around;
        align-items: center;
    }
    .btn_number {}
</style>