<script>
	import { onMount } from 'svelte';
	import { browser } from '$app/env';
	import Map from 'ol/Map';
	import OSM from 'ol/source/OSM';
	import TileLayer from 'ol/layer/Tile';
	import { Icon, Style } from 'ol/style';
	import VectorSource from 'ol/source/Vector';
	import VectorLayer from 'ol/layer/Vector';
	import View from 'ol/View';
	import Feature from 'ol/Feature';
	import Point from 'ol/geom/Point';
	import { fromLonLat } from 'ol/proj';
	import { DragRotateAndZoom, defaults as defaultInteractions } from 'ol/interaction';

	function getRandomArbitrary(min, max) {
		return Math.random() * (max - min) + min;
	}

	function getRandomColor() {
		var letters = '0123456789ABCDEF';
		var color = '#';
		for (var i = 0; i < 6; i++) {
			color += letters[Math.floor(Math.random() * 16)];
		}
		return color;
	}

	function hexToRgbA(hex) {
		var c;
		if (/^#([A-Fa-f0-9]{3}){1,2}$/.test(hex)) {
			c = hex.substring(1).split('');
			if (c.length == 3) {
				c = [c[0], c[0], c[1], c[1], c[2], c[2]];
			}
			c = '0x' + c.join('');
			return [(c >> 16) & 255, (c >> 8) & 255, c & 255, 1];
		}
		throw new Error('Bad Hex');
	}

	onMount(async () => {
		if (browser) {
			var marker = new Feature({
				geometry: new Point(fromLonLat([-110, 45])),
				name: 'test'
			});

            let styles = []

            for (let i = 0; i < 100; i++) {
                let baseStyle = new Style({
				    image: new Icon({
					    scale: 0.015,
					    color: getRandomColor(),
					    crossOrigin: 'anonymous',
					    src: 'https://upload.wikimedia.org/wikipedia/commons/d/d2/White_plane_icon.svg'
				    })
			    });
                styles.push(baseStyle);
            }

			var iconStyleFn = function (feature, resolution) {
				return styles[feature.get('color')];
			};

			marker.setStyle(iconStyleFn);

			let markers = [];

			for (let i = 0; i < 10000; i++) {
				let cl = marker.clone();
				cl.setGeometry(
					new Point(fromLonLat([getRandomArbitrary(-180, 180), getRandomArbitrary(-90, 90)]))
				);
				cl.set('color', Math.round(getRandomArbitrary(0, 99)));
				markers.push(cl);
			}

			var planeVectorSource = new VectorSource({
				features: markers
			});

			var planeVectorLayer = new VectorLayer({
				source: planeVectorSource
			});

			var map = new Map({
				interactions: defaultInteractions().extend([new DragRotateAndZoom()]),
				layers: [
					new TileLayer({
						source: new OSM()
					}),
					planeVectorLayer
				],
				target: 'map',
				renderer: 'webgl',
				view: new View({
					center: fromLonLat([-110, 45]),
					zoom: 2
				})
			});
		}
	});
</script>

<h1>Welcome to SvelteKit</h1>
<div id="map" class="map" />
<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>

<style>
	@import 'ol/ol.css';
	.map {
		width: 100%;
		height: 400px;
	}
</style>
