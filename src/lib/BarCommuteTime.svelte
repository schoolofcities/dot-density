<script>
    import { onMount } from "svelte";
    import Papa from "papaparse";
    import { colorSchemes } from "./colorSchemes.js";
    import { base } from "$app/paths";

    let data = [];

    onMount(() => {
        Papa.parse(`${base}/summary-commute_time.csv`, {
            download: true,
            header: true,
            complete: (results) => {
                data = results.data.map((row) => ({
                    key: row.commute_time,
                    value: parseFloat(row.percentage),
                }));
            },
        });
    });

    const labels = {
        "0_15": "≤ 15 minutes",
        "15_29": "15 to 29 minutes",
        "30_44": "30 to 44 minutes",
        "45_59": "45 to 59 minutes",
        "60plus": " ≥ 60 minutes",
        unknown: "No data",
    };

    function getColor(key) {
        const c =
            colorSchemes.commute_time[key] || colorSchemes.commute_time.default;
        return `rgb(${c[0]},${c[1]},${c[2]})`;
    }
</script>

<div
    style="display: flex; width: 100%; height: 8px; padding-top: 15px; overflow: hidden"
>
    {#each data as d (d.key)}
        {#if d.value > 0}
            <div
                title={labels[d.key] + ": " + d.value.toFixed(1) + "%"}
                style="
                    background: {getColor(d.key)};
                    width: {d.value}%;
                    height: 100%;
                    transition: width 0.3s;
                "
            ></div>
        {/if}
    {/each}
</div>

<div
    style="display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; margin-top: 8px;"
>
    {#each data as d, i (d.key)}
        {#if d.value > 0}
            <span style="display: flex; align-items: center; font-size: 0.8em;">
                <span
                    style="display:inline-block;width:10px;height:10px;background:{getColor(
                        d.key,
                    )};margin-right:5px;border-radius:10px;"
                ></span>
                {labels[d.key]} ({d.value.toFixed(1)}%)
            </span>
        {/if}
    {/each}
</div>
