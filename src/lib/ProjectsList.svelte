<script>
    import Project from "$lib/Project.svelte";  
    export let data = [{ title: '', image: '', description: '' }];
    export let hLevel = '2';
</script>

<style>
    .projects {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(15em, 1fr));
        grid-gap: 1em;
    }
</style>

<div class="projects">
    {#each data as p}
        <Project data={p} hLevel={hLevel} />
    {/each}
</div>