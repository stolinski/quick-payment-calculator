<script lang="ts">
	/**
	 * Reverse Mortgage Calculator Component
	 * Calculates house price FROM monthly payment (not the other way around)
	 * Uses Svelte 5 runes for reactive state management
	 */

	// PRIMARY INPUTS (all regular $state - user editable)
	let monthlyPayment = $state(1600);
	let downPayment = $state(313000);
	let additionalDownPayment = $state(0);
	let interestRate = $state(7.0);
	let loanTermYears = $state(30);

	// DERIVED VALUES

	// Total down payment
	let totalDownPayment = $derived(downPayment + additionalDownPayment);

	// Calculate loan amount from monthly payment using reverse mortgage formula
	// P = M × [(1+r)^n - 1] / [r(1+r)^n]
	// Where: M = monthly payment, P = loan principal, r = monthly interest rate, n = number of payments
	let loanAmount = $derived.by(() => {
		if (monthlyPayment <= 0 || interestRate <= 0) return 0;

		const monthlyRate = interestRate / 100 / 12;
		const numberOfPayments = loanTermYears * 12;

		// Reverse mortgage formula
		const principal =
			(monthlyPayment * (Math.pow(1 + monthlyRate, numberOfPayments) - 1)) /
			(monthlyRate * Math.pow(1 + monthlyRate, numberOfPayments));

		return principal;
	});

	// House price = Loan amount + Total down payment
	let housePrice = $derived(loanAmount + totalDownPayment);

	// Down payment percentage = (Total down payment / House price) × 100
	let downPaymentPercentage = $derived.by(() => {
		if (housePrice <= 0) return 0;
		return (totalDownPayment / housePrice) * 100;
	});

	// Additional calculations for display
	let totalAmountPaid = $derived(monthlyPayment * loanTermYears * 12);
	let totalInterestPaid = $derived(totalAmountPaid - loanAmount);

	/**
	 * Format number as currency with $ and commas
	 */
	function formatCurrency(value: number): string {
		return new Intl.NumberFormat('en-US', {
			style: 'currency',
			currency: 'USD',
			minimumFractionDigits: 0,
			maximumFractionDigits: 0
		}).format(value);
	}

	/**
	 * Format percentage with 2 decimal places
	 */
	function formatPercent(value: number): string {
		return value.toFixed(2) + '%';
	}

	/**
	 * Handle numeric input changes
	 */
	function handleNumberInput(setter: (value: number) => void) {
		return (e: Event) => {
			const input = e.target as HTMLInputElement;
			const value = parseFloat(input.value) || 0;
			setter(Math.max(0, value));
		};
	}
</script>

