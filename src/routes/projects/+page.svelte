<svelte:head>
	<title>Projects</title>
</svelte:head>

<script>
    import projects from '$lib/projects.json'; 
    import ProjectsList from "$lib/ProjectsList.svelte";
    import Pie from "$lib/Pie.svelte";
</script>

<h1>Projects: {projects.length}</h1>

<Pie />

<ProjectsList data={projects} />