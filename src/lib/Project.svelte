<script lang="ts">
    import type { ProjectData } from '$lib/../types';
    
    export let data: ProjectData = { title: '', image: '', description: '', year: 2024 };
    export let hLevel = '2';
</script>

<style>
    article {
        display: grid;
        grid-template-rows: subgrid;
        grid-row: span 3;
    }

    article h2, article h3 {
        margin: 0;
    }

    p.year {
        font-family: Baskerville;
        font-variant-numeric: oldstyle-nums;
    }
</style>

<article>
    <svelte:element this={"h" + hLevel}>{ data.title }</svelte:element>
    <img src={data.image} alt={`Preview of ${data.title}`}>
    <div>
        <p>{data.description}</p>
        <p class="year">{data.year}</p>
    </div>
</article>