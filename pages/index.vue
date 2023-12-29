<template>
  <div>
    <h1>欢迎来到数独游戏</h1>
    <SudokuBoard />
  </div>
</template>

<script>
import SudokuBoard from '~/components/SudokuBoard'

export default {
  components: {
    SudokuBoard
  }
}
</script>
