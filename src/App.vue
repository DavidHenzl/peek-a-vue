<template>
  <h1>Peek-a-Vue</h1>
	<button @click="newGame">New Game</button>
  <section class="game-board">
    <SingleCard
    v-for="(card, index) in cardList"
    :key="`card-${index}`"
		:matched="card.matched"
    :value="card.value"
		:visible="card.visible"
		:position="card.position"
		@select-card="flipCard"
		/>
  </section>
	<h2>{{ status }}</h2>
</template>

<script>
import { computed, ref, watch } from 'vue'
import SingleCard from './components/SingleCard'

export default {
  name: 'App',
  components: {
    SingleCard
  },
  setup() {
    const cardList = ref([])
		const userSelection = ref([])
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

		let defaultFaceValues = [1,1,2,2,3,3,4,4,5,5,6,6,7,7,8,8]

		const newGame = () => {
			cardList.value = ([])
			let currentFaceValues = [...defaultFaceValues]
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
				userSelection.value[1] = payload
			} else {
				userSelection.value[0] = payload
			}
		}

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
						cardList.value[cardOne.position].visible = false
						cardList.value[cardTwo.position].visible = false
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
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.game-board {
  display: grid;
  grid-template-columns: 100px 100px 100px 100px;
  grid-template-rows: 100px 100px 100px 100px;
  grid-column-gap: 30px;
  grid-row-gap: 30px;
  justify-content: center;
}
</style>
