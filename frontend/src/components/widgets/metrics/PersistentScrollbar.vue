<template>
	<div class="persistent-scrollbar" :class="direction.toLowerCase()">
		<button class="arrow decrease" @pointerdown="() => changePosition(-50)" tabindex="-1"></button>
		<div class="scroll-track" ref="scrollTrack" @pointerdown="(e) => grabArea(e)">
			<div class="scroll-thumb" @pointerdown="(e) => grabHandle(e)" :class="{ dragging }" :style="[thumbStart, thumbEnd, sides]"></div>
		</div>
		<button class="arrow increase" @click="() => changePosition(50)" tabindex="-1"></button>
	</div>
</template>

<style lang="scss">
.persistent-scrollbar {
	display: flex;
	flex: 1 1 100%;

	.arrow {
		flex: 0 0 auto;
		background: none;
		border: none;
		border-style: solid;
		width: 0;
		height: 0;
		margin: 0;
		padding: 0;
	}

	.scroll-track {
		flex: 1 1 100%;
		position: relative;

		.scroll-thumb {
			position: absolute;
			border-radius: 4px;
			background: var(--color-5-dullgray);

			&:hover,
			&.dragging {
				background: var(--color-6-lowergray);
			}
		}

		.scroll-click-area {
			position: absolute;
		}
	}

	&.vertical {
		flex-direction: column;

		.arrow.decrease {
			margin: 4px 3px;
			border-width: 0 5px 8px 5px;
			border-color: transparent transparent var(--color-5-dullgray) transparent;

			&:hover {
				border-color: transparent transparent var(--color-6-lowergray) transparent;
			}
			&:active {
				border-color: transparent transparent var(--color-c-brightgray) transparent;
			}
		}

		.arrow.increase {
			margin: 4px 3px;
			border-width: 8px 5px 0 5px;
			border-color: var(--color-5-dullgray) transparent transparent transparent;

			&:hover {
				border-color: var(--color-6-lowergray) transparent transparent transparent;
			}
			&:active {
				border-color: var(--color-c-brightgray) transparent transparent transparent;
			}
		}
	}

	&.horizontal {
		flex-direction: row;

		.arrow.decrease {
			margin: 3px 4px;
			border-width: 5px 8px 5px 0;
			border-color: transparent var(--color-5-dullgray) transparent transparent;

			&:hover {
				border-color: transparent var(--color-6-lowergray) transparent transparent;
			}
			&:active {
				border-color: transparent var(--color-c-brightgray) transparent transparent;
			}
		}

		.arrow.increase {
			margin: 3px 4px;
			border-width: 5px 0 5px 8px;
			border-color: transparent transparent transparent var(--color-5-dullgray);

			&:hover {
				border-color: transparent transparent transparent var(--color-6-lowergray);
			}
			&:active {
				border-color: transparent transparent transparent var(--color-c-brightgray);
			}
		}
	}
}
</style>

<script lang="ts">
import { defineComponent, type PropType } from "vue";

export type ScrollbarDirection = "Horizontal" | "Vertical";

// Linear Interpolation
const lerp = (x: number, y: number, a: number): number => x * (1 - a) + y * a;

// Convert the position of the handle (0-1) to the position on the track (0-1).
// This includes the 1/2 handle length gap of the possible handle positionson each side so the end of the handle doesn't go off the track.
const handleToTrack = (handleLen: number, handlePos: number): number => lerp(handleLen / 2, 1 - handleLen / 2, handlePos);

const pointerPosition = (direction: ScrollbarDirection, e: PointerEvent): number => (direction === "Vertical" ? e.clientY : e.clientX);

export default defineComponent({
	emits: {
		"update:handlePosition": null,
		pressTrack: (pointerOffset: number) => typeof pointerOffset === "number",
	},
	props: {
		direction: { type: String as PropType<ScrollbarDirection>, default: "Vertical" },
		handlePosition: { type: Number as PropType<number>, default: 0.5 },
		handleLength: { type: Number as PropType<number>, default: 0.5 },
	},
	computed: {
		thumbStart(): { left: string } | { top: string } {
			const start = handleToTrack(this.handleLength, this.handlePosition) - this.handleLength / 2;

			return this.direction === "Vertical" ? { top: `${start * 100}%` } : { left: `${start * 100}%` };
		},
		thumbEnd(): { right: string } | { bottom: string } {
			const end = 1 - handleToTrack(this.handleLength, this.handlePosition) - this.handleLength / 2;

			return this.direction === "Vertical" ? { bottom: `${end * 100}%` } : { right: `${end * 100}%` };
		},
		sides(): { left: string; right: string } | { top: string; bottom: string } {
			return this.direction === "Vertical" ? { left: "0%", right: "0%" } : { top: "0%", bottom: "0%" };
		},
	},
	data() {
		return {
			dragging: false,
			pointerPos: 0,
		};
	},
	mounted() {
		window.addEventListener("pointerup", this.pointerUp);
		window.addEventListener("pointermove", this.pointerMove);
	},
	unmounted() {
		window.removeEventListener("pointerup", this.pointerUp);
		window.removeEventListener("pointermove", this.pointerMove);
	},
	methods: {
		trackLength(): number | undefined {
			const track = this.$refs.scrollTrack as HTMLDivElement | undefined;
			if (track) return this.direction === "Vertical" ? track.clientHeight - this.handleLength : track.clientWidth;

			return undefined;
		},
		trackOffset(): number | undefined {
			const track = this.$refs.scrollTrack as HTMLDivElement | undefined;
			if (track) return this.direction === "Vertical" ? track.getBoundingClientRect().top : track.getBoundingClientRect().left;

			return undefined;
		},
		clampHandlePosition(newPos: number) {
			const clampedPosition = Math.min(Math.max(newPos, 0), 1);
			this.$emit("update:handlePosition", clampedPosition);
		},
		updateHandlePosition(e: PointerEvent) {
			const trackLength = this.trackLength();
			if (trackLength === undefined) return;

			const position = pointerPosition(this.direction, e);

			this.clampHandlePosition(this.handlePosition + (position - this.pointerPos) / (trackLength * (1 - this.handleLength)));
			this.pointerPos = position;
		},
		grabHandle(e: PointerEvent) {
			if (!this.dragging) {
				this.dragging = true;
				this.pointerPos = pointerPosition(this.direction, e);
			}
		},
		grabArea(e: PointerEvent) {
			if (!this.dragging) {
				const trackLength = this.trackLength();
				const trackOffset = this.trackOffset();
				if (trackLength === undefined || trackOffset === undefined) return;

				const oldPointer = handleToTrack(this.handleLength, this.handlePosition) * trackLength + trackOffset;
				const pointerPos = pointerPosition(this.direction, e);
				this.$emit("pressTrack", pointerPos - oldPointer);
			}
		},
		pointerUp() {
			this.dragging = false;
		},
		pointerMove(e: PointerEvent) {
			if (this.dragging) this.updateHandlePosition(e);
		},
		changePosition(difference: number) {
			const trackLength = this.trackLength();
			if (trackLength === undefined) return;

			this.clampHandlePosition(this.handlePosition + difference / trackLength);
		},
	},
});
</script>
