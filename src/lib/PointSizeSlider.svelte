<script>
    export let pointSize = 0.65;

    const minSize = 0.3;
    const maxSize = 1;

    function handleInput(event) {
        pointSize = parseFloat(event.target.value);
    }
</script>

<div class="point-size-slider">
    <h4>Dot Size</h4>

    <div class="slider-container">
        <span class="size-label">Smaller</span>
        <input
            id="point-size-range"
            type="range"
            min={minSize}
            max={maxSize}
            step="0.01"
            bind:value={pointSize}
            on:input={handleInput}
            class="slider"
        />
        <span class="size-label">Larger</span>
    </div>
</div>

<style>
    .point-size-slider {
        margin: 1rem 0;
    }

    .slider-container {
        display: flex;
        align-items: center;
        gap: 1rem;
    }

    .size-label {
        font-size: 0.9rem;
        color: #555;
    }

    /* Base track style */
    .slider {
        -webkit-appearance: none;
        width: 100%;
        height: 1px; /* thinner line */
        background: black;
        border-radius: 1px;
        cursor: pointer;
        position: relative;
    }

    /* WebKit (Chrome, Safari) thumb */
    .slider::-webkit-slider-thumb {
        -webkit-appearance: none;
        height: 12px;
        width: 12px;
        border-radius: 50%;
        background: white;
        border: 1px solid black;
        cursor: pointer;
        margin-top: -12px;
    }

    /* Firefox thumb */
    .slider::-moz-range-thumb {
        height: 10px;
        width: 10px;
        border-radius: 50%;
        background: white; 
        border: 1px solid black;
        cursor: pointer;
        transform: translateY(-6px);
    }

    /* Firefox track */
    .slider::-moz-range-track {
        height: 1px; /* thinner line */
        background: black;
        border-radius: 1px;
    }

    .slider:focus {
        outline: none;
    }
</style>
