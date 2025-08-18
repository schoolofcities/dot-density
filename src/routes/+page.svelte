<script>
    // TO DO
    // use tiles for basemap, format labels and zooming on top of dots
    // add N/A category for income, and commuting (subtract working pop from total pop)... data is confusing
    // optimize mobile

    // Modules
    import { onMount } from "svelte";
    import { tweened } from "svelte/motion";
    import { cubicOut } from "svelte/easing";
    import { base } from "$app/paths";
    import maplibregl from "maplibre-gl";
    import Papa from "papaparse";
    import { MapboxOverlay } from "@deck.gl/mapbox";
    import { PointCloudLayer } from "@deck.gl/layers";
    import { colorSchemes } from "$lib/colorSchemes.js";
    import "maplibre-gl/dist/maplibre-gl.css";
    import "../assets/styles.css";

    // Components
    import Loading from "$lib/Loading.svelte";
    import PointSizeSlider from "$lib/PointSizeSlider.svelte";

    // Legends
    import BarPeople from "$lib/BarPeople.svelte";
    import BarEthnicity from "$lib/BarEthnicity.svelte";
    import BarGender from "$lib/BarGender.svelte";
    import BarAge from "$lib/BarAge.svelte";
    import BarTransportation from "$lib/BarTransportation.svelte";
    import BarCommuteTime from "$lib/BarCommuteTime.svelte";
    import BarIncome from "$lib/BarIncome.svelte";
    import BarImmigration from "$lib/BarImmigration.svelte";

    // Logo
    import logoFull from "../assets/top-logo-full.svg";
    import lightDark from "../assets/light-dark.svg";

    // Initialize variables
    let map;
    let currentField = "ethnicity";
    let deckOverlay;
    let pointData = [];
    let invertBasemap = true;
    let showCanvas = true;
    const heightMultiplierTween = tweened(0, {
        duration: 600,
        easing: cubicOut,
    });
    $: heightMultiplier = $heightMultiplierTween;
    let pointSize = 0.5;
    let zoomFactor = 1;
    let basemapInverted = false;
    let loading = true;

    const CSV_URL = `${base}/pp-to-10m-height.csv`;
    const gl = WebGLRenderingContext;

    // Function to create the point cloud layer
    function createPointCloudLayer() {
        return new PointCloudLayer({
            id: `point-cloud-layer-${currentField}-${heightMultiplier}`,
            data: pointData,
            getPosition: (d) => [
                d.coordinates[0],
                d.coordinates[1],
                d.height * 2000 * heightMultiplier,
            ],
            getColor: (d) => {
                if (currentField === "people") {
                    return invertBasemap
                        ? [...colorSchemes.people.light]
                        : [...colorSchemes.people.dark];
                }
                const scheme = colorSchemes[currentField];
                const value = d[currentField];
                return scheme[value] || scheme.default;
            },
            pointSize: pointSize * zoomFactor,
            opacity: 1,
            material: {
                ambient: 0.8,
            },
            updateTriggers: {
                getPosition: [heightMultiplier],
                getColor: [currentField, invertBasemap],
            },
        });
    }

    // Reactive statements
    $: if (currentField && deckOverlay) {
        updateLayer();
    }
    $: if (deckOverlay && pointData.length > 0) {
        updateLayer();
    }
    $: if (heightMultiplier !== undefined && deckOverlay) {
        updateLayer();
    }
    $: if (pointSize !== undefined && deckOverlay) {
        updateLayer();
    }
    $: if (zoomFactor !== undefined && deckOverlay) {
        updateLayer();
    }
    $: if (invertBasemap !== undefined && deckOverlay) {
        updateLayer();
    }

    // Function to update the layer when currentField changes
    function updateLayer() {
        if (deckOverlay && pointData.length > 0) {
            const newLayer = createPointCloudLayer();
            deckOverlay.setProps({
                layers: [newLayer],
            });
        }
    }

    // Zoom factor
    function calculateZoomFactor(zoom) {
        const baseZoom = 10.887;
        const zoomSensitivity = 0.38;
        return Math.pow(2, (zoom - baseZoom) * zoomSensitivity);
    }

    // Interpolate zoom level based on #map width
    function getInterpolatedZoom(width) {
        const minWidth = 300;
        const midWidth = 416;
        const maxWidth = 1090;
        const minZoom = 8.7;
        const midZoom = 9.081832055479255;
        const maxZoom = 10.496;
        const clampedWidth = Math.max(minWidth, Math.min(maxWidth, width));

        if (clampedWidth <= midWidth) {
            return (
                minZoom +
                ((clampedWidth - minWidth) / (midWidth - minWidth)) *
                    (midZoom - minZoom)
            );
        } else {
            return (
                midZoom +
                ((clampedWidth - midWidth) / (maxWidth - midWidth)) *
                    (maxZoom - midZoom)
            );
        }
    }

    // Function to handle pitch toggle
    function handlePitchToggle(isExtended) {
        if (map) {
            const targetPitch = isExtended ? 54 : 0;
            map.easeTo({
                pitch: targetPitch,
                duration: 800,
            });
        }
    }

    // Initialize the map and load data
    onMount(async () => {
        let initialZoom = 10.496;
        const mapDiv = document.getElementById("map");
        if (mapDiv) {
            initialZoom = getInterpolatedZoom(mapDiv.offsetWidth);
        }
        map = new maplibregl.Map({
            container: "map",
            style: "https://basemaps.cartocdn.com/gl/voyager-gl-style/style.json",
            center: [-79.3559225, 43.7138077],
            zoom: initialZoom,
            pitch: 0,
            maxPitch: 80,
            bearing: -17,
        });

        map.on("load", async () => {
            const mapDiv = document.getElementById("map");
            if (mapDiv) {
                console.log("Map div width:", mapDiv.offsetWidth, "px");
            }

            // Initialize the point cloud layer
            const pointCloudLayer = createPointCloudLayer();

            // Place custom buttons inside map div
            const btn = document.querySelector(".invert-btn");
            const threedBtn = document.querySelector(".threed-btn");
            const container = document.querySelector(
                ".maplibregl-canvas-container",
            );
            if (btn && container) {
                container.appendChild(btn);
            }
            if (threedBtn && container) {
                container.appendChild(threedBtn);
            }

            // Map controls
            map.addControl(new maplibregl.NavigationControl(), "top-right");
            map.addControl(
                new maplibregl.ScaleControl({
                    maxWidth: 100,
                    unit: "metric",
                }),
                "bottom-left",
            );
            map.addControl(new maplibregl.FullscreenControl(), "top-right");
            map.doubleClickZoom.disable();

            // Load CSV data
            const response = await fetch(CSV_URL);
            const csvText = await response.text();
            const { data: csvData } = Papa.parse(csvText, {
                header: true,
                skipEmptyLines: true,
            });

            // Process CSV data into pointData array
            pointData = csvData.map((row) => ({
                coordinates: [
                    parseFloat(row.longitude || row.lon || row.lng),
                    parseFloat(row.latitude || row.lat),
                ],
                height: parseFloat(row.height || 0),
                gender: row.gender || "unknown",
                ethnicity: row.ethnicity || "unknown",
                age: row.age || "unknown",
                income: row.income || "unknown",
                journey_type: row.journey_type || "unknown",
                commute_time: row.commute_time || "unknown",
                immigration: row.immigration || "unknown",
            }));

            // Add point cloud layer
            deckOverlay = new MapboxOverlay({
                layers: [pointCloudLayer],
            });
            map.addControl(deckOverlay);

            // Initialize zoom factor
            zoomFactor = calculateZoomFactor(map.getZoom());

            map.on("zoom", () => {
                const newZoomFactor = calculateZoomFactor(map.getZoom());
                if (Math.abs(newZoomFactor - zoomFactor) > 0.01) {
                    zoomFactor = newZoomFactor;
                }
            });

            // Log map position changes
            // map.on("moveend", () => {
            //     console.log("Map position changed:", {
            //         center: map.getCenter(),
            //         zoom: map.getZoom(),
            //         pitch: map.getPitch(),
            //         bearing: map.getBearing(),
            //     });
            // });

            loading = false;
        });
    });
