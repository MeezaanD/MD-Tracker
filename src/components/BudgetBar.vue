<template>
	<div class="container">
		<div>
			<div>
				<h3 class="m-3 text-center">My Budget</h3>
				<div class="progress-container">
					<div class="progress-circle">
						<svg class="progress-svg" width="100" height="100">
							<circle class="progress-bg" cx="50" cy="50" r="45" stroke="#eee" stroke-width="10"
								fill="transparent"></circle>
							<circle class="progress-bar" cx="50" cy="50" r="45" :stroke-dasharray="circumference"
								:stroke-dashoffset="progressOffset" stroke="#dc3545" stroke-width="10" fill="transparent">
							</circle>
						</svg>
					</div>
					<div class="progress-label">
						<span>{{ budgetPercentage.toFixed(0) }}%</span>
					</div>
				</div>
				<div>
					<p class="text-center" v-if="totalBudget === 0">No budget currently selected</p>
					<p class="text-center" v-else>Available Budget: R<span>{{ availableBudget }} / {{ totalBudget }}</span>
					</p>
				</div>
			</div>
			<!-- /// -->
			<div>
				<div class="form-floating my-2 transparent-select">
					<label for="budgetSelect" v-if="!selectedBudget">Select Budget</label>
					<select class="form-select text-white" v-model="selectedBudget">
						<option class="text-black" v-for="(budget, index) in budgets" :key="index" :value="budget.name">{{
							budget.name }}</option>
					</select>
				</div>
			</div>
		</div>
		<AddBudget @budgetAdded="addBudget" />
	</div>
</template>
  
<script>
import AddBudget from "./AddBudget.vue";

export default {
	components: {
		AddBudget
	},
	data() {
		return {
			budgets: [],
			expenses: [],
			selectedBudget: "",
			editingIndex: -1,
		};
	},
	computed: {
		totalBudget() {
			if (this.selectedBudget === "") {
				return 0;
			}
			return this.budgets.find((budget) => budget.name === this.selectedBudget)?.amount || 0;
		},
		budgetPercentage() {
			if (this.totalBudget === 0) return 0;
			return Math.round(((this.totalBudget - this.availableBudget) / this.totalBudget) * 100);
		},
		circumference() {
			return Math.PI * 45 * 2;
		},
		progressOffset() {
			return this.circumference - (this.budgetPercentage / 100) * this.circumference;
		},
		availableBudget() {
			const budget = this.budgets.find((budget) => budget.name === this.selectedBudget);
			if (!budget) {
				return 0;
			}
			const totalExpenses = this.expenses.reduce((total, expense) => {
				if (expense.budgetName === this.selectedBudget) {
					return total + expense.amount;
				}
				return total;
			}, 0);
			return budget.amount - totalExpenses;
		},
		filteredExpenses() {
			return this.expenses.filter((expense) => expense.budgetName === this.selectedBudget);
		},
	},
	mounted() {
		this.loadDataFromLocalStorage();
	},
	methods: {
		addBudget(budget) {
			this.budgets.push(budget);
			this.saveDataToLocalStorage();
		},
		deleteBudget(index) {
			if (confirm("Are you sure you want to delete this budget?")) {
				const deletedBudget = this.budgets.splice(index, 1)[0];
				this.expenses = this.expenses.filter((expense) => expense.budgetName !== deletedBudget.name);

				this.saveDataToLocalStorage();
			}
		},
		saveDataToLocalStorage() {
			localStorage.setItem("budgets", JSON.stringify(this.budgets));
		},
		loadDataFromLocalStorage() {
			const savedBudgets = localStorage.getItem("budgets");
			if (savedBudgets) {
				this.budgets = JSON.parse(savedBudgets);
			}
			const savedExpenses = localStorage.getItem("expenses");
			if (savedExpenses) {
				this.expenses = JSON.parse(savedExpenses);
			}
		}
	},
};
</script>
  
<style scoped>
</style>
  