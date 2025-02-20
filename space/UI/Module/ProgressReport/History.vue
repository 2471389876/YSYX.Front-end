<script setup>
import Dialog from "/components/AppView/Panes/Dialog.vue";
import Button from "/components/Button.vue";
import Paragraph from "/components/Paragraph.vue";
import RefreshButton from "/components/Button/RefreshButton.vue";
import InspectInfo from "./InspectInfo.vue";
import InspectState from "./InspectState.vue";
</script>

<template>
	<div
		Content _1024_
		v-if="content && content.length"
	>
		<span w100 v-for="el in content" :key="el.ID">
			<div
				:class="
					{
						desktop: 'card list-entry shadow-light shadow-dynamic',
						mobile: 'card list-seamless',
					}[env.platform]
				"
			>
				<div title>
					<div class="large">{{ titleFormat(el.ID).Date }}&nbsp;</div>
					<div class="small">{{ titleFormat(el.ID).Year }}</div>
					<div style="flex-grow: 1"></div>
					<div class="suffix">
						<span v-if="el.submitTime != el.updateTime">
							<span zh-CN>最后更新于</span>
							<span en-US>Updated at</span>
						</span>
						<span v-else>
							<span zh-CN>创建于</span>
							<span en-US>Created at</span>
						</span>
						&nbsp;
						<span>{{ localeDate(el.updateTime) }}</span>
					</div>
				</div>
				<div content>
					<h5 w100>
						<span en-US
							>Your progress report content for this day</span
						>
						<span zh-CN>当日的进度报告内容</span>
					</h5>
					<Paragraph
						v-for="(text, i) in el.body"
						:key="i"
						:text="text"
					/>
				</div>
				<InspectInfo
					:inspections="el.inspections"
					v-if="el.checked.length"
				/>
				<InspectState :state="el.state" :pending="el.unchecked" />
			</div>
		</span>
		<RefreshButton @refresh="fetch" />
	</div>
	<Dialog
		v-else
		class="gray"
		style="opacity: 0.8"
		:title="
			intl({
				'en-US': 'No record found',
				'zh-CN': '没有记录',
			})
		"
		:suffix="
			intl({
				'en-US':
					'You haven\'t submit any report yet',
				'zh-CN': '您没有提交过进度报告',
			})
		"
		:buttons="[
			{
				name: intl({ 'en-US': 'Refresh', 'zh-CN': '刷新' }),
				type: 'outlined gray',
				callback: fetch,
			},
		]"
	/>
</template>

<script>
import { Session } from "/space/Session.js";
import { monthShort, localeDate } from "/util/date.js";
import { digVal } from "/util/object.js";
import { env, intl } from "/util/env.js";

export default {
	data() {
		return {
			env,
			lang: Session.language,
			content: [],
		};
	},
	computed: {
		_name_() {
			return "ProgressReport/JSON";
		},
	},
	methods: {
		intl,
		fetch() {
			Session.post("ProgressReport/JSON").then((content) => {
				this.content = Object.keys(content)
					.sort((a, b) => {
						const A = a.split("-").map((el) => parseInt(el)),
							B = b.split("-").map((el) => parseInt(el));
						for (let i = 0; i < 3; i++) {
							if (A[i] !== B[i]) return B[i] - A[i];
						}
						return 0;
					})
					.map((ID) => ({
						ID: ID,
						...((reportEntry) => {
							const { fields, inspections } = reportEntry;
							let checked = Object.keys(fields).filter(
									(field) => fields[field]
								),
								unchecked = Object.keys(fields).filter(
									(field) => checked.indexOf(field) < 0
								),
								state = ((a, b) => {
									if (a == 0 && b == 0) return "empty";
									if (a > 0 && b == 0) return "complete";
									return "pending";
								})(checked.length, unchecked.length);
							for (const inspectorID in inspections) {
								const inspection = inspections[inspectorID];
								["public", "private"].forEach((entry) => {
									if (Array.isArray(inspection[entry]))
										inspection[entry] = inspection[
											entry
										].filter((str) => str.trim());
								});
								inspection.fields = Object.keys(fields).filter(
									(fieldName) =>
										digVal(
											fields,
											fieldName,
											"inspectorID"
										) === inspectorID
								);
							}
							console.log({
								state,
								checked,
								unchecked,
								...reportEntry,
							});
							return {
								state,
								checked,
								unchecked,
								...reportEntry,
							};
						})(content[ID]),
					}));
			});
			// this.$forceUpdate();
		},
		localeDate,
		titleFormat(ID) {
			const [Y, M, D] = ID.split("-");
			return {
				Date: `${monthShort(M)} ${D}`,
				Year: Y,
			};
		},
	},
	activated() {
		console.log(this);
		this.fetch();
	},
};
</script>

<style scoped>
[content] {
	margin-top: 0.5em;
}

h4,
h5 {
	margin-bottom: 1em;
}
</style>