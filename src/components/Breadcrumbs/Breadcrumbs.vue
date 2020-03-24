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

This component renders a breadcrumb bar. It adjusts to the available width
by hiding breadcrumbs in a dropdown menu and emits an event when something
is dropped on a creadcrumb.

### Usage

```vue
<template>
	<div>
		<div class="container">
			<Breadcrumbs @dropped="dropped">
				<Breadcrumb title="Home" href="/" @dropped="droppedOnCrumb" />
				<Breadcrumb title="Folder 1" href="/Folder 1" />
				<Breadcrumb title="Folder 2" href="/Folder 1/Folder 2" :disable-drop="true" />
				<Breadcrumb title="Folder 3" href="/Folder 1/Folder 2/Folder 3" />
				<Breadcrumb title="Folder 4" href="/Folder 1/Folder 2/Folder 3/Folder 4" />
				<Breadcrumb title="Folder 5" href="/Folder 1/Folder 2/Folder 3/Folder 4/Folder 5" :disable-drop="true">
					<ActionButton icon="icon-share" @click="alert('Share')">
						Share
					</ActionButton>
				</Breadcrumb>
			</Breadcrumbs>
		</div>
		<br />
		<div class="dragme" draggable="true">
			<span>Drag me onto the breadcrumbs.</span>
		</div>
	</div>
</template>

<script>
export default {
	methods: {
		dropped(e, path) {
			alert('Global drop on ' + path)
		},
		droppedOnCrumb(e, path) {
			alert('Drop on crumb ' + path)
		},
	}
}
</script>
<style>
.container {
	display: inline-flex;
	width: 100%;
}

.dragme {
	display: block;
	width: 100px;
	height: 44px;
	background-color: var(--color-background-dark);
}
</style>
```
</docs>

<script>
import Vue from 'vue'
import debounce from 'debounce'
import Actions from '../Actions'
import ActionRouter from '../ActionRouter'
import ActionLink from '../ActionLink'
import ValidateSlot from '../../utils/ValidateSlot'

