<template>
	<div :style="`--color-hue: ${hue.value}; --color-hue-percentage: ${hue.percentage}; --color-saturation: ${saturation.percentage}; --color-lightness: ${lightness.percentage}; `">
		<div class="color-chooser" ref="chooser">
			<div class="color-chooser__panel color-chooser__panel--hue" @mousemove="chooseHue">
				<div class="color-chooser__marker" :class="{'frozen' : !hue.free}" @click="freeze(hue)"></div>
			</div>
			<div class="color-chooser__panel color-chooser__panel--saturation" @mousemove="chooseSaturation">
				<div class="color-chooser__marker" :class="{'frozen' : !saturation.free}" @click="freeze(saturation)"></div>
			</div>
			<div class="color-chooser__panel color-chooser__panel--lightness" @mousemove="chooseLightness">
				<div class="color-chooser__marker" :class="{'frozen' : !lightness.free}" @click="freeze(lightness)"></div>
			</div>
		</div>
		<div class="color-chooser__stats">
			<div class="color-chooser__chosen" :style="`color: ${textColor}`">{{ hex }}</div>
			<input type="text" v-model="hue.value">
			<input type="text" v-model="saturation.percentage">
			<input type="text" v-model="lightness.percentage">
		</div>
	</div>

</template>
<script>
export default {
	props: {
		value: {
			type: String,
			default: null
		},
		valueType: {
			type: String,
			default: 'hex'
		}
	},
	data() {
		return {
			hue: {
				free: true,
				value: 180,
				percentage: '50%'
			},
			saturation: {
				free: true,
				value: 50,
				percentage: '50%'
			},
			lightness: {
				free: true,
				value: 50,
				percentage: '50%'
			}
		};
	},
	watch: {
		hex(val) {
			this.$emit('input', this.hex);
		}
	},
	computed: {
		hex: function() {
			let _this = this;
			return this.hslToHex(
				_this.hue.value,
				_this.saturation.value,
				_this.lightness.value
			);
		},
		textColor: function() {
			return this.lightness.value > 50 ? 'black' : 'white';
		}
	},
	methods: {
		chooseHue(e) {
			let hue = this.minMax(
				(Math.floor(360 * this.getPosition(e)) - 360) * -1,
				0,
				360
			);
			let hue_percentage = Math.floor(100 * this.getPosition(e));
			if (this.hue.free) {
				this.hue.percentage = `${hue_percentage}%`;
				this.hue.value = hue;
			}
		},
		chooseSaturation(e) {
			let saturation = this.minMax(
				Math.floor(100 * this.getPosition(e)),
				0,
				100
			);
			if (this.saturation.free) {
				this.saturation.percentage = `${saturation}%`;
				this.saturation.value = saturation;
			}
		},
		chooseLightness(e) {
			let lightness = this.minMax(
				Math.floor(100 * this.getPosition(e)),
				0,
				100
			);
			if (this.lightness.free) {
				this.lightness.percentage = `${lightness}%`;
				this.lightness.value = lightness;
			}
		},
		getPosition(ev) {
			let parent = this.$refs.chooser.getBoundingClientRect();
			return (ev.clientY - parent.top) / parent.height;
		},
		freeze(marker) {
			marker.free = false;
			setTimeout(() => {
				marker.free = true;
			}, 1000);
		},
		minMax(val, min, max) {
			val = val > max ? max : val;
			val = val < min ? min : val;
			return val;
		},
		hslToHex(h, s, l) {
			h /= 360;
			s /= 100;
			l /= 100;
			let r, g, b;
			if (s === 0) {
				r = g = b = l; // achromatic
			} else {
				const hue2rgb = (p, q, t) => {
					if (t < 0) t += 1;
					if (t > 1) t -= 1;
					if (t < 1 / 6) return p + (q - p) * 6 * t;
					if (t < 1 / 2) return q;
					if (t < 2 / 3) return p + (q - p) * (2 / 3 - t) * 6;
					return p;
				};
				const q = l < 0.5 ? l * (1 + s) : l + s - l * s;
				const p = 2 * l - q;
				r = hue2rgb(p, q, h + 1 / 3);
				g = hue2rgb(p, q, h);
				b = hue2rgb(p, q, h - 1 / 3);
			}
			const toHex = (x) => {
				const hex = Math.round(x * 255).toString(16);
				return hex.length === 1 ? '0' + hex : hex;
			};
			return `#${toHex(r)}${toHex(g)}${toHex(b)}`;
		}
	}
};
</script>
<style scoped>
.color-chooser {
	width: 100%;
	height: 300px;
	display: flex;
}

.color-chooser__panel {
	width: 33.33%;
	height: 100%;
	position: relative;
}
.color-chooser__panel + .color-chooser__panel {
	border-left: 1px solid white;
}
.color-chooser__panel--hue {
	background: linear-gradient(
		to top,
		hsl(0, 100%, 50%) 0%,
		hsl(30, 100%, 50%) 8.333%,
		hsl(60, 100%, 50%) 16.666%,
		hsl(90, 100%, 50%) 25%,
		hsl(120, 100%, 50%) 33.333%,
		hsl(150, 100%, 50%) 41.666%,
		hsl(180, 100%, 50%) 50%,
		hsl(210, 100%, 50%) 58.333%,
		hsl(240, 100%, 50%) 66.666%,
		hsl(270, 100%, 50%) 75%,
		hsl(300, 100%, 50%) 83.333%,
		hsl(330, 100%, 50%) 91.666%,
		hsl(360, 100%, 50%) 100%
	);
}
.color-chooser__panel--hue .color-chooser__marker {
	top: var(--color-hue-percentage);
}
.color-chooser__panel--saturation {
	background-color: hsl(var(--color-hue), 100%, 50%);
	background-image: linear-gradient(
		to top,
		hsl(var(--color-hue), 100%, 50%) 0%,
		hsl(var(--color-hue), 0%, 50%) 100%
	);
}
.color-chooser__panel--saturation .color-chooser__marker {
	top: var(--color-saturation);
}
.color-chooser__panel--lightness {
	background-color: hsl(var(--color-hue), 100%, 50%);
	background-image: linear-gradient(
		to top,
		hsl(var(--color-hue), var(--color-saturation), 100%) 0%,
		hsl(var(--color-hue), var(--color-saturation), 50%) 50%,
		hsl(var(--color-hue), var(--color-saturation), 0%) 100%
	);
}
.color-chooser__panel--lightness .color-chooser__marker {
	top: var(--color-lightness);
}

.color-chooser__chosen {
	width: 100%;
	border-radius: 8px;
	margin-top: 10px;
	background-color: hsla(
		var(--color-hue),
		var(--color-saturation),
		var(--color-lightness),
		1
	);
	display: block;
	padding: 0.5rem 1rem;
	border: 1px solid
		hsl(
			var(--color-hue),
			var(--color-saturation),
			calc(var(--color-lightness) + 20)
		);
}

.color-chooser__marker {
	display: block;
	position: absolute;
	left: -2px;
	width: calc(100% + 4px);
	height: 8px;
	background-color: hsla(
		var(--color-hue),
		var(--color-saturation),
		var(--color-lightness),
		1
	);
	box-shadow: 0 0 0.5rem 0 rgba(0, 0, 0, 0.25);
}
.color-chooser__marker.frozen {
	border: 1px solid white;
}
</style>