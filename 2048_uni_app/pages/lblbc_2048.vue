
<template>
	<view class="wrapper">
		<view class="btn btn-mg" @click="startGame">新游戏</view>
		<view>
			<view class="over" v-if="isGameOver">
				<p>游戏结束!</p>
				<view class="btn" @click="startGame">重新开始</view>
			</view>
			<view class="over" v-if="ttt">
				<p>恭喜过关</p>
				<view class="btn" @click="ggg">点击记录</view>
			</view>
			<view class="box">
				<view class="row" v-for="row in cellArr">
					<view class="col" :class="'cell-' + col" v-for="col in row">
						{{ col }}
					</view>
				</view>
			</view>
			
			
			<view>
				{{time}}
			</view>
			<view>
				<view>你的最好的通关时间是{{vvv}}</view>
			</view>
			
			
		</view>
	</view>
</template>

<script lang="ts">
	export default {
		data() {
			return {
				cellArr: [],
				isGameOver: false,
				time: '00:00:00',
				timer: 0,
				hour: 0,
				minutes: 0,
				seconds: 0,
				min:2,
				vvv:'无'
				
				
				
			}
		},
		computed:{
			max(){
				
				
				if(this.seconds != 0 ){
					
					
					for(let i=0;i<4;i++){ //外循环：控制数组的行数。arr.length表示arr数组的行数.i表示行下标
					           for(let j=0;j<4;j++){//内循环：控制数组的列。arr[i].length表示i行的元素个数。j表示列下标
														 
					                if(this.cellArr[i][j] > this.max){
														 this.min = this.cellArr[i][j]
													 }
					           }
					           
					       }
				}
					   return this.min
			},
			
			ttt(){
				var msg = false
				if(this.max !=2048){
					msg =false
				}else if(this.max == 2048){
					msg = true
				}
				
				return msg
			}
		},
		mounted() {
			alert('请用上下左右四个按键操纵游戏。点击新游戏即可开启游戏。')
			document.addEventListener('keyup', this.keyDown)
		},
		methods: {
			startGame() {
				this.pause()
					if(this.timer == 0){
						
						
						this.begin()
					}
				this.min = 2
				this.isGameOver = false
				this.cellArr = Array.from(Array(4)).map(() => Array(4).fill(0))
				this.fillOneEmptyCell()
				this.fillOneEmptyCell()
				
			},
			//找到可以用于生成新数字的单元格，并生成新的数字进行填充
			fillOneEmptyCell() {
				let cellIndex = this.findOneEmptyCell()
				if (cellIndex != -1) {
					let row = Math.floor(cellIndex / 4)
					let col = cellIndex % 4
					this.cellArr[row][col] = this.getRandomValue()
				}
			},
			//生成新的数字，90%几率生成2，10%几率生成4
			getRandomValue() {
				return this.random(1) < 0.9 ? 2 : 4
			},
			//生成随机数，0=<结果<max
			random(max) {
				return Math.floor(Math.random() * max)
			},
			//找到可以用于生成新数字的单元格
			findOneEmptyCell() {
				let cells = []
				for (let i = 0; i < 4; i++) {
					for (let j = 0; j < 4; j++) {
						if (!this.cellArr[i][j]) {
							cells.push(i * 4 + j)
						}
					}
				}
				if (cells.length) {
					return cells[this.random(cells.length)]
				} else {
					return -1
				}
			},
			/*
			 判断是否还可以移动。
			 1、当前单元格是否为0；
			 2、当前单元格和右侧单元格是否相等；
			 3、当前单元格和下方单元格是否相等。
			 */
			canMove() {
				for (let i = 0; i < 4; i++) {
					for (let j = 0; j < 4; j++) {
						let cell = this.cellArr[i][j]
						if (cell) {
							//和右侧单元格比较，是否相等
							if (j < 3 && this.cellArr[i][j] == this.cellArr[i][j + 1]) {
								return true
							}
							//和下方单元格比较，是否相等
							if (i < 3 && this.cellArr[i][j] == this.cellArr[i + 1][j]) {
								return true
							}
						} else {
							return true
						}
					}
				}
				return false
			},
			/**
			 * 将单元格数向左或向右移动，移除0并对相邻相同数进行叠加
			 * toLeft表示是否是向左
			 */
			horizontalMoveCells(toLeft: boolean) {
				for (let i = 0; i < 4; i++) {
					let newArr = Array(4).fill(0)
					for (let j = 0; j < 4; j++) {
						newArr[j] = this.cellArr[i][j]
					}
					let resultArr = this.removeZerosAndAdd(newArr, toLeft)
					for (let j = 0; j < 4; j++) {
						this.cellArr[i][j] = resultArr[j]
					}
				}
			},
			/**
			 * 将单元格数向下或向上移动，移除0并对相邻相同数进行叠加
			 * toUp表示是否是向上
			 */
			verticalMoveCells(toUp: boolean) {
				for (let i = 0; i < 4; i++) {
					let newArr = Array(4).fill(0)
					for (let j = 0; j < 4; j++) {
						newArr[j] = this.cellArr[j][i]
					}
					let resultArr = this.removeZerosAndAdd(newArr, toUp)
					for (let j = 0; j < 4; j++) {
						this.cellArr[j][i] = resultArr[j]
					}
				}
			},
			/**
			 * 1、去掉数组中的0，向头或向尾压缩数组。
			 * 0,4,0,4向左压缩变成：4,4,0,0. 向右压缩变成：0,0,4,4
			 * 2、相邻的数如果相同，则进行相加运算。
			 * 4,4,0,0向左叠加变成：8,0,0,0. 向右叠加变成：0,0,0,8
			 * toHead表示是否是头压缩
			 */
			removeZerosAndAdd(arr: number[], toHead: boolean) {
				let newArr = Array(4).fill(0)
				let arrWithoutZero = arr.filter((x) => x !== 0) //去掉所有的0
				if (arrWithoutZero.length == 0) {
					return newArr
				}
				if (toHead) {
					for (let i = 0; i < arrWithoutZero.length; i++) {
						newArr[i] = arrWithoutZero[i]
					}
					//对相邻相同的数进行叠加
					for (let i = 0; i < newArr.length - 1; i++) {
						if (newArr[3 - i] === newArr[2 - i] && newArr[3 - i] !== 0) {
							newArr[3 - i] = 0
							newArr[2 - i] *= 2
						}
					}
				} else {
					for (let i = 0; i < arrWithoutZero.length; i++) {
						newArr[newArr.length - i - 1] =
							arrWithoutZero[arrWithoutZero.length - i - 1]
					}

					//对相邻相同的数进行叠加
					for (let i = 0; i < newArr.length - 1; i++) {
						if (newArr[i] === newArr[i + 1] && newArr[i] !== 0) {
							newArr[i] = 0
							newArr[i + 1] *= 2
						}
					}
				}

				return newArr
			},
			//键盘监听事件
			keyDown(e) {
				let arr = null
				switch (e.keyCode) {
					case 38: //上
						this.moveUp()
						break
					case 40: //下
						this.moveDown()
						break
					case 37: //左
						this.moveLeft()
						break
					case 39: //右
						this.moveRight()
						break
				}
			},
			moveUp() {
				this.verticalMoveCells(true)
				this.checkGameOverOrContinue()
			},
			moveDown() {
				this.verticalMoveCells(false)
				this.checkGameOverOrContinue()
			},
			moveLeft() {
				this.horizontalMoveCells(true)
				this.checkGameOverOrContinue()
			},
			moveRight() {
				this.horizontalMoveCells(false)
				this.checkGameOverOrContinue()
			},
			checkGameOverOrContinue() {
				if (this.canMove()) {
					this.fillOneEmptyCell()
				} else {
					this.isGameOver = true
				}
			},
			
			
			
			// 开始计时
			begin() {
				
				
				this.timer = setInterval(this.startTimer, 1000);
			},
			startTimer() {
				this.seconds += 1;
				if (this.seconds >= 60) {
					this.seconds = 0;
					this.minutes = this.minutes + 1;
				}
				if (this.minutes >= 60) {
					this.minutes = 0;
					this.hour = this.hour + 1;
				}
				this.time = (this.hour < 10 ? '0' + this.hour : this.hour) + ':' + (this.minutes < 10 ? '0' + this
					.minutes :
					this.minutes) + ':' + (this.seconds < 10 ? '0' + this.seconds : this.seconds);
			}, 
			
			
			//结算计时
			pause() {
				if (this.timer != 0) {
					clearInterval(this.timer);
					this.timer = 0
					this.time = '00:00:00'
					this.seconds = 0
					this.minutes = 0
					
					
				}
			},
			
			
			ggg(){
				
				this.vvv = this.time
				
				this.startGame()
			}
			
		}
	}