<div class="mortgage-calculator">
	<h1>Reverse Mortgage Calculator</h1>
	<p class="subtitle">
		Calculate what house price you can afford based on your desired monthly payment
	</p>

	<div class="calculator-grid">
		<!-- Input Section -->
		<div class="input-section">
			<h2>Your Inputs</h2>

			<div class="input-group featured">
				<label for="monthly-payment">
					<span class="label-text">Desired Monthly Payment</span>
					<span class="label-hint">How much do you want to pay per month?</span>
				</label>
				<input
					id="monthly-payment"
					type="number"
					bind:value={monthlyPayment}
					step="100"
					min="0"
					class="featured-input"
				/>
			</div>

			<div class="input-row">
				<div class="input-group">
					<label for="down-payment">
						<span class="label-text">Down Payment</span>
					</label>
					<input id="down-payment" type="number" bind:value={downPayment} step="1000" min="0" />
				</div>

				<div class="input-group">
					<label for="additional-down-payment">
						<span class="label-text">Additional Down Payment</span>
					</label>
					<input
						id="additional-down-payment"
						type="number"
						bind:value={additionalDownPayment}
						step="1000"
						min="0"
					/>
				</div>
			</div>

			<div class="input-row">
				<div class="input-group">
					<label for="interest-rate">
						<span class="label-text">Interest Rate (%)</span>
						<span class="label-hint">Colorado default: 7.0%</span>
					</label>
					<input id="interest-rate" type="number" bind:value={interestRate} step="0.1" min="0" />
				</div>

				<div class="input-group">
					<label for="loan-term">
						<span class="label-text">Loan Term (years)</span>
					</label>
					<input
						id="loan-term"
						type="number"
						bind:value={loanTermYears}
						step="1"
						min="1"
						max="50"
					/>
				</div>
			</div>
		</div>

		<!-- Results Section -->
		<div class="results-section">
			<h2>Calculated Results</h2>

			<div class="result-highlight">
				<div class="result-label">You Can Afford a House Worth</div>
				<div class="result-value-large">{formatCurrency(housePrice)}</div>
			</div>

			<div class="results-grid">
				<div class="result-item">
					<span class="result-label">Loan Amount</span>
					<span class="result-value">{formatCurrency(loanAmount)}</span>
				</div>

				<div class="result-item">
					<span class="result-label">Total Down Payment</span>
					<span class="result-value">{formatCurrency(totalDownPayment)}</span>
				</div>

				<div class="result-item">
					<span class="result-label">Down Payment %</span>
					<span class="result-value">{formatPercent(downPaymentPercentage)}</span>
				</div>

				<div class="result-item">
					<span class="result-label">Total Interest Paid</span>
					<span class="result-value">{formatCurrency(totalInterestPaid)}</span>
				</div>

				<div class="result-item">
					<span class="result-label">Total Amount Paid</span>
					<span class="result-value">{formatCurrency(totalAmountPaid)}</span>
				</div>
			</div>

			<div class="breakdown">
				<h3>Breakdown</h3>
				<ul>
					<li>Monthly Payment: <strong>{formatCurrency(monthlyPayment)}</strong></li>
					<li>Number of Payments: <strong>{loanTermYears * 12}</strong></li>
					<li>Down Payment: <strong>{formatCurrency(downPayment)}</strong></li>
					{#if additionalDownPayment > 0}
						<li>Additional Down: <strong>{formatCurrency(additionalDownPayment)}</strong></li>
					{/if}
				</ul>
			</div>
		</div>
	</div>
</div>

<style>
	.mortgage-calculator {
		max-width: 1400px;
		margin: 0 auto;
		padding: 2rem;
		font-family:
			system-ui,
			-apple-system,
			BlinkMacSystemFont,
			'Segoe UI',
			sans-serif;
	}

	h1 {
		font-size: 2.5rem;
		margin-bottom: 0.5rem;
		color: #1a1a1a;
		font-weight: 700;
	}

	.subtitle {
		font-size: 1.1rem;
		color: #666;
		margin-bottom: 2rem;
	}

	h2 {
		font-size: 1.5rem;
		margin-bottom: 1.5rem;
		color: #333;
		font-weight: 600;
	}

	h3 {
		font-size: 1.1rem;
		margin-bottom: 1rem;
		color: #444;
		font-weight: 600;
	}

	.calculator-grid {
		display: grid;
		grid-template-columns: 1fr 1fr;
		gap: 2rem;
		align-items: start;
	}

	@media (max-width: 968px) {
		.calculator-grid {
			grid-template-columns: 1fr;
		}
	}

	.input-section,
	.results-section {
		background: #ffffff;
		padding: 2rem;
		border-radius: 12px;
		box-shadow: 0 4px 6px rgba(0, 0, 0, 0.07);
		border: 1px solid #e5e7eb;
	}

	.input-group {
		margin-bottom: 1.5rem;
	}

	.input-group.featured {
		background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
		padding: 1.5rem;
		border-radius: 8px;
		margin-bottom: 2rem;
	}

	.input-group.featured .label-text,
	.input-group.featured .label-hint {
		color: white;
	}

	.input-row {
		display: grid;
		grid-template-columns: 1fr 1fr;
		gap: 1rem;
		margin-bottom: 1.5rem;
	}

	@media (max-width: 640px) {
		.input-row {
			grid-template-columns: 1fr;
		}
	}

	label {
		display: flex;
		flex-direction: column;
		gap: 0.5rem;
	}

	.label-text {
		font-weight: 600;
		color: #374151;
		font-size: 0.95rem;
	}

	.label-hint {
		font-size: 0.8rem;
		color: #6b7280;
		font-weight: 400;
	}

	input[type='number'] {
		padding: 0.75rem;
		font-size: 1rem;
		border: 2px solid #d1d5db;
		border-radius: 6px;
		transition: all 0.2s;
		background: white;
		font-family: inherit;
	}

	input[type='number']:focus {
		outline: none;
		border-color: #667eea;
		box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
	}

	.featured-input {
		font-size: 1.5rem !important;
		font-weight: 700;
		padding: 1rem !important;
		border: 3px solid white !important;
		background: white;
		color: #1a1a1a;
	}

	.featured-input:focus {
		box-shadow: 0 0 0 4px rgba(255, 255, 255, 0.3) !important;
	}

	.result-highlight {
		background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
		padding: 2rem;
		border-radius: 8px;
		text-align: center;
		margin-bottom: 2rem;
		box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
	}

	.result-label {
		font-weight: 600;
		color: #4b5563;
		font-size: 0.9rem;
	}

	.result-highlight .result-label {
		color: rgba(255, 255, 255, 0.9);
		font-size: 1rem;
		margin-bottom: 0.5rem;
		text-transform: uppercase;
		letter-spacing: 0.5px;
	}

	.result-value-large {
		font-size: 3rem;
		font-weight: 800;
		color: white;
		line-height: 1.2;
	}

	@media (max-width: 640px) {
		.result-value-large {
			font-size: 2rem;
		}
	}

	.results-grid {
		display: flex;
		flex-direction: column;
		gap: 0.75rem;
		margin-bottom: 2rem;
	}

	.result-item {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 1rem;
		background: #f9fafb;
		border-radius: 6px;
		border-left: 4px solid #667eea;
	}

	.result-value {
		font-size: 1.1rem;
		font-weight: 700;
		color: #1a1a1a;
	}

	.breakdown {
		background: #f9fafb;
		padding: 1.5rem;
		border-radius: 6px;
		border: 1px solid #e5e7eb;
	}

	.breakdown ul {
		list-style: none;
		padding: 0;
		margin: 0;
	}

	.breakdown li {
		padding: 0.5rem 0;
		color: #4b5563;
		display: flex;
		justify-content: space-between;
		border-bottom: 1px solid #e5e7eb;
	}

	.breakdown li:last-child {
		border-bottom: none;
	}

	.breakdown strong {
		color: #1a1a1a;
	}

	/* Remove spinner from number inputs */
	input[type='number']::-webkit-inner-spin-button,
	input[type='number']::-webkit-outer-spin-button {
		-webkit-appearance: none;
		margin: 0;
	}

	input[type='number'] {
		-moz-appearance: textfield;
	}
</style>
