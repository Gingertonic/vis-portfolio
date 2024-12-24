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

    let query = "";
    let filteredProjects;
    $: filteredProjects = projects.filter(project => {
        if (query) {
            let values = Object.values(project).join("\n").toLowerCase();
	        return values.includes(query.toLowerCase());
        }

        return true;
    });

    let selectedYearIdx = -1;
    let selectedYear;
    $: selectedYear = selectedYearIdx > -1 ? pieData[selectedYearIdx].label : null;

    let filteredByYear;
    $: filteredByYear = filteredProjects.filter(project => {
        if (selectedYear) {
            return project.year === selectedYear;
        }

        return true;
    });

    let pieData;

    $: {
        pieData = {};
        let rolledData = d3.rollups(filteredProjects, v => v.length, d => d.year);
        pieData = rolledData.map(([year, count]) => {
            return { value: count, label: year };
        });
    }
</script>

<h1>Projects: {projects.length}</h1>

<Pie data={pieData} bind:selectedIdx={selectedYearIdx} />

<input 
    type="search" 
    bind:value={query}
    aria-label="Search projects" 
    placeholder="🔍 Search projects…" />

<p>Results: {filteredByYear.length}</p>

<ProjectsList data={filteredByYear} />