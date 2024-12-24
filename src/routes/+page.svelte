<svelte:head>
	<title>Home</title>
</svelte:head>

<script>
	import projects from '$lib/projects.json'; 
    import ProjectsList from "$lib/ProjectsList.svelte";
</script>

<h1>Gingertonic</h1>
<p>Circus musician who ran away with the coders</p>
<img src="images/headshot.jpeg" alt="My actual face">

<section>
	<h2>Github Data</h2>
	{#await fetch("https://api.github.com/users/gingertonic") }
		<p>Loading...</p>
	{:then response}
		{#await response.json()}
			<p>Decoding...</p>
		{:then data}
			<dl>
				<dt>Followers</dt>
				<dd>{data.followers}</dd>
				<dt>Following</dt>
				<dd>{data.following}</dd>
				<dt>Public Repos</dt>
				<dd>{data.public_repos}</dd>
				<dt>Public Gists</dt>
				<dd>{data.public_gists}</dd>
			</dl>
		{:catch error}
			<p class="error">
				Something went wrong: {error.message}
			</p>
		{/await}
	{:catch error}
		<p class="error">
			Something went wrong: {error.message}
		</p>
	{/await}
</section>

<h2>Latest projects</h2>
<ProjectsList data={projects.slice(0,3)} hLevel='3' />