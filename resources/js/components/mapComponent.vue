<template>
    <div class="flex-container">
        <div class="flex-geometry" v-if="edit">
            <input ref="file" :id="field.name" type="file" :class="errorClasses" :placeholder="field.name"
                @change="updateLinestring($event)" accept=".geojson,.gpx,.kml" />
            <p v-if="hasError" class="my-2 text-danger">
                {{ firstError }}
            </p>
        </div>
    </div>
    <div id="container">
        <div :id="mapRef" class="wm-map"></div>
    </div>
</template>

<script>
import { FormField, HandlesValidationErrors } from 'laravel-nova'
import "leaflet/dist/leaflet.css";
import L from "leaflet";
import "leaflet.fullscreen/Control.FullScreen.js";
import "leaflet.fullscreen/Control.FullScreen.css";
import "leaflet-draw/dist/leaflet.draw-src.js";
import "leaflet-draw/dist/leaflet.draw-src.css";

const DEFAULT_TILES = 'https://{s}.tile.openstreetthis.map.org/{z}/{x}/{y}.png';
const VERSION = "0.0.6"
const VERSION_IMAGE = `<img class="version-image" src="https://img.shields.io/badge/wm--map--multi--linestring-${VERSION}-blue">`;
const DEFAULT_ATTRIBUTION = '<a href="https://www.openstreetthis.map.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery (c) <a href="https://www.mapbox.com/">Mapbox</a>';
const DEFAULT_CENTER = [0, 0];
const DEFAULT_MINZOOM = 7;
const DEFAULT_MAXZOOM = 17;
const DEFAULT_DEFAULTZOOM = 8;
const LINESTRING_OPTIONS = {
    fillColor: '#f03',
    fillOpacity: 0.5,
};

