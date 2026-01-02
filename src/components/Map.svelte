<script>
    import { onMount } from "svelte";
    import mapboxgl from "mapbox-gl";
    import "mapbox-gl/dist/mapbox-gl.css";
    import booleanPointInPolygon from "@turf/boolean-point-in-polygon";
    import getCSSVar from "../utils/getCSSVar.js";
    import hexToRGB from "../utils/hexToRGB.js";

    export let sites;
    export let basin;

    let map;
    let isLoaded = false;

    mapboxgl.accessToken = import.meta.env.VITE_MAPBOX_TOKEN;

    // Convert sites to GeoJSON and determine if they're in basin
    function processSites(sitesData, basinData) {
        const features = sitesData.map((site) => {
            const point = {
                type: "Feature",
                geometry: {
                    type: "Point",
                    coordinates: [parseFloat(site.Lon), parseFloat(site.Lat)],
                },
                properties: {
                    ...site,
                    inBasin: false,
                },
            };

            // Check if point is in any basin polygon
            if (basinData && basinData.features) {
                for (const basinFeature of basinData.features) {
                    if (
                        basinFeature.geometry &&
                        booleanPointInPolygon(point, basinFeature.geometry)
                    ) {
                        point.properties.inBasin = true;
                        break;
                    }
                }
            }

            return point;
        });

        return {
            type: "FeatureCollection",
            features,
        };
    }

    onMount(() => {
        if (!basin || !sites) return;

        const sitesGeoJSON = processSites(sites, basin);

        // Calculate bounds from sites
        const coordinates = sitesGeoJSON.features.map(
            (f) => f.geometry.coordinates,
        );
        const lons = coordinates.map((c) => c[0]);
        const lats = coordinates.map((c) => c[1]);
        const bounds = [
            [Math.min(...lons), Math.min(...lats)],
            [Math.max(...lons), Math.max(...lats)],
        ];

        map = new mapboxgl.Map({
            container: "map",
            style: "mapbox://styles/agwaterdesk/clwqjql88011601qe1sd89qte",
            interactive: true,
        });

        map.fitBounds(bounds, {
            duration: 0,
            padding: 50,
        });

        // Set max bounds and max zoom to the initial view (prevent zooming out or panning beyond)
        map.setMaxBounds(map.getBounds());

        // Add zoom controls (includes zoom in/out buttons)
        map.addControl(new mapboxgl.NavigationControl(), "top-right");

        map.on("load", () => {
            isLoaded = true;

            // Add basin overlay first (will be behind sites)
            if (basin) {
                map.addSource("basin", {
                    type: "geojson",
                    data: basin,
                });

                map.addLayer(
                    {
                        id: "basin-fill",
                        type: "fill",
                        source: "basin",
                        paint: {
                            "fill-color": "#B0B3B3",
                            "fill-opacity": 0.3,
                        },
                    },
                    "settlement-subdivision-label",
                );

                map.addLayer(
                    {
                        id: "basin-outline",
                        type: "line",
                        source: "basin",
                        paint: {
                            "line-color": "#B0B3B3",
                            "line-width": 1,
                        },
                    },
                    "settlement-subdivision-label",
                );
            }

            // Add the sites GeoJSON data as a new source
            map.addSource("sites", {
                type: "geojson",
                data: sitesGeoJSON,
            });

            // Get marker colors from CSS variables
            const markerColor = getCSSVar("marker-color");
            const markerColorSubdued = hexToRGB(markerColor, 0.4);
            const markerFill = getCSSVar("marker-fill");
            const markerFillSubdued = hexToRGB(markerFill, 0.4);

            // Add a layer to visualize the features as circle markers (on top of basin)
            map.addLayer(
                {
                    id: "sites-markers",
                    type: "circle",
                    source: "sites",
                    paint: {
                        "circle-color": [
                            "case",
                            ["get", "inBasin"],
                            markerFill,
                            markerFillSubdued,
                        ],
                        "circle-radius": 5,
                        "circle-stroke-width": 2,
                        "circle-stroke-color": [
                            "case",
                            ["get", "inBasin"],
                            markerColor,
                            markerColorSubdued,
                        ],
                    },
                },
                "settlement-subdivision-label",
            );

            // Create a popup but don't add it to the map yet
            const popup = new mapboxgl.Popup({
                closeButton: true,
                closeOnClick: false,
                offset: 15,
            });

            // Show popup on mouseenter
            map.on("mouseenter", "sites-markers", (e) => {
                map.getCanvas().style.cursor = "pointer";

                const feature = e.features[0];
                const props = feature.properties;

                // Format the popup content
                const formatField = (label, value) => {
                    if (!value || value === "" || value === "#N/A") return "";
                    return `<div class="popup-field"><strong>${label}:</strong> ${value}</div>`;
                };

                const html = `
                    <div class="popup-title">${props.Facility_Name || ""}</div>
                    ${formatField("Owner", props.Owner)}
                    ${formatField("Location", `${props.City}, ${props.County}, ${props.State} ${props.Zipcode}`)}
          
                `;

                popup.setLngLat(e.lngLat).setHTML(html).addTo(map);
            });

            // Hide popup on mouseleave
            map.on("mouseleave", "sites-markers", () => {
                map.getCanvas().style.cursor = "";
                popup.remove();
            });
        });
    });
</script>

<div id="map"></div>

<style lang="scss">
    #map {
        width: 100%;
        height: 100%;
    }

    // Style Mapbox popup
    /* svelte-ignore a11y-no-unused-selector */
    :global {
        .mapboxgl-popup-content {
            font-family: var(
                --font-sans,
                -apple-system,
                BlinkMacSystemFont,
                "Segoe UI",
                Roboto,
                "Helvetica Neue",
                Arial,
                sans-serif
            );

            .popup-title {
                font-weight: 700;
                font-size: 14px;
                margin-bottom: 8px;
                color: #333;
            }

            .popup-field {
                font-size: 14px;
                line-height: 1.5;
                margin-bottom: 4px;
                color: #666;

                strong {
                    color: #333;
                    font-weight: 600;
                }
            }

            .popup-field:last-child {
                margin-bottom: 0;
            }
        }
    }
</style>
