<script>
	import ActionBar from './ActionBar/index.svelte';
	import Keyboard from './Keyboard.svelte';
	import { hint } from '@sudoku/stores/hint';
</script>

<div class="px-4 pb-5 flex justify-center">
	<div class="w-full max-w-xl">
		<ActionBar />

		{#if $hint}
			<div class="hint-message">
				<span class="hint-label">Hint</span>
				<span class="hint-value">{$hint.value}</span>
				<span class="hint-type">{$hint.hintType}</span>
				<span class="hint-text">{$hint.reason}</span>
			</div>
		{/if}

		<Keyboard />
	</div>
</div>

<style>
	.hint-message {
		@apply w-full mb-4 px-4 py-3 bg-white text-gray-700 rounded-xl shadow-md flex items-start;
	}

	.hint-label {
		@apply mr-3 text-xs font-semibold uppercase tracking-widest text-blue-600;
	}

	.hint-value {
		@apply mr-3 px-2 py-1 rounded-lg bg-blue-100 text-blue-700 font-semibold text-sm leading-none;
	}

	.hint-type {
		@apply mr-3 text-xs font-semibold uppercase tracking-widest text-gray-500;
	}

	.hint-text {
		@apply text-sm leading-snug;
	}
</style>
