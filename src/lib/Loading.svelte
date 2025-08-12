<script>
    export let message = "Loading...";
</script>

<div class="loading-overlay">
    <div class="loader-container">
        <div class="center-circle-area">
            <div class="center-circle"></div>
        </div>
        <div class="loading-message">{message}</div>
    </div>
</div>

<style>
    .loader-container {
        display: flex;
        flex-direction: column;
        align-items: center;
    }
    .center-circle-area {
        /* width: 64px; */
        /* height: 64px; */
        display: flex;
        align-items: center;
        justify-content: center;
        margin-bottom: 16px;
    }
    .center-circle {
        width: 32px;
        height: 32px;
        background: transparent;
        border: 1.5px solid #000;
        border-radius: 50%;
    }
    .loading-message {
        color: #000;
        font-size: 1.2em;
        letter-spacing: 0.05em;
        text-align: center;
    }
</style>
