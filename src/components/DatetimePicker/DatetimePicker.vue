<!--
  - @copyright Copyright (c) 2018 John Molakvoæ <skjnldsv@protonmail.com>
  -
  - @author John Molakvoæ <skjnldsv@protonmail.com>
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
> We're wrapping the awesome datepicker library here https://github.com/mengxiong10/vue2-datepicker
> Please check there for all the available options.

### Defaults
- cleareable: false
- minute-step: 10

### Example
```vue
<template>
	<span>
		<DatetimePicker
			v-model="time"
			type="datetime" />
		{{ time }}
	</span>
</template>
<script>
export default {
	data() {
		return {
			time: null,
		}
	},
}
</script>
```

### Range picker
```vue
<template>
	<span>
		<DatetimePicker
			v-model="time"
			range
			type="date" />
		{{ time }}
	</span>
</template>
<script>
export default {
	data() {
		return {
			time: null,
		}
	},
}
</script>
```

### Timezone-aware date picker
```vue
<template>
	<span>
		<DatetimePicker
			v-model="time"
			type="datetime"
			:show-timezone-select="true"
			:timezone-id.sync="tz" />
		{{ time }} | {{ tz }}
	</span>
</template>
<script>
export default {
	data() {
		return {
			time: null,
			tz: 'Hawaiian Standard Time',
		}
	},
}
</script>
```
</docs>

<template>
	<DatePicker
		ref="datepicker"
		:clearable="clearable"
		:minute-step="minuteStep"
		:format="format"
		:formatter="formatter"
		:type="type"
		:value="value"
		:append-to-body="appendToBody"
		:show-week-number="showWeekNumber"
		:popup-class="{ 'show-week-number': showWeekNumber }"
		v-bind="$attrs"
		v-on="$listeners"
		@select-year="handleSelectYear"
		@select-month="handleSelectMonth"
		@update:value="$emit('update:value', value)">
		<template v-if="showTimezoneSelect" #icon-calendar>
			<button
				class="datetime-picker-inline-icon icon-timezone icon"
				:class="{'datetime-picker-inline-icon--highlighted': highlightTimezone}"
				@click.stop.prevent="toggleTimezonePopover"
				@mousedown.stop.prevent="() => {}" />
			<Popover
				:open.sync="showTimezonePopover"
				open-class="timezone-popover-wrapper">
				<div class="timezone-popover-wrapper__title">
					<strong>
						{{ t('Please select a timezone:') }}
					</strong>
				</div>
				<TimezoneSelect
					class="timezone-popover-wrapper__timezone-select"
					v-model="tzVal"
					@change="this.$emit('update:timezone-id', arguments[0])" />
			</Popover>
		</template>
		<template v-for="(_, slot) of $scopedSlots" #[slot]="scope">
			<slot :name="slot" v-bind="scope" />
		</template>
	</DatePicker>
</template>

<script>
import DatePicker from 'vue2-datepicker'

import Popover from '../Popover/index'
import TimezoneSelect from '../TimezonePicker'

export default {
	name: 'DatetimePicker',

	components: {
		DatePicker,
		Popover,
		TimezoneSelect,
	},

	inheritAttrs: false,

	props: {
		clearable: {
			type: Boolean,
			default() {
				return false
			},
		},

		minuteStep: {
			type: Number,
			default() {
				return 10
			},
		},

		type: {
			type: String,
			default: 'date',
		},

		format: {
			type: String,
			default() {
				const map = {
					date: 'YYYY-MM-DD',
					datetime: 'YYYY-MM-DD H:mm:ss',
					year: 'YYYY',
					month: 'YYYY-MM',
					time: 'H:mm:ss',
					week: 'w',
				}
				return map[this.type] || map.date
			},
		},

		formatter: {
			type: Object,
			default() {
				return null
			},
		},

		// eslint-disable-next-line
		value: {
			default() {
				return new Date()
			},
		},

		timezoneId: {
			type: String,
			default: 'floating'
		},

		showTimezoneSelect: {
			type: Boolean,
			default: false,
		},

		highlightTimezone: {
			type: Boolean,
			default: false,
		},

		appendToBody: {
			type: Boolean,
			default: false,
		},

		showWeekNumber: {
			type: Boolean,
			default: false,
		},
	},

	data() {
		return {
			showTimezonePopover: false,
			tzVal: this.timezoneId
		}
	},

	methods: {
		handleSelectYear(year) {
			const value = this.$refs.datepicker.currentValue
			if (value) {
				try {
					const date = new Date(new Date(value).setFullYear(year))
					this.$refs.datepicker.selectDate(date)
				} catch (error) {
					console.error('Invalid value', value, year)
				}
			}
		},

		handleSelectMonth(month) {
			const value = this.$refs.datepicker.currentValue
			if (value) {
				try {
					const date = new Date(new Date(value).setMonth(month))
					this.$refs.datepicker.selectDate(date)
				} catch (error) {
					console.error('Invalid value', value, month)
				}
			}
		},

		/**
		 * Toggles the visibility of the timezone popover
		 */
		toggleTimezonePopover() {
			if (!this.showTimezoneSelect) {
				// Just a click on the icon, but not for timezones -> ignore
				return
			}

			this.showTimezonePopover = !this.showTimezonePopover
		},
	},
}
</script>

<style lang="scss" scoped>
.datetime-picker-inline-icon {
	margin-top: 17px;
	opacity: .3;
	border: none;
	background-color: transparent;
	border-radius: 0;
	padding: 6px !important;

	&--highlighted {
		opacity: .7;
	}

	&:focus,
	&:hover {
		opacity: 1;
	}
}
</style>
