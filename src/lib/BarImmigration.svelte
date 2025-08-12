<script>
    import { onMount } from "svelte";
    import Papa from "papaparse";
    import { colorSchemes } from "./colorSchemes.js";
    import { base } from "$app/paths";

    let data = [];

    onMount(() => {
        Papa.parse(`${base}/summary-immigration.csv`, {
            download: true,
            header: true,
            complete: (results) => {
                data = results.data.map((row) => ({
                    key: row.immigration,
                    value: parseFloat(row.percentage),
                }));
            },
        });
    });

    const labels = {
        non_immigrant: "Not an immigrant",
        before_1980: "Before 1980",
        "1980_1990": "1980 to 1990",
        "1991_2000": "1991 to 2000",
        "2001_2010": "2001 to 2010",
        "2011_2021": "2011 to 2021",
    };

    function getColor(key) {
        const c =
            colorSchemes.immigration[key] || colorSchemes.immigration.default;
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
