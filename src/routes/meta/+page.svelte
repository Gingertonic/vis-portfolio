<svelte:head>
	<title>Meta</title>
</svelte:head>

<style>
	svg {
		overflow: visible;
	}
    .gridlines {
        stroke-opacity: .2;
    }
    dl.info {
        display: grid;
        grid-template-columns: repeat(2, 1fr);
        grid-template-rows: repeat(2, 1fr);
        grid-column-gap: 10px;
        grid-row-gap: 5px;
        padding: 10px;
        transition-duration: 500ms;
        transition-property: opacity, visibility;

        &[hidden]:not(:hover, :focus-within) {
            opacity: 0;
            visibility: hidden;
        }
        font-size: 60%;
    }
    .tooltip {
        position: fixed;
        top: 1em;
        left: 1em;
        background-color: rgba(45,45,100,0.6);
        backdrop-filter: blur(4px);
        width: 400px;
        height: 100px;
        box-shadow: 2px 2px orange;
    }
    circle {
        transition: 200ms;
        transform-origin: center;
        transform-box: fill-box;
        &:hover {
            transform: scale(1.5);
        }
        &:not(:hover) {
            fill-opacity: 0.4;
        }
    }
    circle.selected {
        fill: orange;
    }
    .dot-container {
        cursor: pointer;
        outline: none;
    }
    .dot-container:focus circle {
        stroke: orange;
        stroke-width: 2px;
    }
    @keyframes marching-ants {
        to {
            stroke-dashoffset: -2 /* 5 + 3 */
        }
    }

    svg :global(.selection) {
        fill-opacity: 10%;
        stroke: orange;
        stroke-opacity: 70%;
        stroke-dasharray: 1 6;
        animation: marching-ants 0.5s linear infinite alternate-reverse;
    }
</style>

<script lang="ts">
    import Stats from "$lib/Stats.svelte";
    import * as d3 from "d3";
    import { onMount } from "svelte";
    import type { StatItem } from "../../types";
    import {
        computePosition,
        autoPlacement,
        offset,
        type ComputePositionReturn,
    } from '@floating-ui/dom';
    import Pie from "$lib/Pie.svelte";

    interface CommitData {
        commit: string;
        author: string;
        date: Date; 
        datetime: Date
        depth: number; 
        file: string;
        length: number; 
        line: number; 
        time: string;
        timezone: string;
        type: string;
    }
    interface CommitInstance {
        id: string;
        url: string;
        author: string;
        date: Date;
        time: string;
        timezone: string;
        datetime: Date;
        hourFrac: number;
        totalLines: number;
        lines?: CommitData[];
    }
    let data: d3.DSVParsedArray<CommitData>;
    let totalLines: number;
    let commits: CommitInstance[] = [];
    let stats: StatItem[] = [];

    let width = 1000, height = 600; 
    let margin = {top: 10, right: 10, bottom: 30, left: 20};
    let usableArea = {
        top: margin.top,
        right: width - margin.right,
        bottom: height - margin.bottom,
        left: margin.left,
        width: 0,
        height: 0
    };
    usableArea.width = usableArea.right - usableArea.left;
    usableArea.height = usableArea.bottom - usableArea.top;
    let yScale: d3.ScaleLinear<number, number>;
    let xScale: d3.ScaleTime<number, number>;
    let rScale: d3.ScaleLinear<number, number>;
    let xAxis: SVGGElement
    let yAxis: SVGGElement;
    let yAxisGridlines: SVGGElement;
    let colorScale: d3.ScaleLinear<string, string>;

    let commitTooltip: HTMLElement;
    let tooltipPosition: ComputePositionReturn = {
        x: 0, y: 0,
        placement: "top",
        strategy: "fixed",
        middlewareData: {}
    };
    let svg: SVGGElement;

    let brushSelection: any[][] | null;

    $: selectedCommits = brushSelection ? commits.filter(isCommitSelected) : [];
    $: hasSelection = brushSelection && selectedCommits.length > 0;
    $: selectedLines = (hasSelection ? selectedCommits : commits).flatMap(d => d.lines);
    $: languageBreakdown = d3.rollups(
        selectedLines?.filter((d): d is CommitData => d !== undefined) ?? [],
        group => group.length,
        d => d.type
    );
    $: {
        console.log('Language breakdown:', languageBreakdown);
    }
    async function dotInteraction (index: number, e: Event) {
        let hoveredDot = e.target;
        hoveredCommitIdx = index;
        if (e.type === "mouseenter" || e.type === "focus") {
            tooltipPosition = await computePosition(hoveredDot as HTMLElement, commitTooltip, {
                strategy: "fixed", // because we use position: fixed
                middleware: [
                    offset(5), // spacing from tooltip to dot
                    autoPlacement() // see https://floating-ui.com/docs/autoplacement
                ],
            });
        }
        else if (e.type === "mouseleave" || e.type === "blur") {
            hoveredCommitIdx = -1;
        }
    }

    function brushed(e: d3.D3BrushEvent<unknown>) {
        brushSelection = e.selection as [[number, number], [number, number]] | null;
    }

    function isCommitSelected (commit: CommitInstance) {
        if (!brushSelection) return false;
        let min = {x: brushSelection[0][0], y: brushSelection[0][1]};
        let max = {x: brushSelection[1][0], y: brushSelection[1][1]};
        let x = xScale(commit.datetime);
        let y = yScale(commit.hourFrac);
        return x >= min.x && x <= max.x && y >= min.y && y <= max.y;
    }

    yScale = d3.scaleLinear()
        .domain([0, 24])
        .range([0, 0]); // temporary range

    xScale = d3.scaleTime()
        .domain([new Date(), new Date()])
        .range([0, 0]); // temporary range

    // Only update axes when scales and DOM elements exist
    $: if (xScale && yScale && xAxis && yAxis) {
        d3.select(xAxis).call(d3.axisBottom(xScale));
        d3.select(yAxis).call(d3.axisLeft(yScale));
        d3.select(yAxis).call(d3.axisLeft(yScale).tickFormat(d => String(Number(d) % 24).padStart(2, "0") + ":00"));
        d3.select(yAxisGridlines).call(
        d3.axisLeft(yScale)
            .tickFormat(() => "")
            .tickSize(-usableArea.width)
        );
    }

    $: {
        // activate brush
        d3.select(svg).call(d3.brush().on("start brush end", brushed));
        // keep tooltips available by raising above brush overlay
        d3.select(svg).selectAll(".dots, .overlay ~ *").raise();
    }

    onMount(async() => {
        data = await d3.csv<CommitData>("loc.csv", row => ({
            commit: row.commit,
            author: row.author,
            file: row.file,
            type: row.type,
            time: row.time,
            timezone: row.timezone,
            line: Number(row.line),
            depth: Number(row.depth),
            length: Number(row.length),
            date: new Date(row.date + "T00:00" + row.timezone),
            datetime: new Date(row.datetime)
        }));
        
        totalLines = data?.length ?? null;  
        commits = d3.groups(data, d => d.commit).map(([commit, lines]) => {
            let first = lines[0];
            let {author, date, time, timezone, datetime} = first;
            let ret = {
                id: commit,
                url: "https://github.com/vis-society/lab-7/commit/" + commit,
                author, date, time, timezone, datetime,
                hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
                totalLines: lines.length
            };

            // Like ret.lines = lines
            // but non-enumerable so it doesn’t show up in JSON.stringify
            Object.defineProperty(ret, "lines", {
                value: lines,
                configurable: true,
                writable: true,
                enumerable: false,
            });

            return ret;
        });
        commits = d3.sort(commits, d => -d.totalLines);
    })

    let files: string[] = [];
    let mostLines: number;
    let activePeriod: string;

    let hoveredCommitIdx = -1;
    let hoveredCommit: Partial<CommitInstance> = {};
    $: hoveredCommit = commits[hoveredCommitIdx] ?? {};

    $: if (data) {
        // Create stats block
        const rollupPeriods = d3.rollups(
            data,
            d => d.length,
            d => d.datetime.toLocaleString("en", {dayPeriod: "short"})
        );
        activePeriod = d3.greatest(rollupPeriods, (p) => p[1])?.[0] ?? '';    
        files = [...new Set(d3.map(data, (d) => d.file))];
        mostLines = d3.max(commits, c => c.totalLines) ?? 0;

        stats = [
            { title: 'Commits', value: commits.length },
            { title: 'Lines of code', value: totalLines ?? 0 },
            { title: 'Total Files', value: files.length },
            { title: 'Most Active Period', value: activePeriod },
            { title: 'Most Lines in Commit', value: mostLines }
        ]
        
        // Create scatter chart
        yScale = d3.scaleLinear([24, 0], [usableArea.bottom, usableArea.top]);
        const dtsMinMax = d3.extent(d3.map(commits, c => c.datetime)) as Date[];
        xScale = d3.scaleTime(dtsMinMax, [usableArea.left, usableArea.right]);
        colorScale = d3.scaleLinear<string>().domain([0,24]).range(["orangered", "royalBlue"])

        const linesMinMax = d3.extent(d3.map(commits, c => c.totalLines)) as number[];        
        rScale = d3.scaleSqrt<number>(linesMinMax, [2, 30])
    }