export default {
	name: 'Breadcrumbs',
	components: {
		Actions,
		ActionRouter,
		ActionLink,
	},
	props: {
		/**
		 * Set a css icon-class for the icon of the root breadcrumb to be used.
		 */
		rootIcon: {
			type: String,
			default: 'icon-home',
		},
	},
	data() {
		return {
			/**
			 * The breadcrumbs which should be shown in a dropdown Actions menu.
			 */
			hiddenCrumbs: [],
			/**
			 * Array to track the hidden breadcrumbs by their index.
			 * Comparing two crumbs somehow does not work, so we use the indices.
			 */
			hiddenIndices: [],
		}
	},
	beforeMount() {
		// Filter all invalid items, only Breadcrumb components are allowed
		ValidateSlot(this.$slots.default, ['Breadcrumb'], this)
	},
	beforeUpdate() {
		// Also check before every update
		ValidateSlot(this.$slots.default, ['Breadcrumb'], this)
	},
	created() {
		/**
		 * Add a listener so the component reacts on resize
		 */
		window.addEventListener('resize', debounce(() => {
			this.handleWindowResize()
		}, 100))
	},
	mounted() {
		this.handleWindowResize()
	},
	updated() {
		/**
		 * Check the size on update
		 */
		this.$nextTick(() => {
			this.handleWindowResize()
		})
	},
	beforeDestroy() {
		window.removeEventListener('resize', this.handleWindowResize)
	},
	methods: {
		/**
		 * Close the actions menu
		 *
		 * @param {Object} e The event
		 */
		closeActions(e) {
			// Don't do anything if we leave towards a child element.
			if (this.$refs.actions.$el.contains(e.relatedTarget)) {
				return
			}
			this.$refs.actions.opened = false
		},
		/**
		 * Check the width of the breadcrumb and hide breadcrumbs
		 * if we overflow otherwise.
		 */
		handleWindowResize() {
			// All breadcrumb components passed into the default slot
			const breadcrumbs = this.$slots.default || []
			// If there is no container yet, we cannot determine its size
			if (this.$refs.container) {
				const nrCrumbs = breadcrumbs.length
				const hiddenIndices = []
				const availableWidth = this.$refs.container.offsetWidth
				const totalWidth = this.getTotalWidth(breadcrumbs)
				let overflow = totalWidth - availableWidth
				// If we overflow, we have to take the action-item width into account as well.
				overflow += (overflow > 0) ? 51 : 0
				let i = 0
				// We start hiding the breadcrumb in the center
				const startIndex = Math.floor(nrCrumbs / 2)
				// Don't hide the first and last breadcrumb
				while (overflow > 0 && i < nrCrumbs - 2) {
					// We hide elements alternating to the left and right
					const currentIndex = startIndex + ((i % 2) ? i + 1 : i) / 2 * Math.pow(-1, i + (nrCrumbs % 2))
					// Calculate the remaining overflow width after hiding this breadcrumb
					overflow -= this.getWidth(breadcrumbs[currentIndex].elm)
					hiddenIndices.push(currentIndex)
					i++
				}
				// We only update the hidden crumbs if they have changed,
				// otherwise we will run into an infinite update loop.
				if (!this.arraysEqual(this.hiddenIndices, hiddenIndices.sort())) {
					// Get all breadcrumbs based on the hidden indices
					this.hiddenCrumbs = hiddenIndices.map((index) => { return breadcrumbs[index] })
					this.hiddenIndices = hiddenIndices
				}
			}
		},
		/**
		 * Checks if two arrays are equal.
		 * Only works for primitive arrays, but that's enough here.
		 *
		 * @param {Array} a The first array
		 * @param {Array} b The second array
		 * @returns {boolean} Wether the arrays are equal
		 */
		arraysEqual(a, b) {
			if (a.length !== b.length) return false
			if (a === b) return true
			if (a === null || b === null) return false

			for (let i = 0; i < a.length; ++i) {
				if (a[i] !== b[i]) {
					return false
				}
			}
			return true
		},
		/**
		 * Calculates the total width of all breadcrumbs
		 *
		 * @param {Array} breadcrumbs All breadcrumbs
		 * @returns {Integer} The total width
		 */
		getTotalWidth(breadcrumbs) {
			return breadcrumbs.reduce((width, crumb, index) => width + this.getWidth(crumb.elm), 0)
		},
		/**
		 * Calculates the width of the provided element
		 *
		 * @param {Object} el The element
		 * @returns {Integer} The width
		 */
		getWidth(el) {
			if (!el.classList) return 0
			const hide = el.classList.contains('crumb--hidden')
			el.classList.remove('crumb--hidden')
			const w = el.offsetWidth
			if (hide) {
				el.classList.add('crumb--hidden')
			}
			return w
		},
		/**
		 * Prevents the default of a provided event
		 *
		 * @param {Object} e The event
		 * @returns {boolean}
		 */
		preventDefault(e) {
			if (e.preventDefault) {
				e.preventDefault()
			}
			return false
		},
		/**
		 * Handles the drag start.
		 * Prevents a breadcrumb from being draggable.
		 *
		 * @param {Object} e The event
		 * @returns {boolean}
		 */
		dragStart(e) {
			return this.preventDefault(e)
		},
		/**
		 * Handles when something is dropped on the breadcrumb.
		 *
		 * @param {Object} e The drop event
		 * @param {String} path The path of the breadcrumb
		 * @param {boolean} disabled Whether dropping is disabled for this breadcrumb
		 * @returns {boolean}
		 */
		dropped(e, path, disabled) {
			/**
			 * Don't emit if dropping is disabled.
			 */
			if (!disabled) {
				/**
				 * Event emitted when something is dropped on the breadcrumb.
				 * @type {null}
				 */
				this.$emit('dropped', e, path)
			}
			// Close the actions menu after the drop
			this.$refs.actions.opened = false
			// Remove all hovering classes
			const crumbs = document.querySelectorAll('.crumb')
			crumbs.forEach((f) => { f.classList.remove('crumb--hovered') })
			return this.preventDefault(e)
		},
		/**
		 * Handles the drag over event
		 *
		 * @param {Object} e The drag over event
		 * @returns {boolean}
		 */
		dragOver(e) {
			return this.preventDefault(e)
		},
		/**
		 * Handles the drag enter event
		 *
		 * @param {Object} e The drag over event
		 * @param {boolean} disabled Whether dropping is disabled for this breadcrumb
		 */
		dragEnter(e, disabled) {
			/**
			 * Don't do anything if dropping is disabled.
			 */
			if (disabled) {
				return
			}
			// Get the correct element, in case we hover a child.
			if (e.target.closest) {
				const target = e.target.closest('.crumb')
				if (target.classList && target.classList.contains('crumb')) {
					const crumbs = document.querySelectorAll('.crumb')
					crumbs.forEach((f) => { f.classList.remove('crumb--hovered') })
					target.classList.add('crumb--hovered')
				}
			}
		},
		/**
		 * Handles the drag leave event
		 *
		 * @param {Object} e The drag leave event
		 * @param {boolean} disabled Whether dropping is disabled for this breadcrumb
		 */
		dragLeave(e, disabled) {
			/**
			 * Don't do anything if dropping is disabled.
			 */
			if (disabled) {
				return
			}
			// Don't do anything if we leave towards a child element.
			if (e.target.contains(e.relatedTarget)) {
				return
			}
			// Get the correct element, in case we leave directly from a child.
			if (e.target.closest) {
				const target = e.target.closest('.crumb')
				if (target.contains(e.relatedTarget)) {
					return
				}
				if (target.classList && target.classList.contains('crumb')) {
					target.classList.remove('crumb--hovered')
				}
			}
		},
		/**
		 * Check for each crumb if we have to hide it and
		 * add it to the array of all crumbs.
		 *
		 * @param {Array} crumbs The array of all crumbs
		 * @param {Array} newCrumbs The array of the crumbs to hide and add
		 */
		addCrumbs(crumbs, newCrumbs) {
			newCrumbs.forEach((crumb, i) => {
				if (crumb.elm && crumb.elm.classList) {
					if (this.hiddenCrumbs.includes(crumb)) {
						crumb.elm.classList.add('crumb--hidden')
					} else {
						crumb.elm.classList.remove('crumb--hidden')
					}
				}
				crumbs.push(crumb)
			})
		},
	},
	/**
	 * The render function to display the component
	 *
	 * @param {Function} createElement The function to create VNodes
	 * @returns {VNodes} The created VNodes
	 */
	render: function(createElement) {
		// Get the breadcrumbs
		const breadcrumbs = this.$slots.default || []

		// Check that we have at least one breadcrumb
		if (breadcrumbs.length === 0) {
			return
		}

		// Add the root icon to the first breadcrumb
		Vue.set(breadcrumbs[0].componentOptions.propsData, 'icon', this.rootIcon)

		// The array of all created VNodes
		const crumbs = []
		/**
		 * We show the first half of the breadcrumbs before the Actions dropdown menu
		 * which shows the hidden breadcrumbs.
		 */
		const crumbs1 = this.hiddenCrumbs.length
			? breadcrumbs.slice(0, Math.round(breadcrumbs.length / 2))
			: breadcrumbs
		// Add the breadcrumbs to the array of the created VNodes, check if hiding them is necessary.
		this.addCrumbs(crumbs, crumbs1)

		// The Actions menu
		if (this.hiddenCrumbs.length) {
			// Create the Actions menu
			crumbs.push(createElement('div', { class: 'crumb' },
				[createElement('Actions', {
					class: 'dropdown',
					// We want the Actions to always show as a dropdown menu
					props: {
						forceMenu: true,
					},
					// Add a ref to the Actions menu
					ref: 'actions',
					// Add handlers so the Actions menu opens on hover
					nativeOn: {
						dragstart: this.dragStart,
						dragenter: () => { this.$refs.actions.opened = true },
						dragleave: this.closeActions,
					},
				// Add all hidden breadcrumbs as ActionRouter or ActionLink
				}, this.hiddenCrumbs.map(crumb => {
					// Get the parameters from the breadcrumb component props
					const to = crumb.componentOptions.propsData.to
					const href = crumb.componentOptions.propsData.href
					const disabled = crumb.componentOptions.propsData.disableDrop
					// Decide whether to show the breadcrumbs as ActionRouter or ActionLink
					let element = 'ActionLink'
					let path = href
					if (to) {
						element = 'ActionRouter'
						path = to
					}
					return createElement(element, {
						class: 'crumb',
						props: {
							to: to,
							href: href,
							icon: 'icon-folder',
						},
						// Prevent the breadcrumbs from being draggable
						attrs: {
							draggable: false,
						},
						// Add the drag and drop handlers
						nativeOn: {
							dragstart: this.dragStart,
							drop: ($event) => this.dropped($event, path, disabled),
							dragover: this.dragOver,
							dragenter: ($event) => this.dragEnter($event, disabled),
							dragleave: ($event) => this.dragLeave($event, disabled),
						},
					},
					crumb.componentOptions.propsData.title
					)
				}))]
			))
		}
		// The second half of the breadcrumbs
		const crumbs2 = this.hiddenCrumbs.length
			? breadcrumbs.slice(Math.round(breadcrumbs.length / 2))
			: []
		this.addCrumbs(crumbs, crumbs2)

		return createElement('div', { class: 'breadcrumb', ref: 'container' }, crumbs)
	},
}
</script>

<style lang="scss" scoped>
@import '../../fonts/scss/iconfont-vue';

.breadcrumb {
	width: 100%;
	flex-grow: 1;

	div.crumb {
		@include iconfont('breadcrumb');
		background-image: none;
		display: inline-flex;
		height: $clickable-area;
		padding: 0;

		&::before {
			display: flex;
			align-items: center;
			order: 1;
			color: var(--color-border-dark);
			font-size: 26px;
		}

		&--hidden {
			display: none;
		}
	}

	.crumb--hovered{
		background-color: var(--color-primary-light);
	}
}
</style>