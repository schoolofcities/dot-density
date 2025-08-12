<script>
    // Exported value for point size
    export let pointSize = 1;

    const minSize = 0.5;
    const maxSize = 0.8;

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

    .slider-label {
        display: block;
        margin-bottom: 0.5rem;
        font-weight: bold;
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
        height: 15px;
        width: 15px;
        border-radius: 50%;
        background: white;
        border: 1px solid black;
        cursor: pointer;
        margin-top: -14px;
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