</script>

<h1>Meta</h1>

<Stats data={stats} />
<p>{hasSelection ? selectedCommits.length : "No"} commit(s) selected</p>

<p>Commits by time of day</p>
<svg viewBox="0 0 {width} {height}" bind:this={svg}>
    <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
    <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />
    <g class="gridlines" transform="translate({usableArea.left}, 0)" bind:this={yAxisGridlines} />
    <g class="dots">
        {#each commits as commit, index }
            <g 
                role="button"
                tabindex="0"
                class="dot-container"
                on:mouseenter={e => dotInteraction(index, e)}
                on:mouseleave={e => dotInteraction(index, e)}
                on:focus={e => dotInteraction(index, e)}
                on:blur={e => dotInteraction(index, e)}
                on:keydown={e => e.key === 'Enter' && dotInteraction(index, e)}
                aria-describedby="commit-tooltip"
            >
                <circle
                    class:selected={isCommitSelected(commit)}
                    cx={xScale(commit.datetime)}
                    cy={yScale(commit.hourFrac)}
                    r={rScale(commit.totalLines)}
                    fill={colorScale(commit.hourFrac)}
                />
            </g>
        {/each}
    </g>
</svg>
<dl id="commit-tooltip" class="info tooltip" 
    bind:this={commitTooltip}
    hidden={hoveredCommitIdx === -1}
    style="top: {tooltipPosition.y}px; left: {tooltipPosition.x}px"
>
    <dt>Commit</dt>
    <dd><a href="{ hoveredCommit.url }" target="_blank">{ hoveredCommit.id }</a></dd>

    <dt>Date</dt>
    <dd>{ hoveredCommit.datetime?.toLocaleString("en", {dateStyle: "full"}) }</dd>
   
    <dt>Time</dt>
    <dd>{ hoveredCommit.time }</dd>
    
    <dt># Lines</dt>
    <dd>{ hoveredCommit.totalLines }</dd>

    <!-- Add: Time, author, lines edited -->
</dl>

<Stats data={languageBreakdown.map(l => ({
        title: l[0], 
        value: `${d3.format(".1f")(selectedLines.length / l[1])}%`
    }))} />

<Pie data={languageBreakdown.map(l => ({ value: l[1], label: l[0]}))} />