export default {
    name: "MapMultiLineString",
    mixins: [FormField, HandlesValidationErrors],
    props: ["field", "edit"],
    data() {
        return {
            mapRef: `mapContainer-${Math.floor(Math.random() * 10000 + 10)}`,
            deleteIcon: null,
            map: null,
            linestring: null,
            geojson: null,
        }
    },
    methods: {
        initMap() {
            setTimeout(() => {
                this.center = this.field.center ?? DEFAULT_CENTER;
                this.maxZoom = this.field.maxZoom ?? DEFAULT_MAXZOOM;
                this.minZoom = this.field.minZoom ?? DEFAULT_MINZOOM;
                this.defaultZoom = this.field.defaultZoom ?? DEFAULT_DEFAULTZOOM;
                this.attribution = this.field.attribution ?? DEFAULT_ATTRIBUTION;
                this.initLeafletEditMode();
                this.buildMap();
                this.buildLinestring(this.geojson);
                this.buildLeafletEditMode();
                this.buildDeleteGeometry();
            }, 300);
        },
        /**
         * Initialize Leaflet edit mode by assigning Leaflet object (L)
         * to the "L" properties of the "document" and "window" objects.
         */
        initLeafletEditMode() {
            document.L = L;
            window.L = L;
        },
        buildMap() {
            var currentGeojson = this.field.geojson != null ? JSON.parse(this.field.geojson) : null;
            this.updateGeojson(currentGeojson)
            this.map = L.map(this.mapRef, {
                fullscreenControl: true,
                fullscreenControlOptions: {
                    position: "topleft"
                }
            }).setView(this.center, this.defaultZoom);
            L.tileLayer(this.field.tiles ?? DEFAULT_TILES, {
                attribution: `${this.attribution}, ${VERSION_IMAGE}`,
                maxZoom: this.maxZoom,
                minZoom: this.minZoom,
                id: "mapbox/streets-v11"
            }).addTo(this.map);
        },
        buildLinestring(geojson) {
            if (geojson != null) {
                this.linestring = L.geoJson(geojson, {
                    style: LINESTRING_OPTIONS
                }).addTo(this.map);
                this.map.fitBounds(this.linestring.getBounds());
            }
            try {
                if (this.edit) {
                    if (geojson != null) {
                        this.setEditMode();
                    } else {
                        this.setDrawMode();
                    }
                }
            } catch (_) { }
        },
        buildDeleteGeometry() {
            if (!this.edit) {
                return;
            }
            L.Control.deleteGeometry = L.Control.extend({
                onAdd: () => {
                    this.deleteIcon = L.DomUtil.create('div')
                    L.DomUtil.addClass(this.deleteIcon, 'delete-button');
                    var img = L.DomUtil.create('img');
                    img.src = 'https://cdn-icons-png.flaticon.com/512/2891/2891491.png';
                    this.deleteIcon.appendChild(img);
                    L.DomEvent.on(this.deleteIcon, "click", (e) => {
                        L.DomEvent.stopPropagation(e);
                        this.updateLinestring(null);
                        this.setDrawMode();
                        this.deleteIcon.style.visibility = "hidden";
                    });
                    if (this.edit && this.geojson != null) {

                        this.setEditMode();
                    } else {
                        this.deleteIcon.style.visibility = "hidden";
                        this.setDrawMode();
                    }
                    return this.deleteIcon;
                }
            });
            L.control.deleteGeometry = function (opts) {
                return new L.Control.deleteGeometry(opts);
            }
            L.control.deleteGeometry({ position: 'topright' }).addTo(this.map);
            if (this.linestring != null && this.edit) {
                this.deleteIcon.style.visibility = "visible";
            }
        },
        updateLinestring(event) {
            if (this.linestring !== null) {
                this.map.removeLayer(this.linestring);
                this.linestring = null;
            }
            if (event) {
                const reader = new FileReader();
                let fileName = event.target.files[0].name || '';
                reader.onload = (event) => {
                    let res = event.target.result;
                    if (fileName.indexOf('gpx') > -1) {
                        const parser = new DOMParser().parseFromString(res, 'text/xml');
                        res = t.gpx(parser);
                    } else if (fileName.indexOf('kml') > -1) {
                        const parser = new DOMParser().parseFromString(res, 'text/xml');
                        res = t.kml(parser);
                    } else {
                        res = JSON.parse(res);
                    }
                    this.updateGeojson(res)
                    try {
                        this.buildLinestring(this.geojson.features[0].geometry);
                    } catch (_) {
                        this.$refs.file.value = null;
                        this.deleteIcon.style.visibility = "hidden";
                        window.alert('The file is corrupted');
                    }
                };
                reader.readAsText(event.target.files[0]);
            } else {
                this.updateGeojson(null)
                this.$refs.file.value = null;
            }
        },
        updateGeojson(geojson) {
            this.geojson = geojson;
            this.$emit("geojson", geojson);
        },
        buildLeafletEditMode() {
            if (!this.edit) {
                return;
            }
            if (this.linestring == null) {
                this.setDrawMode();
            } else {
                this.setEditMode();
            }
            this.map.on('draw:created', (e) => {
                const layer = e.layer;
                if (this.linestring === null) {
                    this.linestring = L.featureGroup().addTo(this.map);
                    this.drawControl.setDrawingOptions({
                        edit: {
                            featureGroup: this.linestring,
                            remove: false
                        }
                    });
                }
                this.linestring.addLayer(layer);
                const geojson = this.linestring.toGeoJSON();
                this.updateGeojson(geojson);
            });
            this.map.on('draw:edited', (e) => {
                L.DomEvent.stopPropagation(e);
                var geojson = this.linestring.toGeoJSON();
                this.updateGeojson(geojson);
            });
            this.map.on('draw:deletestop', (e) => {
                L.DomEvent.stopPropagation(e);
            });
            this.map.on('draw:drawstop', (e) => {
                this.setEditMode();
                L.DomEvent.stopPropagation(e);
            })
        },
        setEditMode() {
            try {
                this.map.removeControl(this.drawControl);
            } catch (_) { }
            try {
                this.deleteIcon.style.visibility = "visible";
            } catch (_) { }
            this.drawControl = new L.Control.Draw({
                draw: false,
                edit: {
                    featureGroup: this.linestring,
                    remove: false
                }
            });
            this.map.addControl(this.drawControl);
        },
        setDrawMode() {
            try {
                this.map.removeControl(this.drawControl);
            } catch (_) { }
            try {
                this.deleteIcon.style.visibility = "hidden";
            } catch (_) { }
            this.drawControl = new L.Control.Draw({
                draw: {
                    polyline: {
                        shapeOptions: LINESTRING_OPTIONS
                    },
                    polygon: false,
                    rectangle: false,
                    circle: false,
                    marker: false,
                    circlemarker: false
                },
                edit: false
            });
            this.map.addControl(this.drawControl);
        }
    },
    mounted() {
        this.initMap();
    }
};
</script>
