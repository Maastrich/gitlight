<script lang="ts">
	import { onDestroy } from 'svelte';
	import { fetchGithub, formatRelativeDate, getHex, lightenColor } from '~/lib/helpers';
	import { Check, ExternalLink, Pin, Mail, Unpin } from '~/lib/icons';
	import { Button, Tooltip } from '../common';
	import { githubNotifications, settings } from '~/lib/stores';
	import type { NotificationData } from '~/lib/types';

	export let data: NotificationData;
	export let interactive = true;

	let {
		id,
		unread,
		pinned,
		isNew,
		author,
		title,
		description,
		time,
		icon,
		iconColor,
		owner,
		repo,
		number,
		labels,
		url,
		previously
	} = data;
	let displayTime = formatRelativeDate(time);

	const interval = setInterval(() => {
		displayTime = formatRelativeDate(time);
	}, 60000);

	onDestroy(() => {
		clearInterval(interval);
	});

	function markAsReadInGitHub() {
		fetchGithub(`notifications/threads/${id}`, { method: 'PATCH' });
	}

	function handleToggle(key: 'unread' | 'pinned') {
		return () => {
			githubNotifications.update((previous) =>
				previous.map((notification) => {
					if (notification.id !== id) {
						return notification;
					}
					if (key === 'pinned' && !notification.pinned && $settings.readWhenPin) {
						return { ...notification, pinned: !notification.pinned, unread: false };
					}
					return { ...notification, [key]: !notification[key] };
				})
			);

			if (pinned) {
				unread = !unread;
			}

			if (unread) {
				markAsReadInGitHub();
			}
		};
	}

	function handleOpenInBrowser() {
		if ($settings.readWhenOpenInBrowser) {
			githubNotifications.update((previous) =>
				previous.map((event) => {
					if (event.id === id) {
						if (event.unread) {
							markAsReadInGitHub();
						}
						return { ...event, unread: false };
					}
					return event;
				})
			);
		}
	}
</script>

