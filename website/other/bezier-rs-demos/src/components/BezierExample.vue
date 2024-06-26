<template>
	<div>
		<h4 class="example-header">{{ title }}</h4>
		<figure @mousedown="onMouseDown" @mouseup="onMouseUp" @mousemove="onMouseMove" class="example-figure" v-html="bezierSVG"></figure>
		<div v-for="(slider, index) in sliderOptions" :key="index">
			<div class="slider-label">{{ slider.variable }} = {{ sliderData[slider.variable] }}{{ getSliderValue(sliderData[slider.variable], sliderUnits[slider.variable]) }}</div>
			<input class="slider" v-model.number="sliderData[slider.variable]" type="range" :step="slider.step" :min="slider.min" :max="slider.max" />
		</div>
	</div>
</template>

<style></style>

<script lang="ts">
import { defineComponent, PropType } from "vue";

import { WasmBezier } from "@/../wasm/pkg";
import { getConstructorKey, getCurveType, BezierCallback, BezierCurveType, WasmBezierManipulatorKey, SliderOption, ComputeType } from "@/utils/types";

const SELECTABLE_RANGE = 10;

// Given the number of points in the curve, map the index of a point to the correct manipulator key
const MANIPULATOR_KEYS_FROM_BEZIER_TYPE: { [key in BezierCurveType]: WasmBezierManipulatorKey[] } = {
	Linear: ["set_start", "set_end"],
	Quadratic: ["set_start", "set_handle_start", "set_end"],
	Cubic: ["set_start", "set_handle_start", "set_handle_end", "set_end"],
};

export default defineComponent({
	props: {
		title: { type: String as PropType<string>, required: true },
		points: { type: Array as PropType<Array<Array<number>>>, required: true, mutable: true },
		callback: { type: Function as PropType<BezierCallback>, required: true },
		sliderOptions: { type: Object as PropType<Array<SliderOption>>, default: () => ({}) },
		triggerOnMouseMove: { type: Boolean as PropType<boolean>, default: false },
		computeType: { type: String as PropType<ComputeType>, default: "Parametric" },
	},
	data() {
		const curveType = getCurveType(this.points.length);
		const manipulatorKeys = MANIPULATOR_KEYS_FROM_BEZIER_TYPE[curveType];

		const bezier = WasmBezier[getConstructorKey(curveType)](this.points);
		const sliderData = Object.assign({}, ...this.sliderOptions.map((s) => ({ [s.variable]: s.default })));

		return {
			bezier,
			bezierSVG: this.callback(bezier, sliderData, undefined, "Euclidean"),
			manipulatorKeys,
			activeIndex: undefined as number | undefined,
			mutablePoints: JSON.parse(JSON.stringify(this.points)),
			sliderData,
			sliderUnits: Object.assign({}, ...this.sliderOptions.map((s) => ({ [s.variable]: s.unit }))),
		};
	},
	methods: {
		onMouseDown(event: MouseEvent) {
			const mx = event.offsetX;
			const my = event.offsetY;
			for (let pointIndex = 0; pointIndex < this.points.length; pointIndex += 1) {
				const point = this.mutablePoints[pointIndex];
				if (point && Math.abs(mx - point[0]) < SELECTABLE_RANGE && Math.abs(my - point[1]) < SELECTABLE_RANGE) {
					this.activeIndex = pointIndex;
					return;
				}
			}
		},
		onMouseUp() {
			this.activeIndex = undefined;
		},
		onMouseMove(event: MouseEvent) {
			const mx = event.offsetX;
			const my = event.offsetY;
			if (this.activeIndex !== undefined) {
				this.bezier[this.manipulatorKeys[this.activeIndex]](mx, my);
				this.mutablePoints[this.activeIndex] = [mx, my];
				this.bezierSVG = this.callback(this.bezier, this.sliderData, undefined, this.computeType);
			} else if (this.triggerOnMouseMove) {
				this.bezierSVG = this.callback(this.bezier, this.sliderData, [mx, my], this.computeType);
			}
		},
		getSliderValue: (sliderValue: number, sliderUnit?: string | string[]) => (Array.isArray(sliderUnit) ? sliderUnit[sliderValue] : sliderUnit),
	},
	watch: {
		sliderData: {
			handler() {
				this.bezierSVG = this.callback(this.bezier, this.sliderData, undefined, this.computeType);
			},
			deep: true,
		},
		computeType: {
			handler() {
				this.bezierSVG = this.callback(this.bezier, this.sliderData, undefined, this.computeType);
			},
		},
	},
});
</script>
