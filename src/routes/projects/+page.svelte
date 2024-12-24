<svelte:head>
	<title>Projects</title>
</svelte:head>

<script>
    import * as d3 from 'd3';
    import projects from '$lib/projects.json'; 
    import ProjectsList from "$lib/ProjectsList.svelte";
    import Pie from "$lib/Pie.svelte";

    // let pieData = [
    //     { value: 1, label: "apples" },
    //     { value: 2, label: "oranges" },
    //     { value: 3, label: "mangos" },
    //     { value: 4, label: "pears" },
    //     { value: 5, label: "limes" },
    //     { value: 5, label: "cherries" }
    // ];

    let rolledData = d3.rollups(projects, v => v.length, d => d.year);
    let pieData = rolledData.map(([year, count]) => {
        return { value: count, label: year };
    });
</script>

<h1>Projects: {projects.length}</h1>

<Pie data={pieData}/>

<ProjectsList data={projects} />