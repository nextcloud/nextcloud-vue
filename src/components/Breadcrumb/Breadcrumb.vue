<!--
 - @copyright Copyright (c) 2020 Raimund Schlüßler <raimund.schluessler@mailbox.org>
 -
 - @author Raimund Schlüßler <raimund.schluessler@mailbox.org>
 -
 - @license GNU AGPL version 3 or any later version
 -
 - This program is free software: you can redistribute it and/or modify
 - it under the terms of the GNU Affero General Public License as
 - published by the Free Software Foundation, either version 3 of the
 - License, or (at your option) any later version.
 -
 - This program is distributed in the hope that it will be useful,
 - but WITHOUT ANY WARRANTY; without even the implied warranty of
 - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
 - GNU Affero General Public License for more details.
 -
 - You should have received a copy of the GNU Affero General Public License
 - along with this program. If not, see <http://www.gnu.org/licenses/>.
 -
 -->

<docs>

### General description

This component is meant to be used inside a Breadcrumbs component.

</docs>

<template>
	<div ref="crumb"
		class="crumb"
		:class="{'crumb--with-action': $slots.default, 'crumb--hovered': hovering}"
		draggable="false"
		@dragstart.prevent="() => {/** Prevent the breadcrumb from being draggable. */}"
		@drop.prevent="dropped"
		@dragover.prevent="() => {}"
		@dragenter="dragEnter"
		@dragleave="dragLeave">
		<element
			:is="tag"
			v-if="title || icon"
			:to="to"
			:href="href">
			<!-- @slot Slot for passing a material design icon. Precedes the icon and title prop. -->
			<slot name="icon">
				<span v-if="icon" :class="icon" class="icon" />
				<span v-else>{{ title }}</span>
			</slot>
		</element>
		<Actions ref="actions"
			:force-menu="forceMenu"
			:open="open"
			@update:open="onOpenChange">
			<!-- @slot All action elements passed into the default slot will be used -->
			<slot />
		</Actions>
	</div>
</template>

<script>
import Actions from '../Actions'

export default {
	name: 'Breadcrumb',
	components: {
		Actions,
	},
	props: {
		/**
		 * The displayed title of the breadcrumb.
		 */
		title: {
			type: String,
			required: true,
		},
		/**
		 * The router-link to prop [https://router.vuejs.org/api/#to](https://router.vuejs.org/api/#to)
		 * If set, the breadcrumbs will be rendered by router-link.
		 */
		to: {
			type: String,
			default: undefined,
		},
		/**
		 * Set this prop if your app doesn't use vue-router, breadcrumbs will show as normal links.
		 */
		href: {
			type: String,
			default: undefined,
		},
		/**
		 * Set a css icon-class to show an icon instead of the title text.
		 */
		icon: {
			type: String,
			default: '',
		},
		/**
		 * Disable dropping on this breadcrumb.
		 */
		disableDrop: {
			type: Boolean,
			default: false,
		},
		/**
		 * Force the actions to display in a three dot menu
		 */
		forceMenu: {
			type: Boolean,
			default: false,
		},
		/**
		 * Open state of the Actions menu
		 */
		open: {
			type: Boolean,
			default: false,
		},
	},
	data() {
		return {
			/**
			 * Variable to track if we hover over the breadcrumb
			 */
			hovering: false,
		}
	},
	computed: {
		/**
		 * Determines which element tag to use
		 *
		 * @returns {String} the tag
		 */
		tag() {
			return this.to ? 'router-link' : 'a'
		},
	},
	methods: {
		/**
		 * Function to handle changing the open state of the Actions menu
		 * $emit the open state.
		 *
		 * @param {boolean} open The open state of the Actions menu
		 */
		onOpenChange(open) {
			/**
			 * Event emitted when the open state of the Actions menu changes
			 * @type {null}
			 */
			this.$emit('update:open', open)
		},
		/**
		 * Function to handle a drop on the breadcrumb.
		 * $emit the event and the path, remove the hovering state.
		 *
		 * @param {Object} e The drop event
		 * @returns {boolean}
		 */
		dropped(e) {
			/**
			 * Don't do anything if dropping is disabled.
			 */
			if (this.disableDrop) {
				return false
			}
			/**
			 * Event emitted when something is dropped on the breadcrumb.
			 * @type {null}
			 */
			this.$emit('dropped', e, this.to || this.href)
			this.$parent.$emit('dropped', e, this.to || this.href)
			this.hovering = false
			return false
		},
		/**
		 * Add the hovering state on drag enter
		 *
		 * @param {Object} e The drag enter event
		 */
		dragEnter(e) {
			/**
			 * Don't do anything if dropping is disabled.
			 */
			if (this.disableDrop) {
				return
			}
			this.hovering = true
		},
		/**
		 * Remove the hovering state on drag leave
		 *
		 * @param {Object} e The drag leave event
		 */
		dragLeave(e) {
			/**
			 * Don't do anything if dropping is disabled.
			 */
			if (this.disableDrop) {
				return
			}
			// Don't do anything if we
			// - leave towards a child element.
			// - or are still within the crumb
			if (e.target.contains(e.relatedTarget)
				|| this.$refs.crumb.contains(e.relatedTarget)) {
				return
			}
			this.hovering = false
		},
	},
}
</script>

<style lang="scss" scoped>

.crumb {
	background-image: none;
	display: inline-flex;
	height: $clickable-area;
	padding: 0;

	&:last-child {
		max-width: 210px;

		a {
			flex-shrink: 1;
		}
	}

	&::after {
		content: '';
		display: flex;
		align-items: center;
		color: var(--color-border-dark);
		font-size: 26px;
		width: 8px;
		min-width: 8px;
		background-image: url('./breadcrumb.svg');
		background-size: contain;
		background-repeat: no-repeat;
		background-position: center;
		opacity: .3;
		body.theme--dark & {
			background-image: url('./breadcrumb-light.svg');
		}
	}

	&--hidden {
		display: none;
	}

	&--with-action a {
		padding-right: 2px;
	}

	> a, > span {
		max-width: 100%;
	}

	a {
		align-items: center;
		display: inline-flex;

		> span {
			overflow: hidden;
			text-overflow: ellipsis;
			white-space: nowrap;
		}
	}
}
</style>
