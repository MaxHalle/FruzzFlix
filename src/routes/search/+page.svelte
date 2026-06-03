<script>
	import { series } from '$lib/series.js';

	let query = $state('');
	let selected = $state(null);
	let unavailable = $state(false);

	let results = $derived(
		query.trim().length === 0
			? []
			: series.filter(
					(s) =>
						s.title.toLowerCase().includes(query.toLowerCase()) ||
						s.genre.toLowerCase().includes(query.toLowerCase()) ||
						s.creator.toLowerCase().includes(query.toLowerCase()) ||
						s.starring.toLowerCase().includes(query.toLowerCase())
				)
	);

	function open(show) {
		selected = show;
		unavailable = false;
		document.body.style.overflow = 'hidden';
	}

	function close() {
		selected = null;
		unavailable = false;
		document.body.style.overflow = '';
	}

	function play() {
		if (selected?.video) {
			// future: launch video player
		} else {
			unavailable = true;
		}
	}
</script>

{#if selected}
	<div
		class="detail"
		role="dialog"
		aria-modal="true"
		tabindex="-1"
		style="background: {selected.bg}"
		onkeydown={(e) => e.key === 'Escape' && close()}
	>
		<div class="detail-fade"></div>
		<div class="detail-content">
			<p class="detail-genre">{selected.genre}</p>
			<h1 class="detail-title">{selected.title}</h1>
			<div class="detail-meta">
				<span>{selected.year}</span>
				<span class="pill">{selected.rating}</span>
				<span>{selected.seasons}</span>
			</div>
			<p class="detail-desc">{selected.description}</p>
			<p class="detail-cast"><strong>Starring</strong> {selected.starring}</p>
			<p class="detail-cast"><strong>Creator</strong> {selected.creator}</p>
			<div class="actions">
				<button class="play-btn" onclick={play}>
					<span class="play-icon">▶</span> Play
				</button>
			</div>
			{#if unavailable}
				<p class="unavailable">Content is currently unavailable.</p>
			{/if}
		</div>
		<button class="close-btn" onclick={close}>✕</button>
	</div>
{/if}

<main>
	<div class="search-bar">
		<input
			type="text"
			placeholder="Search titles, genres, creators..."
			bind:value={query}
			autofocus
		/>
	</div>

	{#if query.trim().length > 0}
		{#if results.length > 0}
			<div class="results-header">
				{results.length} result{results.length !== 1 ? 's' : ''} for &ldquo;{query}&rdquo;
			</div>
			<div class="grid">
				{#each results as show (show.id)}
					<button class="card" style="background: {show.thumb ?? show.bg}" onclick={() => open(show)}>
						<div class="card-info">
							<span class="card-title">{show.title}</span>
							<span class="card-genre">{show.genre}</span>
						</div>
					</button>
				{/each}
			</div>
		{:else}
			<p class="empty">No results for &ldquo;{query}&rdquo;</p>
		{/if}
	{/if}
</main>

<style>
	main {
		padding: 40px 48px;
		min-height: calc(100vh - 64px);
	}

	.search-bar {
		display: flex;
		justify-content: center;
		margin-bottom: 40px;
	}

	input {
		width: 100%;
		max-width: 560px;
		padding: 16px 24px;
		font-size: 1.1em;
		border: none;
		border-bottom: 2px solid #2563eb;
		background: transparent;
		color: #fff;
		outline: none;
		text-align: center;
	}

	input::placeholder {
		color: #555;
	}

	.results-header {
		font-size: 0.82em;
		color: #888;
		text-transform: uppercase;
		letter-spacing: 1px;
		margin-bottom: 16px;
	}

	.grid {
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
		gap: 10px;
	}

	.card {
		height: 150px;
		border: none;
		border-radius: 6px;
		cursor: pointer;
		position: relative;
		overflow: hidden;
		transition: transform 0.25s ease, box-shadow 0.25s ease;
		z-index: 1;
		padding: 0;
	}

	.card:hover {
		transform: scale(1.07);
		z-index: 10;
		box-shadow: 0 12px 40px rgba(0, 0, 0, 0.8);
	}

	.card-info {
		position: absolute;
		inset: 0;
		display: flex;
		flex-direction: column;
		justify-content: flex-end;
		padding: 10px 12px;
		background: linear-gradient(to top, rgba(0, 0, 0, 0.85) 0%, transparent 60%);
		opacity: 0;
		transition: opacity 0.2s ease;
	}

	.card:hover .card-info {
		opacity: 1;
	}

	.card-title {
		font-size: 0.85em;
		font-weight: bold;
		color: #fff;
		line-height: 1.2;
	}

	.card-genre {
		font-size: 0.72em;
		color: #aaa;
		margin-top: 2px;
	}

	.empty {
		color: #666;
		font-size: 0.95em;
		text-align: center;
		margin-top: 60px;
	}

	/* ── Detail overlay ── */

	.detail {
		position: fixed;
		inset: 0;
		z-index: 200;
		display: flex;
		align-items: center;
	}

	.detail-fade {
		position: absolute;
		inset: 0;
		background: linear-gradient(to right, rgba(0, 0, 0, 0.92) 0%, rgba(0, 0, 0, 0.6) 45%, rgba(0, 0, 0, 0.1) 100%);
	}

	.detail-content {
		position: relative;
		z-index: 1;
		max-width: 520px;
		padding: 64px 64px 64px 72px;
	}

	.detail-genre {
		margin: 0 0 10px;
		font-size: 0.8em;
		text-transform: uppercase;
		letter-spacing: 2px;
		color: #2563eb;
	}

	.detail-title {
		margin: 0 0 18px;
		font-size: 3.2em;
		font-weight: 900;
		line-height: 1;
		color: #fff;
		letter-spacing: -1px;
	}

	.detail-meta {
		display: flex;
		align-items: center;
		gap: 12px;
		margin-bottom: 20px;
		font-size: 0.85em;
		color: #ccc;
	}

	.pill {
		border: 1px solid #aaa;
		padding: 1px 8px;
		border-radius: 4px;
		font-size: 0.9em;
		color: #aaa;
	}

	.detail-desc {
		margin: 0 0 20px;
		font-size: 0.95em;
		line-height: 1.6;
		color: #ddd;
		max-width: 440px;
	}

	.detail-cast {
		margin: 0 0 6px;
		font-size: 0.82em;
		color: #aaa;
	}

	.detail-cast strong {
		color: #fff;
		margin-right: 4px;
	}

	.close-btn {
		position: absolute;
		top: 24px;
		right: 28px;
		background: rgba(0, 0, 0, 0.5);
		border: 1px solid #444;
		color: #fff;
		width: 40px;
		height: 40px;
		border-radius: 50%;
		font-size: 1em;
		cursor: pointer;
		display: flex;
		align-items: center;
		justify-content: center;
		transition: background-color 0.15s;
		z-index: 1;
	}

	.close-btn:hover {
		background: rgba(37, 99, 235, 0.6);
	}

	.actions {
		margin-top: 28px;
	}

	.play-btn {
		display: inline-flex;
		align-items: center;
		gap: 10px;
		padding: 12px 32px;
		background: #fff;
		color: #000;
		border: none;
		border-radius: 6px;
		font-size: 1em;
		font-weight: 700;
		cursor: pointer;
		transition: background 0.15s, transform 0.15s;
	}

	.play-btn:hover {
		background: #e0e0e0;
		transform: scale(1.04);
	}

	.play-icon {
		font-size: 0.85em;
	}

	.unavailable {
		margin-top: 14px;
		font-size: 0.85em;
		color: #f87171;
	}
</style>
