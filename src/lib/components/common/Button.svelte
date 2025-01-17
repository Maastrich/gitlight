<script lang="ts">
	import { createEventDispatcher } from 'svelte';

	export let type: 'primary' | 'secondary' = 'primary';
	export let small = false;
	export let href: string | undefined = undefined;
	export let external = false;
	export let loading = false;
	export let disabled = false;

	const dispatch = createEventDispatcher();

	function handleClick(event: MouseEvent) {
		if (!disabled && !loading) {
			dispatch('click', { event });
		}
	}
</script>

{#if href}
	<a
		class="button {type}"
		class:small
		class:loading
		class:disabled
		{href}
		target={external ? '_blank' : undefined}
		rel={external ? 'noreferrer' : undefined}
		on:click={handleClick}
	>
		<span class="content">
			<slot />
		</span>
	</a>
{:else}
	<button class="button {type}" class:small class:loading class:disabled on:click={handleClick}>
		<span class="content">
			<slot />
		</span>
		{#if loading}
			<span class="spinner" />
		{/if}
	</button>
{/if}

<style lang="scss">
	.button {
		@include mixins.shiny(variables.$blue-2);
		@include mixins.shadow;

		&.secondary {
			@include mixins.shiny(variables.$grey-3);
		}

		&:not(.small) {
			@include typography.bold;
			padding: 0.75em 1em;
		}

		&.small {
			padding: 0.5rem;
		}

		&.loading,
		&.disabled {
			opacity: 0.5;
			cursor: not-allowed;
		}

		&.loading .content {
			opacity: 0;
		}

		&:active .content {
			scale: 95%;
		}

		.content {
			width: 100%;
			height: 100%;
			display: flex;
			align-items: center;
			justify-content: center;
			gap: 0.5rem;
			white-space: nowrap;
			transition: scale 0.05s ease-in-out;

			:global(svg) {
				flex: 0 0 1.25rem;
				height: 1.25rem;
			}
		}

		.spinner {
			position: absolute;
			inset: 50%;
			width: 1.5rem;
			height: 1.5rem;
			border-top: 2px solid;
			border-left: 2px solid;
			border-radius: 50%;
			animation: spin 1s linear infinite;

			@keyframes spin {
				0% {
					transform: translate(-50%, -50%) rotate(0deg);
				}
				100% {
					transform: translate(-50%, -50%) rotate(360deg);
				}
			}
		}
	}
</style>
