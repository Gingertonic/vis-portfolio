<style>
	svg {
		overflow: visible;
	}
    .gridlines {
        stroke-opacity: .2;
    }
</style>

<script lang="ts">
    import Stats from "$lib/Stats.svelte";
    import * as d3 from "d3";
    import { onMount } from "svelte";
    import type { StatItem } from "../../types";

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
    let xAxis: SVGGElement
    let yAxis: SVGGElement;
    let yAxisGridlines: SVGGElement;
    let colorScale: d3.ScaleLinear<string, string>;

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

    })

    let files: string[] = [];
    let mostLines: number;
    let activePeriod: string;

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
        colorScale = d3.scaleLinear<string>().domain([0,24]).range(["pink", "royalBlue"])

    }
</script>

<h1>Meta</h1>

<Stats data={stats} />

<p>Commits by time of day</p>
<svg viewBox="0 0 {width} {height}">
    <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
    <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />
    <g class="gridlines" transform="translate({usableArea.left}, 0)" bind:this={yAxisGridlines} />
    <g class="dots">
        {#each commits as commit, index }
            <circle
                cx={ xScale(commit.datetime) }
                cy={ yScale(commit.hourFrac) }
                r="5"
                fill={ colorScale(commit.hourFrac) }
            />
        {/each}
    </g>
</svg>
