<script>
    import { colorSchemes } from "./colorSchemes.js";

    const data = [
        { key: "w", value: 45.83 },
        { key: "sa", value: 14.01 },
        { key: "ea", value: 12.46 },
        { key: "b", value: 9.55 },
        { key: "sea", value: 7.59 },
        { key: "la", value: 3.14 },
        { key: "wa", value: 3.42 },
        { key: "m", value: 3.29 },
        // { key: "o", value: 1.23 },
        // { key: "a", value: 1.19 },
        { key: "i", value: 0.66 },
        // { key: "unknown", value: 0.04 }
    ];

    const labels = {
        w: "White",
        sa: "South Asian",
        ea: "East Asian",
        b: "Black",
        sea: "Southeast Asian",
        la: "Latin American",
        wa: "Arab / West Asian",
        m: "Mixed / Other",
        // o: "Other",
        // a: "Arab",
        i: "Indigenous",
        // unknown: "Unknown"
    };

    function getColor(key) {
        const c = colorSchemes.ethnicity[key] || colorSchemes.ethnicity.default;
        return `rgb(${c[0]},${c[1]},${c[2]})`;
    }
</script>

<div style="display: flex; width: 100%; height: 8px; padding-top: 15px; overflow: hidden">
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

<div style="display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; margin-top: 8px;">
    {#each data as d, i (d.key)}
        {#if d.value > 0}
            <span style="display: flex; align-items: center; font-size: 0.8em;">
                <span
                    style="display:inline-block;width:10px;height:10px;background:{getColor(d.key)};margin-right:5px;border-radius:10px;"
                ></span>
                {labels[d.key]} ({d.value.toFixed(1)}%)
            </span>
        {/if}
    {/each}
</div>