<div class="container" class:unread>
	<div
		class="notification"
		on:mouseenter={isNew && interactive ? () => (isNew = false) : undefined}
	>
		{#if isNew && unread}
			<div class="new" />
		{/if}
		<div class="top">
			<p class="repo">{owner}/<span class="bold">{repo}</span></p>
			<p class="time">{displayTime}</p>
		</div>
		<p class="description">
			{#if author}
				<span class="strong">{author.login}</span>
				{#if author.avatar}
					<img class="image" src={author.avatar} alt="" width="20px" height="20px" loading="lazy" />
				{/if}
				{description}
			{:else}
				<span class="strong">{description}</span>
			{/if}
		</p>
		<div class="main">
			<span class="icon-container" style:color={getHex(iconColor)}>
				<svelte:component this={icon} />
			</span>
			<h3 class="title">{title}</h3>
			{#if number}
				<span class="number">#{number}</span>
			{/if}
		</div>
		{#if labels && labels.length}
			<ul class="labels">
				{#each labels as label}
					<li class="label" style:color={lightenColor(label.color)}>
						{label.name}
						<div class="label-background" style:background-color={`#${label.color}`} />
					</li>
				{/each}
			</ul>
		{/if}
		{#if interactive}
			<div class="over">
				<Tooltip content="Mark as {unread ? '' : 'un'}read" position="left">
					<Button type={unread ? 'primary' : 'secondary'} small on:click={handleToggle('unread')}>
						{#if unread}
							<Check />
						{:else}
							<Mail />
						{/if}
					</Button>
				</Tooltip>
				{#if url}
					<Tooltip content="Open in GitHub" position="left">
						<Button type="secondary" small href={url} external on:click={handleOpenInBrowser}>
							<ExternalLink />
						</Button>
					</Tooltip>
				{:else}
					<Tooltip content="Cannot open in GitHub" position="left">
						<Button type="secondary" small disabled>
							<ExternalLink />
						</Button>
					</Tooltip>
				{/if}
				<Tooltip content={pinned ? 'Unpin' : 'Pin'} position="left">
					<Button type="secondary" small on:click={handleToggle('pinned')}>
						{#if pinned}
							<Unpin />
						{:else}
							<Pin />
						{/if}
					</Button>
				</Tooltip>
			</div>
		{/if}
	</div>
	{#if previously}
		<div class="previously">
			<div class="description">
				<span>Previously: </span>
				{#if previously.author}
					<span class="strong">{previously.author.login}</span>
					{#if previously.author.avatar}
						<img
							class="image"
							src={previously.author.avatar}
							alt=""
							width="20px"
							height="20px"
							loading="lazy"
						/>
					{/if}
				{/if}
				{previously.description}
			</div>
		</div>
	{/if}
</div>

<style lang="scss">
	.container {
		&:not(:hover) {
			.over {
				opacity: 0;
			}
		}

		&:not(.unread) {
			opacity: 0.65;
			transition: opacity variables.$transition;

			&:hover {
				opacity: 1;
			}
		}
	}

	.notification {
		background-color: variables.$grey-2;
		border: 1px solid variables.$grey-3;
		border-radius: variables.$radius;
		padding: 1rem;
		position: relative;
		display: flex;
		flex-direction: column;
		gap: 0.75rem;
	}

	.new {
		width: 0.5rem;
		height: 0.5rem;
		border-radius: 50%;
		background-color: variables.$blue-2;
		position: absolute;
		inset: 0.25rem 0 0 0.25rem;
		box-shadow: 0 0 0.5rem variables.$grey-2;
	}

	.top {
		@include typography.small;
		display: flex;
		justify-content: space-between;
		color: variables.$grey-4;

		.repo .bold {
			@include typography.bold;
		}
	}

	.main {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		width: 100%;

		.icon-container {
			flex: 0 0 2rem;

			:global(svg) {
				width: 2rem;
				height: 2rem;
			}
		}

		.title {
			@include typography.bold;
			flex: 0 1 auto;
			white-space: nowrap;
			overflow: hidden;
			text-overflow: ellipsis;
		}

		.number {
			color: variables.$blue-3;
		}
	}

	.labels {
		display: flex;
		flex-wrap: wrap;
		gap: 0.5rem;

		.label {
			@include typography.small;
			padding: 0.25rem 0.5rem;
			border-radius: variables.$radius;
			border: 1px solid;
			position: relative;

			.label-background {
				position: absolute;
				inset: 0;
				border-radius: variables.$radius;
				opacity: 0.1;
			}
		}
	}

	.over {
		position: absolute;
		inset: 0 0 0 auto;
		width: 12rem;
		background-image: linear-gradient(to right, transparent, variables.$grey-1);
		border-radius: 0 calc(variables.$radius - 1px) calc(variables.$radius - 1px) 0;
		display: flex;
		flex-direction: column;
		padding: 0.5rem;
		align-items: end;
		justify-content: start;
		gap: 0.5rem;
		transition: opacity variables.$transition;
	}

	.description {
		width: 100%;
		color: variables.$grey-4;
		line-height: 1.3em;
		display: -webkit-box;
		overflow: hidden;
		-webkit-line-clamp: 2;
		-webkit-box-orient: vertical;
		word-wrap: break-word;

		.strong {
			color: variables.$white;
		}

		.image {
			display: inline;
			vertical-align: sub;
			border-radius: 50%;
		}
	}

	.previously {
		padding: 0.75rem 1rem;
		position: relative;
		z-index: -1;

		&::before,
		&::after {
			content: '';
			position: absolute;
			inset: -0.5rem 0 0;
			z-index: -1;
		}

		&::before {
			background-color: variables.$grey-2;
			border: 1px solid variables.$grey-3;
			border-radius: 0 0 variables.$radius variables.$radius;
		}

		&::after {
			background-image: linear-gradient(rgba(variables.$grey-1, 0.75), transparent);
		}
	}
</style>
