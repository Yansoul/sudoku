<template>
  <div class="sudoku-container">
    <div class="button-container">
      <button class="generate-button" @click="generateSudoku">
        出题
      </button>
    </div>
    <div class="sudoku-board">
      <div v-for="(row, rowIndex) in board" :key="rowIndex" class="sudoku-row">
        <div v-for="(cell, colIndex) in row" :key="colIndex" class="sudoku-cell">
          <input
            v-model.number="board[rowIndex][colIndex]"
            type="number"
            min="1"
            max="9"
            @input="handleInput(rowIndex, colIndex)"
          >
        </div>
      </div>
    </div>
    <!-- 错误信息显示，当 validationError 为 true 时显示 -->
    <div v-if="validationError" class="error-message">
      输入无效，请重新输入。
    </div>
  </div>
</template>

<script>
import sudoku from 'sudoku'

export default {
  data () {
    return {
      board: [],
      validationError: false // 用于存储验证状态
    }
  },
  created () {
    this.generateSudoku()
  },
  methods: {
    // 验证输入的数独数字是否合法
    handleInput (rowIndex, colIndex) {
      // 检查值是否为1到9之间的数字
      const value = this.board[rowIndex][colIndex]
      if (value && (value < 1 || value > 9)) {
        this.board[rowIndex][colIndex] = null
        this.validationError = true
        return
      }

      // 验证是否发生重复数字
      if (!this.validateCell(rowIndex, colIndex)) {
        this.validationError = true
      } else {
        this.validationError = false
      }

      // 检查是否所有格子都已填满
      if (this.isBoardFull()) {
        if (this.validateSudoku()) {
          alert('恭喜，你成功解出了数独！')
        } else {
          alert('答案不正确，请再检查一下。')
        }
      }
    },
    // 验证特定单元格所在的行、列和宫
    validateCell (rowIndex, colIndex) {
      return (
        this.isValidGroup(this.getRow(rowIndex)) &&
      this.isValidGroup(this.getColumn(colIndex)) &&
      this.isValidGroup(this.getBox(Math.floor(rowIndex / 3) * 3 + Math.floor(colIndex / 3)))
      )
    },
    // 生成数独游戏板
    generateSudoku () {
      // 生成一个新的数独游戏板
      const rawBoard = sudoku.makepuzzle()
      const solvedBoard = sudoku.solvepuzzle(rawBoard) // 完整解决的数独
      const revealedCells = 45 // 调整这个值来控制难度（更多的数字表示更低的难度）

      for (let i = 0; i < rawBoard.length; i++) {
        if (Math.random() > revealedCells / 81) {
          // 随机决定是否保留每个单元格的数字
          rawBoard[i] = null
        } else {
          // 保留数字
          rawBoard[i] = solvedBoard[i] + 1
        }
      }

      // 转换格式以适应你的应用
      this.board = this.transformBoard(rawBoard)
    },
    // 将 rawBoard 转换为 9x9 的二维数组
    transformBoard (rawBoard) {
      // 'null' 代表空白格
      const board = []
      for (let i = 0; i < 9; i++) {
        board[i] = rawBoard.slice(i * 9, (i + 1) * 9).map(v => (v === null ? null : v + 1))
      }
      return board
    },
    // 获取指定索引的行
    getRow (rowIndex) {
      return this.board[rowIndex]
    },
    // 获取指定索引的列
    getColumn (colIndex) {
      return this.board.map(row => row[colIndex])
    },
    getBox (boxIndex) {
      // 计算3x3宫格的起始行和列
      const startRow = Math.floor(boxIndex / 3) * 3
      const startCol = (boxIndex % 3) * 3
      const box = []
      // 遍历宫格内的每个单元格并添加到宫格数组
      for (let row = startRow; row < startRow + 3; row++) {
        for (let col = startCol; col < startCol + 3; col++) {
          box.push(this.board[row][col])
        }
      }

      // 返回宫格内的单元格数组
      return box
    },
    // 校验数独解答结果正确性
    validateSudoku () {
    // 遍历数独的每一行、每一列和每个3x3宫格
      for (let i = 0; i < 9; i++) {
        // 如果任何行、列或宫格无效，则数独无效
        if (!this.isValidGroup(this.getRow(i)) ||
          !this.isValidGroup(this.getColumn(i)) ||
          !this.isValidGroup(this.getBox(i))) {
          // 返回 false 表示数独解答不正确
          return false
        }
      }

      // 所有行、列和宫格都有效，返回 true 表示数独解答正确
      return true
    },
    // 验证数独特定group的正确性
    isValidGroup (group) {
      // 过滤掉所有null值
      const filteredGroup = group.filter(value => value !== null)
      // 检查过滤后的数组是否没有重复（即 Set 的大小与数组长度相同）
      return new Set(filteredGroup).size === filteredGroup.length
    },
    isBoardFull () {
      return this.board.every(row => row.every(cell => cell !== null && cell !== 0))
    }
  }

}
</script>

<style>
.sudoku-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.button-container {
  margin-bottom: 10px;
}
.generate-button {
  padding: 5px 10px; /* 调整按钮的填充来改变大小 */
  font-size: 16px; /* 调整字体大小 */
  cursor: pointer; /* 鼠标悬停时显示指针 */
}
.sudoku-board {
  display: grid;
  grid-template-columns: repeat(9, 1fr);
  grid-gap: 1px;
  max-width: 500px;
}
.sudoku-cell {
  border: 1px solid #ddd;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 55px; /*
   保持宽高一致 */
}
.sudoku-cell:nth-child(3n){
  border-bottom: 2px solid #000;
}
.sudoku-cell:nth-child(1){
  border-top: 2px solid #000;
}
.sudoku-row:nth-child(3n) .sudoku-cell,
.sudoku-row:nth-child(3n) .sudoku-cell:nth-child(3n) {
  border-right: 2px solid black;
}
.sudoku-row:nth-child(1) .sudoku-cell,
.sudoku-row:nth-child(1) .sudoku-cell:nth-child(1) {
  border-left: 2px solid black;
}
.sudoku-cell input {
  width: 100%;
  height: 100%;
  border: none;
  text-align: center;
  font-size: 20px; /* 根据需要调整字体大小 */
  box-sizing: border-box; /* 确保边框和内边距不会影响总尺寸 */
}
.sudoku-cell input:focus {
  outline: none; /* 移除焦点时的轮廓线 */
}
.error-message {
  color: red; /* 错误信息的颜色 */
  margin-top: 10px; /* 错误信息与数独板的间距 */
}
</style>
