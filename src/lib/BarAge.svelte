<script>
    import { onMount } from "svelte";
    import Papa from "papaparse";
    import { colorSchemes } from "./colorSchemes.js";
    import { base } from "$app/paths";

    let data = [];

    onMount(() => {
        Papa.parse(`${base}/summary-age.csv`, {
            download: true,
            header: true,
            complete: (results) => {
                data = results.data.map((row) => ({
                    key: row.age,
                    value: parseFloat(row.percentage),
                }));
            },
        });
    });

const labels = {
    "0_19": "Youth 0-19",
    "20_39": "Early Career 20-39",
    "40_64": "Late Career 40-64",
    "65plus": "Senior 65+",
};
    
    function getColor(key) {
        const c = colorSchemes.age[key] || colorSchemes.age.default;
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
    style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px; margin-top: 8px;"
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
