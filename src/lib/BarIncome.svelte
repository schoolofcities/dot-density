<script>
    import { onMount } from "svelte";
    import Papa from "papaparse";
    import { colorSchemes } from "./colorSchemes.js";
    import { base } from "$app/paths";

    let data = [];

    onMount(() => {
        Papa.parse(`${base}/summary-income.csv`, {
            download: true,
            header: true,
            complete: (results) => {
                data = results.data.map((row) => ({
                    key: row.income,
                    value: parseFloat(row.percentage),
                }));
            },
        });
    });

    const labels = {
        no_income: "Non-working",
        under_30k: "≤ $30,000",
        "30k_60k": "$30,000 to $60,000",
        "60k_90k": "$60,000 to $90,000",
        "90k_150k": "$90,000 to $150,000",
        over_150k: "≥ $150,000",
    };

    function getColor(key) {
        const c = colorSchemes.income[key] || colorSchemes.income.default;
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
