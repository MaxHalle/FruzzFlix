<script>
	import { series, sections } from '$lib/series.js';
	import { base } from '$app/paths';

	function fixBg(str) {
		return str.replace(/url\(\//g, `url(${base}/`);
	}

	const otherSections = sections.filter((s) => s.title !== 'Continue Watching');

	let continueWatching = $state([]);
	let selected = $state(null);
	let unavailable = $state(false);
	let playing = $state(false);

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
		if (!continueWatching.some((s) => s.id === selected.id)) {
			continueWatching = [selected, ...continueWatching];
		}
		if (selected?.video) {
			playing = true;
		} else {
			unavailable = true;
		}
	}

	function stopVideo() {
		playing = false;
	}
</script>

{#if playing && selected?.video}
	<div class="video-overlay">
		<!-- svelte-ignore a11y_media_has_caption -->
		<video src="{base}{selected.video}" autoplay controls onended={stopVideo}></video>
		<button class="close-btn" onclick={stopVideo}>✕</button>
	</div>
{/if}

{#if selected}
	<div
		class="detail"
		role="dialog"
		aria-modal="true"
		tabindex="-1"
		style="background: {fixBg(selected.bg)}"
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
	{#if continueWatching.length > 0}
		<section>
			<h2>Continue Watching</h2>
			<div class="row-wrap">
				<div class="row">
					{#each continueWatching as show (show.id)}
						<button class="card" style="background: {fixBg(show.thumb ?? show.bg)}" onclick={() => open(show)}>
							<div class="card-info">
								<span class="card-title">{show.title}</span>
								<span class="card-genre">{show.genre}</span>
							</div>
						</button>
					{/each}
				</div>
			</div>
		</section>
	{/if}

	{#each otherSections as section (section.title)}
		<section>
			<h2>{section.title}</h2>
			<div class="row-wrap">
				<div class="row">
					{#each section.ids as id (id)}
						{@const show = series.find((s) => s.id === id)}
						{#if show}
							<button class="card" style="background: {fixBg(show.thumb ?? show.bg)}" onclick={() => open(show)}>
								<div class="card-info">
									<span class="card-title">{show.title}</span>
									<span class="card-genre">{show.genre}</span>
								</div>
							</button>
						{/if}
					{/each}
				</div>
			</div>
		</section>
	{/each}
</main>

<style>
	main {
		padding: 32px 48px;
		min-height: calc(100vh - 64px);
	}

	section {
		margin-bottom: 40px;
	}

	h2 {
		margin: 0 0 12px;
		font-size: 1em;
		color: #aaa;
		text-transform: uppercase;
		letter-spacing: 1px;
	}

	/* Clips the row horizontally without affecting vertical hover overflow */
	.row-wrap {
		overflow-x: clip;
		overflow-y: visible;
		/* Right-edge fade signals more content */
		mask-image: linear-gradient(to right, black 82%, transparent 100%);
		-webkit-mask-image: linear-gradient(to right, black 82%, transparent 100%);
	}

	.row {
		display: flex;
		gap: 10px;
		overflow-x: auto;
		scroll-behavior: smooth;
		scrollbar-width: none;
		scroll-snap-type: x mandatory;
		/* Vertical padding so hover scale isn't clipped by row-wrap */
		padding-bottom: 16px;
		margin-bottom: -16px;
	}

	.row::-webkit-scrollbar {
		display: none;
	}

	/* 4 gaps of 10px = 40px; divide remaining width by 4.5 → exactly 4.5 cards visible */
	.card {
		min-width: calc((100% - 40px) / 4.5);
		flex-shrink: 0;
		height: 150px;
		border: none;
		border-radius: 6px;
		cursor: pointer;
		position: relative;
		overflow: hidden;
		transition: transform 0.25s ease, box-shadow 0.25s ease;
		z-index: 1;
		padding: 0;
		scroll-snap-align: start;
	}

	.card:hover {
		transform: scale(1.08);
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
		white-space: pre-line;
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

	.video-overlay {
		position: fixed;
		inset: 0;
		z-index: 300;
		background: #000;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.video-overlay video {
		width: 100%;
		height: 100%;
		object-fit: contain;
	}
</style>
