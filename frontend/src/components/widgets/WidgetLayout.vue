<!-- TODO: Refactor this component (together with `WidgetRow.vue`) to be more logically consistent with our layout definition goals, in terms of naming and capabilities -->

<template>
	<div class="widget-layout">
		<component :is="layoutGroupType(layoutRow)" :widgetData="layoutRow" :layoutTarget="layout.layoutTarget" v-for="(layoutRow, index) in layout.layout" :key="index" />
	</div>
</template>

<style lang="scss">
.widget-layout {
	height: 100%;
	flex: 0 0 auto;
	display: flex;
	flex-direction: column;
}
</style>

<script lang="ts">
import { defineComponent, type PropType } from "vue";

import { isWidgetColumn, isWidgetRow, isWidgetSection, type LayoutGroup, type WidgetLayout } from "@/wasm-communication/messages";

import WidgetSection from "@/components/widgets/groups/WidgetSection.vue";
import WidgetRow from "@/components/widgets/WidgetRow.vue";

export default defineComponent({
	props: {
		layout: { type: Object as PropType<WidgetLayout>, required: true },
	},
	methods: {
		layoutGroupType(layoutRow: LayoutGroup): unknown {
			if (isWidgetColumn(layoutRow)) return WidgetRow;
			if (isWidgetRow(layoutRow)) return WidgetRow;
			if (isWidgetSection(layoutRow)) return WidgetSection;

			throw new Error("Layout row type does not exist");
		},
	},
	components: {
		WidgetRow,
		WidgetSection,
	},
});
</script>