</script>

<style scoped lang="less">
	.wrapper {
		display: flex;
		justify-content: center;
		flex-direction: column;
		align-items: center;
		margin-top: 50px;

		.over {
			position: absolute;
			width: 400px;
			height: 400px;
			background: rgba(238, 228, 218, 0.73);
			z-index: 1000;
			border-radius: 5px;
			text-align: center;
			color: #8f7a66;

			p {
				font-size: 60px;
				font-weight: bold;
				height: 60px;
				line-height: 60px;
			}
		}

		.btn {
			display: inline-block;
			padding: 0 20px;
			height: 40px;
			line-height: 40px;
			border-radius: 5px;
			cursor: pointer;
			text-align: center;
			color: #f9f6f2;
			background: #8f7a66;

			&.btn-mg {
				margin-bottom: 10px;
			}
		}

		.box {
			width: 400px;
			height: 400px;
			padding: 15px;
			display: flex;
			flex-direction: column;
			justify-content: space-between;
			box-sizing: border-box;
			border-radius: 5px;
			background: #bbada0;

			.row {
				width: 100%;
				height: 23%;
				display: flex;
				flex-direction: row;
				justify-content: space-between;

				.col {
					width: 23%;
					height: 100%;
					display: flex;
					align-items: center;
					justify-content: center;
					font-size: 24px;
					border-radius: 3px;
					background: #cec1b3;

					&.cell-0 {
						color: #00000000;
					}

					&.cell-2 {
						background: #eee4da;
					}

					&.cell-4 {
						background: #ede0c8;
					}

					&.cell-8 {
						background: #f26179;
					}

					&.cell-16 {
						background: #f59563;
					}

					&.cell-32 {
						background: #f67c5f;
					}

					&.cell-64 {
						background: #f65e36;
					}

					&.cell-128 {
						background: #edcf72;
					}

					&.cell-256 {
						background: #edcc61;
					}

					&.cell-512 {
						background: #90c000;
					}

					&.cell-1024 {
						background: #3365a5;
					}

					&.cell-2048 {
						background: #90c000;
					}

					&.cell-4096 {
						background: #60b0c0;
					}

					&.cell-8192 {
						background: #9030c0;
					}
				}
			}
		}
	}
</style>
