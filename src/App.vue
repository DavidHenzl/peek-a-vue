<template>
	<h1 class="sr-only">Peek-a-Vue</h1>
  <img src="/images/peek-a-vue-title.png"
	alt="Peek-a-Vue" class="title">
  <transition-group tag="section" class="game-board">
		<SingleCard
    v-for="(card, index) in cardList"
    :key="`card-${index}`"
		:matched="card.matched"
    :value="card.value"
		:visible="card.visible"
		:position="card.position"
		@select-card="flipCard"
		/>
  </transition-group>
	<h2>{{ status }}</h2>
	<button @click="newGame" class="button">
		<img src="/images/restart.svg" alt="Restart Icon"> Restart Game
	</button>
</template>

<script>
import { computed, ref, watch, onMounted } from 'vue'
import { launchConfetti } from './utilities/confetti'
import SingleCard from './components/SingleCard'

export default {
  name: 'App',
  components: {
    SingleCard
  },
	setup() {
		const cardList = ref([])
		const userSelection = ref([])

		onMounted(() => {
			newGame()
		})

		const status = computed(() => {
			if (remainingPairs.value === 0) {
				return 'Player wins!'
			} else {
				return `Remaining Pairs: ${remainingPairs.value}`
			}
		})

		const remainingPairs = computed(() => {
			const remainingCards = cardList.value.filter(card => card.matched === false).length

			return remainingCards / 2
		})

		let defaultFaceValues = [
			'bat',
			'candy',
			'cauldron',
			'cupcake',
			'ghost',
			'moon',
			'pumpkin',
			'witch-hat'
		]

		const newGame = () => {
			cardList.value = ([])
			let currentFaceValues = [...defaultFaceValues, ...defaultFaceValues]
			for (let i = 0; i < 16; i++) {
				let faceValueOfCard = currentFaceValues[Math.floor(Math.random()*currentFaceValues.length)]
				let index = currentFaceValues.indexOf(faceValueOfCard)
				currentFaceValues.splice(index, 1)
				cardList.value.push({
					value: faceValueOfCard,
					visible: false,
					position: i,
					matched: false
				})
			}
		}

		const flipCard = (payload) => {
			cardList.value[payload.position].visible = true

			if (userSelection.value[0]) {
				if (userSelection.value[0].position === payload.position) {
					return
				}
				userSelection.value[1] = payload
			} else {
				userSelection.value[0] = payload
			}
		}

		watch(remainingPairs, currentValue => {
			if (currentValue === 0) {
				launchConfetti()
			}
		})

		watch(
			userSelection,
			currentValue => {
				if (currentValue.length === 2) {
					const cardOne = currentValue[0]
					const cardTwo = currentValue[1]

					if (cardOne.faceValue === cardTwo.faceValue) {
						cardList.value[cardOne.position].matched = true
						cardList.value[cardTwo.position].matched = true
					} else {
						setTimeout(() => {
							cardList.value[cardOne.position].visible = false
							cardList.value[cardTwo.position].visible = false
						}, 1000)
					}


					userSelection.value.length = 0
				}
			},
			{ deep: true}
		)

    return {
      cardList,
			flipCard,
			userSelection,
			status,
			newGame
    }
  }
}
</script>

<style>
html, body {
	margin: 0;
	padding: 0;
}

h1 {
	margin-top: 0;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
	background-image: url('../public/images/page-bg.png');
	background-color: #00070c;
	height: 100vh;
	color: white;
	padding-top: 50px;
}

.button {
	background-color: orange;
	color: white;
	padding: 0.75rem 0.5rem;
	display: flex;
	align-items: center;
	justify-content: center;
	margin: 0 auto;
	font-weight: bold;
}

.button img {
	padding-right: 5px;
}

.game-board {
  display: grid;
  grid-template-columns: repeat(4, 100px);
  grid-template-rows: repeat(4, 100px);
  grid-column-gap: 24px;
  grid-row-gap: 24px;
  justify-content: center;
}

.sr-only {
	position: absolute;
	width: 1px;
	height: 1px;
	padding: 0;
	margin: -1px;
	overflow: hidden;
	clip: rect(0, 0, 0, 0);
	border: 0;
}

.title {
	padding-bottom: 30px;
}

</style>
