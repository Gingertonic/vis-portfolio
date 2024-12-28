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
        console.log('data[4]', data[4]);
        
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
        const rollupPeriods = d3.rollups(
            data,
            d => d.length,
            d => d.datetime.toLocaleString("en", {dayPeriod: "short"})
        );
        activePeriod = d3.greatest(rollupPeriods, (p) => p[1])?.[0] ?? '';    
        files = [...new Set(d3.map(data, (d) => d.file))];
        mostLines = d3.max(commits, c => c.totalLines) ?? 0;

        stats = [
            { title: 'Lines of code', value: totalLines ?? 0 },
            { title: 'Total Files', value: files.length },
            { title: 'Most Active Period', value: activePeriod },
            { title: 'Most Lines in Commit', value: mostLines }
        ]
    }    
</script>

<h1>Meta</h1>

<Stats data={stats} />
