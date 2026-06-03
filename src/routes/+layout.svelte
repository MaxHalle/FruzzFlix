<script>
	import favicon from '$lib/assets/favicon.svg';
	import { resolve } from '$app/paths';
	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
</svelte:head>

<nav>
	<a href={resolve('/')} class="logo">FruzzFlix</a>
	<div class="nav-links">
		<a href={resolve('/')}>Home</a>
		<a href={resolve('/search')}>Search</a>
	</div>
</nav>

{@render children()}

<style>
	:global(*, *::before, *::after) {
		box-sizing: border-box;
	}

	:global(body) {
		margin: 0;
		background-color: #000;
		color: #fff;
		font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
	}

	nav {
		position: sticky;
		top: 0;
		z-index: 100;
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 0 48px;
		height: 64px;
		background-color: rgba(0, 0, 0, 0.95);
		border-bottom: 1px solid #222;
	}

	.logo {
		font-size: 1.6em;
		font-weight: 900;
		color: #2563eb;
		text-decoration: none;
		letter-spacing: 1px;
	}

	.nav-links {
		display: flex;
		gap: 28px;
		align-items: center;
	}

	.nav-links a {
		color: #ccc;
		text-decoration: none;
		font-size: 0.9em;
		transition: color 0.15s;
	}

	.nav-links a:hover {
		color: #fff;
	}
</style>
