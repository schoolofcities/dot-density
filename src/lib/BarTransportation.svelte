<script>
    import { colorSchemes } from "./colorSchemes.js";

    const data = [
        { key: "c", value: 61.930385673740474 },
        { key: "t", value: 25.947976545504314 },
        { key: "w", value: 7.918413507945426 },
        { key: "o", value: 2.430090331503652 },
        { key: "b", value: 1.7043408105343532 },
    ];

    const labels = {
        c: "Car",
        t: "Transit",
        w: "Walk",
        o: "Other",
        b: "Bicycle",
    };

    function getColor(key) {
        const c = colorSchemes.journey_type[key] || colorSchemes.journey_type.default;
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