<template>
	<LayoutRow class="popover-button">
		<IconButton :class="{ open }" :disabled="disabled" :action="() => onClick()" :icon="icon" :size="16" data-floating-menu-spawner :tooltip="tooltip" />
		<FloatingMenu v-model:open="open" :type="'Popover'" :direction="'Bottom'">
			<slot></slot>
		</FloatingMenu>
	</LayoutRow>
</template>

<style lang="scss">
.popover-button {
	position: relative;
	width: 16px;
	height: 24px;
	flex: 0 0 auto;

	.floating-menu {
		left: 50%;
		bottom: 0;
	}

	.icon-button {
		width: 100%;
		height: 100%;
		padding: 0;
		border: none;
		border-radius: 2px;
		background: var(--color-1-nearblack);
		fill: var(--color-e-nearwhite);

		&:hover,
		&.open {
			background: var(--color-6-lowergray);
			fill: var(--color-f-white);
		}

		&.disabled {
			background: var(--color-2-mildblack);
			fill: var(--color-8-uppergray);
		}
	}

	// TODO: Refactor this and other complicated cases dealing with joined widget margins and border-radius by adding a single standard set of classes: joined-first, joined-inner, and joined-last
	div[class*="-input"] + & {
		margin-left: 1px;

		.icon-button {
			border-radius: 0 2px 2px 0;
		}
	}
}
</style>

<script lang="ts">
import { defineComponent, type PropType } from "vue";

import { type IconName } from "@/utility-functions/icons";

import FloatingMenu from "@/components/layout/FloatingMenu.vue";
import LayoutRow from "@/components/layout/LayoutRow.vue";
import IconButton from "@/components/widgets/buttons/IconButton.vue";

export default defineComponent({
	props: {
		icon: { type: String as PropType<IconName>, default: "DropdownArrow" },
		tooltip: { type: String as PropType<string | undefined>, required: false },
		disabled: { type: Boolean as PropType<boolean>, default: false },

		// Callbacks
		action: { type: Function as PropType<() => void>, required: false },
	},
	data() {
		return {
			open: false,
		};
	},
	methods: {
		onClick() {
			this.open = true;

			this.action?.();
		},
	},
	components: {
		FloatingMenu,
		IconButton,
		LayoutRow,
	},
});
</script>
