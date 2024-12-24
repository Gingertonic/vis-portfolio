<style>
    svg {
        max-width: 20em;
        margin-block: 2em;

        /* Do not clip shapes outside the viewBox */
        overflow: visible;
    }

    .container {
        display: flex;
        gap: 30px;
        align-items: center;
    }

    .legend {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(9em, 1fr));
        gap: 30px;
        border: 2px solid orangered;
        padding: 30px;
        flex: 1;
        max-height: 80%;
    }

    .legend li {
        list-style: none;
        display: flex;
        align-items: center;
        gap: 10px;
    }

    .swatch {
        height: 20px;
        aspect-ratio: 1/1;
        background-color: var(--color);
        border-radius: 50%;
        display: inline-block;
    }
</style>

<script>
    import * as d3 from 'd3';
    export let data = [];
    
    let arcGenerator = d3.arc().innerRadius(10).outerRadius(50);
    
    // let data = [
    //     { value: 1, label: "apples" },
    //     { value: 2, label: "oranges" },
    //     { value: 3, label: "mangos" },
    //     { value: 4, label: "pears" },
    //     { value: 5, label: "limes" },
    //     { value: 5, label: "cherries" }
    // ];

    let colors = d3.scaleOrdinal(d3.schemeTableau10);

    let sliceGenerator = d3.pie().value(d => d.value);
    let arcData = sliceGenerator(data);
    let arcs = arcData.map(d => arcGenerator(d));
</script>

<div class="container">
    <svg viewBox="-50 -50 100 100">
        {#each arcs as arc, i}
            <path d={arc} fill={colors(i)} />
        {/each}
    </svg>
    
    <ul class="legend">
        {#each data as d, index}
            <li style="--color: { colors(index) }">
                <span class="swatch"></span>
                {d.label} <em>({d.value})</em>
            </li>
        {/each}
    </ul>
</div>
