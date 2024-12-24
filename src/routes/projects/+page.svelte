<svelte:head>
	<title>Projects</title>
</svelte:head>

<script>
    import projects from '$lib/projects.json'; 
    import ProjectsList from "$lib/ProjectsList.svelte";   
</script>

<h1>Projects: {projects.length}</h1>

<ProjectsList data={projects} />