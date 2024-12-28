<svelte:head>
	<title>Home</title>
</svelte:head>

<script lang="ts">
	import projects from '$lib/projects.json'; 
    import ProjectsList from "$lib/ProjectsList.svelte";
	import Stats from '$lib/Stats.svelte';

	import { onMount } from 'svelte';
    import type { StatItem } from '../types';
	let resp: Response;
	let stats: StatItem[] = []
	let error: Error | null = null;

	onMount(async () => {
		try {
			resp = await fetch('https://api.github.com/users/gingertonic')
			const data = await resp.json();
			stats = [
				{ title: 'Followers', value: data.followers },
				{ title: 'Following', value: data.following },
				{ title: 'Public Repos', value: data.public_repos },
				{ title: 'Public Gists', value: data.public_gists }
			]
		} catch (e) {
            error = e instanceof Error ? e : new Error('Unknown error occurred');
		}
	});
</script>

<h1>Gingertonic</h1>
<p>Circus musician who ran away with the coders</p>
<img src="images/headshot.jpeg" alt="My actual face">

<section>
	<h2>Github Data</h2>
	{#if !resp }
		<p>Loading...</p>
	{:else if !stats}
		<p>Decoding...</p>
	{:else if stats}
		<Stats data={stats} />
	{:else}
		<p class="error">
			Something went wrong: {error?.message ?? 'Unknown error'}
		</p>
	{/if}
</section>

<h2>Latest projects</h2>
<ProjectsList data={projects.slice(0,3)} hLevel='3' />