<template lang="html">
	<div class="ffz--badge-visibility tw-pd-t-05">
		<div
			v-if="source && source !== profile"
			class="tw-c-background-accent tw-c-text-overlay tw-pd-1 tw-mg-b-1"
		>
			<span class="ffz-i-info" />
			{{ t('setting.warn-inheritence', 'These values are being overridden by another profile and may not take effect.') }}
		</div>

		<div class="tw-mg-b-2 tw-align-right">
			<button
				:disabled="! has_value"
				:class="{'tw-button--disabled': ! has_value}"
				class="tw-mg-l-05 tw-button ffz-button--hollow ffz-il-tooltip__container"
				@click="clear"
			>
				<span class="tw-button__icon tw-button__icon--left">
					<figure class="ffz-i-cancel" />
				</span>
				<span class="tw-button__text">
					{{ t('setting.reset-all', 'Reset All to Default') }}
				</span>
			</button>
		</div>

		<section
			v-for="sec in badges"
			:key="sec.title"
			class="ffz--menu-container tw-border-t"
		>
			<header
				v-if="sec.id"
				:class="{default: badgeDefault(sec.id)}"
				class="tw-flex ffz-checkbox tw-align-items-center tw-z-above"
			>
				<input
					:id="sec.id"
					:checked="badgeChecked(sec.id)"
					type="checkbox"
					class="ffz-checkbox__input"
					@click="onChange(sec.id, $event)"
				>
				<label
					:for="sec.id"
					class="ffz-checkbox__label"
				>
					<span class="tw-mg-l-1">
						{{ sec.title }}
					</span>
				</label>

				<div class="ffz--reset-button">
					<button
						v-if="! badgeDefault(sec.id)"
						class="tw-mg-l-05 tw-button tw-button--text ffz-il-tooltip__container"
						@click="reset(sec.id)"
					>
						<span class="tw-button__text ffz-i-cancel" />
						<div class="ffz-il-tooltip ffz-il-tooltip--down ffz-il-tooltip--align-right">
							{{ t('setting.reset', 'Reset to Default') }}
						</div>
					</button>
				</div>
			</header>
			<header v-else>
				{{ sec.title }}
			</header>
			<ul v-if="! sec.id || badgeChecked(sec.id)" class="tw-flex tw-flex-wrap tw-align-content-start tw-align-items-start">
				<li
					v-for="i in sec.badges"
					:key="i.id"
					:class="{default: badgeDefault(i.id)}"
					class="ffz--badge-info ffz--pointer-events--none tw-mg-y-05 tw-mg-r-05 tw-flex ffz-checkbox ffz-interactable ffz-interactable--hover-enabled ffz-interactable--default tw-border-radius-large"
				>
					<input
						:id="i.id"
						:checked="badgeChecked(i.id)"
						type="checkbox"
						class="ffz-checkbox__input"
						@click="onChange(i.id, $event)"
					>

					<label :for="i.id" class="ffz-checkbox__label tw-flex-grow-1 tw-mg-05 ffz--pointer-events">
						<div class="tw-mg-l-1 tw-flex">
							<div
								:style="{backgroundColor: i.color, backgroundImage: i.styleImage }"
								class="preview-image ffz-badge tw-mg-r-1 tw-flex-shrink-0"
							/>
							<div>
								<h5 class="ffz-font-size-5">{{ i.name }}</h5>
								<section
									v-if="i.versions && (i.always_versions || i.versions.length > 1)"
									class="tw-mg-t-05"
								>
									<span
										v-for="v in i.versions"
										:key="`badge-${i.id}-${v.version}`"
										:title="v.name"
										:style="{backgroundColor: i.color, backgroundImage: v.styleImage}"
										data-tooltip-type="html"
										class="ffz-badge ffz-tooltip"
									/>
								</section>
								<button
									class="tw-mg-t-05 tw-button ffz-button--hollow ffz-il-tooltip__container"
									@click="reset(i.id)"
								>
									<span class="tw-button__text ffz-i-cancel" />
									<span class="ffz-il-tooltip ffz-il-tooltip--down ffz-il-tooltip--align-right">
										{{ t('setting.reset', 'Reset to Default') }}
									</span>
								</button>
							</div>
						</div>
					</label>
				</li>
			</ul>
			<div v-else class="tw-c-text-alt-2 ffz-font-size-4 tw-align-center tw-pd-05">
				{{ t('setting.badges.hidden', '{count,plural,one{# badge is} other{# badges are} } hidden in this set', {
					count: sec.badges.length
				}) }}
			</div>
		</section>
	</div>
</template>

<script>

import SettingMixin from '../setting-mixin';
import {has} from 'utilities/object';

function sortBadges(items) {
	return items.sort((a, b) => {
		const an = a.name.toLowerCase(),
			bn = b.name.toLowerCase();

		if ( an < bn ) return -1;
		if ( an > bn ) return 1;
		return 0;
	});
}

export default {
	mixins: [SettingMixin],
	props: ['item', 'context'],

	data() {
		return {
			badges: []
		}
	},

	created() {
		this.updateBadges();
	},

	methods: {
		updateBadges() {
			const badges = this.item.getBadges(() => this.updateBadges());
			for(const section of badges) {
				section.badges = sortBadges(section.badges);
			}

			this.badges = badges;
		},

		badgeChecked(id) {
			return ! this.value[id];
		},

		badgeDefault(id) {
			return ! has(this.value, id);
		},

		onChange(id, event) {
			const control = event.target,
				new_val = {[id]: ! control.checked};

			this.set(Object.assign({}, this.value, new_val));
		},

		reset(id) {
			const val = Object.assign({}, this.value);
			delete val[id];

			if ( ! Object.keys(val).length )
				this.clear();
			else
				this.set(val);
		}
	}
}

</script>
