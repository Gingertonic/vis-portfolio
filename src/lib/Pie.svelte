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

    svg path {
        opacity: 1;
        transition: opacity 0.3s;
        cursor: pointer;
        outline: none;
    }

    svg path:hover, svg path:focus-visible {
        opacity: 1 !important;
    }

    svg:hover path, svg:focus-visible path {
        opacity: 0.5;
    }

    svg:hover path:hover, svg:hover path:focus-visible {
        opacity: 1 !important;
    }

    .selected {
        --color: orangered !important;

        &:is(path) {
            fill: var(--color);
            opacity: 1 !important;
        }
    }

    path {
        --angle: calc(var(--end-angle) - var(--start-angle));
        --mid-angle: calc(var(--start-angle) + var(--angle) / 2);

        transform: rotate(var(--mid-angle))
	           translateY(0)
	           rotate(calc(-1 * var(--mid-angle)));

        &.selected {
            transform: rotate(var(--mid-angle))
		           translateY(-6px) scale(1.1)
		           rotate(calc(-1 * var(--mid-angle)))
        }
    }
</style>

<script lang="ts">
    import * as d3 from 'd3';

    type DataItem = { value: number, label: number };
    export let data: DataItem[] = [];
    export let selectedIdx = -1;

    let arcGenerator = d3.arc().innerRadius(10).outerRadius(50);
    let colors = d3.scaleOrdinal(d3.schemeTableau10);

    let sliceGenerator = d3.pie<DataItem>().value((d: DataItem) => d.value);
    let arcData;
    let arcs;
    $: arcData = sliceGenerator(data);
    $: arcs = arcData.map<DataItem>((d: DataItem) => arcGenerator(d));

    function toggleWedge (i, e) {
        if (!e.key || e.key === "Enter") {          
            selectedIdx = selectedIdx === i ? -1 : i;
        }
    }
</script>

<div class="container">
    <svg viewBox="-50 -50 100 100">
        {#each arcs as arc, i}
            <path
                style="
                --start-angle: { arcData[i]?.startAngle }rad;
                --end-angle: { arcData[i]?.endAngle }rad;"
                d={arc} 
                fill={colors(i)}
                class:selected={selectedIdx===i}
                on:click={e => toggleWedge(i, e)}
                on:keyup={e => toggleWedge(i, e)}
                tabindex="0" 
                role="button"
                aria-label={`Segment for ${i}`} />
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
