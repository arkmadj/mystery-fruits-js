<script setup>
import { computed, onMounted, ref } from "vue";

import HistorySection from "@/components/HistorySection.vue";
import FruitButton from "../components/FruitButton.vue";
import GameOverSection from "../components/GameOverSection.vue";
import { useFruits } from "../composables/fruits";
import { useHistory } from "../composables/history";
import { countMatchingArray, countMatchingArrayOrder } from "../lib/counter";

const { hiddenFruits, randomFruits, resetFruits } = useFruits();
const { history, addToHistory, clearHistory } = useHistory();

const availableAttempts = ref(6);
const selectedFruits = ref([]);
const isGameOver = ref(false);
const isLoading = ref(false);
const showInstructions = ref(true);

function selectFruit(fruit) {
	if (isLoading.value || isGameOver.value) return;

	if (selectedFruits.value.length < 3) {
		if (selectedFruits.value.includes(fruit)) {
			unselectFruit(fruit);
		} else {
			selectedFruits.value.push(fruit);
		}
	}

	if (selectedFruits.value.length === 3) {
		isLoading.value = true;
		availableAttempts.value--;

		setTimeout(function () {
			isGameOver.value =
				availableAttempts.value <= 0 ||
				countMatchingArray(selectedFruits.value, hiddenFruits.value) ===
					hiddenFruits.value.length;

			if (!isGameOver.value) {
				addSelectedFruitsToHistory();
				clearSelectedFruits();
			}

			isLoading.value = false;
		}, 400);
	}
}

function clearSelectedFruits() {
	selectedFruits.value = [];
}

function unselectFruit(fruit) {
	selectedFruits.value = selectedFruits.value.filter((f) => f !== fruit);
}

function isCorrectAnswer() {
	const correctFruitsOrder = countMatchingArrayOrder(
		hiddenFruits.value,
		selectedFruits.value
	);
	return correctFruitsOrder === 3;
}

function addSelectedFruitsToHistory() {
	addToHistory({
		selectedFruits: selectedFruits.value,
		correctFruits: countMatchingArray(hiddenFruits.value, selectedFruits.value),
		correctFruitsOrder: countMatchingArrayOrder(
			hiddenFruits.value,
			selectedFruits.value
		),
	});
}

function getSelectedFruitIndex(fruit) {
	return selectedFruits.value.indexOf(fruit);
}

function restartGame() {
	resetFruits();

	selectedFruits.value = [];
	availableAttempts.value = 6;
	clearHistory();
	isGameOver.value = false;
	console.log(`Answer: ${hiddenFruits.value}`);
}

onMounted(() => {
	console.log(`Answer: ${hiddenFruits.value}`);
});

const reversedHistory = computed(() => history.value.slice().reverse());
</script>

<template>
	<main class="md:p-12 p-2 relative">
		<div
			:class="`mb-10 bg-red-100 inline-block p-7 border-2 border-red-300 rounded-lg relative ${
				showInstructions ? '' : 'animate-close-section'
			} overflow-hidden`"
		>
			<div
				class="bg-red-400 p-1 rounded-full h-6 w-6 flex items-center justify-center text-red-900 absolute top-3 right-3 hover:bg-red-900 hover:text-red-400 cursor-pointer transition-all delay-75 ease-in"
				@click="showInstructions = !showInstructions"
			>
				✕
			</div>

			<p>Hello Stranger 🕵️‍♂️</p>
			<p>
				Are you in search of the mystery fruits🍓? You’ve come to the right
				place🌴. All you have to do is select the mystery fruits in the correct
				order 1️⃣ 2️⃣ 3️⃣.
			</p>
			<p>
				You have 6 attempts before the fruits are lost forever😞. You can check
				the history tab on the left to see details on your previous
				selections⌛.
			</p>
			<p>Good luck and Have fun🌟.</p>
		</div>
		<div
			v-if="!isGameOver"
			id="game"
			class="grid lg:grid-cols-2 place-items-start justify-items-center content-center gap-4"
		>
			<div class="h-full w-full">
				<div class="mt-4 text-pink-500 font-semibold">
					Remaining Attempts:
					<span class="font-black">{{ availableAttempts }}</span>
				</div>
				<div class="mt-4 grid grid-cols-3 gap-2 place-items-center">
					<div v-for="fruit in randomFruits" :key="fruit">
						<FruitButton
							:fruit="fruit"
							:selected-index="getSelectedFruitIndex(fruit)"
							:disabled="isLoading.value"
							class="h-24 w-24 border"
							:class="[
								{
									'bg-red-100':
										selectedFruits.length === 3 &&
										!isCorrectAnswer() &&
										selectedFruits.includes(fruit),
									'bg-green-100':
										selectedFruits.length === 3 &&
										isCorrectAnswer() &&
										selectedFruits.includes(fruit),
								},
							]"
							@click="selectFruit(fruit)"
						/>
					</div>
				</div>
			</div>
			<div class="h-full w-full">
				<HistorySection :history="reversedHistory" />
			</div>
		</div>
		<GameOverSection
			v-if="isGameOver"
			:restart-game="restartGame"
			:hidden-fruits="hiddenFruits"
			:attempts-left="availableAttempts"
			:selected-fruits="selectedFruits"
		/>
	</main>
</template>
