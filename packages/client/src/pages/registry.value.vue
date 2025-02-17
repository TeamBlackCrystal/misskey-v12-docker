<template>
<MkStickyContainer>
	<template #header><MkPageHeader :actions="headerActions" :tabs="headerTabs"/></template>
	<MkSpacer :content-max="600" :margin-min="16">
		<FormInfo warn>{{ $ts.editTheseSettingsMayBreakAccount }}</FormInfo>

		<template v-if="value">
			<FormSplit>
				<MkKeyValue class="_formBlock">
					<template #key>{{ $ts._registry.domain }}</template>
					<template #value>{{ $ts.system }}</template>
				</MkKeyValue>
				<MkKeyValue class="_formBlock">
					<template #key>{{ $ts._registry.scope }}</template>
					<template #value>{{ scope.join('/') }}</template>
				</MkKeyValue>
				<MkKeyValue class="_formBlock">
					<template #key>{{ $ts._registry.key }}</template>
					<template #value>{{ key }}</template>
				</MkKeyValue>
			</FormSplit>
			
			<FormTextarea v-model="valueForEditor" tall class="_formBlock _monospace">
				<template #label>{{ $ts.value }} (JSON)</template>
			</FormTextarea>

			<MkButton class="_formBlock" primary @click="save"><i class="fas fa-save"></i> {{ $ts.save }}</MkButton>

			<MkKeyValue class="_formBlock">
				<template #key>{{ $ts.updatedAt }}</template>
				<template #value><MkTime :time="value.updatedAt" mode="detail"/></template>
			</MkKeyValue>

			<MkButton danger @click="del"><i class="fas fa-trash"></i> {{ $ts.delete }}</MkButton>
		</template>
	</MkSpacer>
</MkStickyContainer>
</template>

<script lang="ts" setup>
import { ref, watch } from 'vue';
import JSON5 from 'json5';
import * as os from '@/os';
import { i18n } from '@/i18n';
import { definePageMetadata } from '@/scripts/page-metadata';
import FormLink from '@/components/form/link.vue';
import FormSection from '@/components/form/section.vue';
import MkButton from '@/components/ui/button.vue';
import MkKeyValue from '@/components/key-value.vue';
import FormTextarea from '@/components/form/textarea.vue';
import FormSplit from '@/components/form/split.vue';
import FormInfo from '@/components/ui/info.vue';

const props = defineProps<{
	path: string;
}>();

const scope = $computed(() => props.path.split('/').slice(0, -1));
const key = $computed(() => props.path.split('/').at(-1));

let value = $ref(null);
let valueForEditor = $ref(null);

function fetchValue() {
	os.api('i/registry/get-detail', {
		scope,
		key,
	}).then(res => {
		value = res;
		valueForEditor = JSON5.stringify(res.value, null, '\t');
	});
}

async function save() {
	try {
		JSON5.parse(valueForEditor);
	} catch (err) {
		os.alert({
			type: 'error',
			text: i18n.ts.invalidValue,
		});
		return;
	}
	os.confirm({
		type: 'warning',
		text: i18n.ts.saveConfirm,
	}).then(({ canceled }) => {
		if (canceled) return;
		os.apiWithDialog('i/registry/set', {
			scope,
			key,
			value: JSON5.parse(valueForEditor),
		});
	});
}

function del() {
	os.confirm({
		type: 'warning',
		text: i18n.ts.deleteConfirm,
	}).then(({ canceled }) => {
		if (canceled) return;
		os.apiWithDialog('i/registry/remove', {
			scope,
			key,
		});
	});
}

watch(() => props.path, fetchValue, { immediate: true });

const headerActions = $computed(() => []);

const headerTabs = $computed(() => []);

definePageMetadata({
	title: i18n.ts.registry,
	icon: 'fas fa-cogs',
});
</script>

<style lang="scss" scoped>
</style>
