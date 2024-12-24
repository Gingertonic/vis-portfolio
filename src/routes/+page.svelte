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

<h2>Latest projects</h2>
<ProjectsList data={projects.slice(0,3)} hLevel='3' />