</script>

<button
    class="invert-btn"
    class:active={!invertBasemap}
    on:click={() => (invertBasemap = !invertBasemap)}
    style="display: flex; align-items: center; justify-content: center;"
>
    <img
        src={lightDark}
        alt="Toggle basemap"
        style="width: 22px; height: 22px; transform: rotate({invertBasemap
            ? 180
            : 0}deg);"
    />
</button>

<button
    class="threed-btn"
    class:active={heightMultiplier === 1}
    on:click={() => {
        heightMultiplierTween.set(heightMultiplier === 1 ? 0 : 1);
        handlePitchToggle(heightMultiplier === 0);
        console.log(
            "heightMultiplier changed to:",
            heightMultiplier === 1 ? 0 : 1,
        );
    }}
    style="display: flex; align-items: center; justify-content: center; {heightMultiplier ===
    1
        ? 'background: #f3f3f3; border: 1px solid #ccc; box-shadow: inset 0 1px 3px #bbb;'
        : ''}"
>
    <strong>3D</strong>
</button>

<div id="dashboard">
    <div class="title-block">
        <div class="title-content">
            <svg width="45" height="45" viewBox="0 0 40 40">
                <circle
                    cx="20"
                    cy="20"
                    r="18"
                    fill="none"
                    stroke="black"
                    stroke-width="1.5"
                />
            </svg>
            <h2>Toronto Dot <br /> Density Map</h2>
            <p>People of the 2021 Census in Toronto, Ontario, Canada.</p>
        </div>
        <div class="logo-mobile">
            <a href="https://schoolofcities.utoronto.ca/" target="_blank">
                <img
                    src={logoFull}
                    alt="Logo"
                    style="height: 40px; z-index: 1000;"
                />
            </a>
        </div>
    </div>

    <div class="content">
        <div class="field-selector">
            <h4 style="display: none;">Thematic</h4>
            <div class="field-buttons">
                <button
                    class:active={currentField === "people"}
                    on:click={() => (currentField = "people")}
                >
                    People
                </button>
                <button
                    class:active={currentField === "density"}
                    on:click={() => (currentField = "density")}
                    style="display: none;"
                >
                    Population Density
                </button>
                <button
                    class:active={currentField === "gender"}
                    on:click={() => (currentField = "gender")}
                >
                    Gender
                </button>
                <button
                    class:active={currentField === "age"}
                    on:click={() => (currentField = "age")}
                >
                    Age
                </button>
                <button
                    class:active={currentField === "income"}
                    on:click={() => (currentField = "income")}
                >
                    Income
                </button>
                <button
                    class:active={currentField === "ethnicity"}
                    on:click={() => (currentField = "ethnicity")}
                >
                    Ethnicity
                </button>
                <button
                    class:active={currentField === "immigration"}
                    on:click={() => (currentField = "immigration")}
                >
                    Immigration
                </button>

                <button
                    class:active={currentField === "journey_type"}
                    on:click={() => (currentField = "journey_type")}
                >
                    Commute Mode
                </button>
                <button
                    class:active={currentField === "commute_time"}
                    on:click={() => (currentField = "commute_time")}
                >
                    Commute Time
                </button>
            </div>

            <div>
                <p style="font-size: 0.8em; margin-bottom: 0;">
                    1 dot = 10 people
                </p>
            </div>

            <div class="legend-space">
                {#if currentField === "ethnicity"}
                    <BarEthnicity />
                {/if}
                {#if currentField === "gender"}
                    <BarGender />
                {/if}
                {#if currentField === "age"}
                    <BarAge />
                {/if}
                {#if currentField === "journey_type"}
                    <BarTransportation />
                {/if}
                {#if currentField === "commute_time"}
                    <BarCommuteTime />
                {/if}
                {#if currentField === "people"}
                    <BarPeople />
                {/if}
                {#if currentField === "income"}
                    <BarIncome />
                {/if}
                {#if currentField === "immigration"}
                    <BarImmigration />
                {/if}
            </div>

            <h4 style="display: none;">Basemap</h4>
            <div class="field-buttons">
                <button
                    class="basemap-btn invert-btn"
                    class:active={!invertBasemap}
                    on:click={() => (invertBasemap = !invertBasemap)}
                    style="display: none;"
                >
                    {invertBasemap ? "Light" : "Dark"}
                </button>
                <button
                    class="basemap-btn toggle-btn"
                    class:active={!showCanvas}
                    on:click={() => (showCanvas = !showCanvas)}
                    style="display: none;"
                >
                    {showCanvas ? "On" : "Off"}
                </button>
            </div>
        </div>
        <!-- 
        <div class="point-size-control">
            <PointSizeSlider bind:pointSize />
        </div> -->

        <div class="info-block">
            <h3>Info</h3>
            <p>
                This map shows the population of Toronto, Ontario, Canada as
                collected by Statistics Canada for the <a
                    href="https://www12.statcan.gc.ca/census-recensement/index-eng.cfm"
                    target="_blank">2021 Canadian Census</a
                >.
            </p>
            <p>
                Using dots to visualize people, this map displays geographic
                nuance that is focused on the continuous mesh of people, instead
                of administrative boundaries.
            </p>

            <h3>Methodology</h3>
            <p>
                The dots were created using population data by Toronto
                dissemination areas as collected from the 2021 Canadian Census.
            </p>

            <p>
                Focusing on Toronto, we explored different thematics by
                classifying each dot by a categorical variable, distributed by
                the relative percentages within their dissemination area.
            </p>

            <p>
                The result shows variations and diversity in colourful dotted
                patterns that span the entire city.
            </p>

            <h3>3D view</h3>

            <p>
                The 3D toggle is an experimental feature that visualizes
                population density as a point cloud, with peaks representing
                higher density areas and valleys representing lower density
                areas.
            </p>

            <p>
                The heights of the dots were calculated using nearest neighbor
                calculations that take into account the density of surrounding
                points.
            </p>

            <h3>References</h3>
            <p>
                This work takes inspiration from Jonathan Critchley's <a
                    href="https://jonathancritchley.ca/todot.html"
                    target="_blank">Toronto 2006 Dot Density</a
                > maps.
            </p>
            <p class="credit">
                Created by <a
                    href="https://www.linkedin.com/in/scott-christian-mccallum/"
                    target="_blank">Scott McCallum</a
                >
                and
                <a href="https://jamaps.github.io/" target="_blank"
                    >Jeff Allen</a
                >

                | August 2025
            </p>
        </div>
    </div>

    <div class="footer">
        <a href="https://schoolofcities.utoronto.ca/" target="_blank">
            <img
                src={logoFull}
                alt="Logo"
                style="height: 40px; z-index: 1000;"
            />
        </a>
    </div>
</div>

<div
    id="map"
    class:invert-basemap={!invertBasemap}
    class:hide-canvas={!showCanvas}
    style="position: relative;"
>
    {#if loading}
        <div class="map-loading-overlay">
            <Loading message="Loading data..." />
        </div>
    {/if}
</div>
