<template>
<div class="pwnqwyet">
	<ui-card>
		<div slot="title"><fa :icon="faCloud"/> {{ $t('@.drive') }}</div>
		<section class="fit-top">
			<ui-horizon-group inputs>
				<ui-select v-model="sort">
					<span slot="label">{{ $t('sort.title') }}</span>
					<option value="-createdAt">{{ $t('sort.createdAtAsc') }}</option>
					<option value="+createdAt">{{ $t('sort.createdAtDesc') }}</option>
					<option value="-size">{{ $t('sort.sizeAsc') }}</option>
					<option value="+size">{{ $t('sort.sizeDesc') }}</option>
				</ui-select>
				<ui-select v-model="origin">
					<span slot="label">{{ $t('origin.title') }}</span>
					<option value="combined">{{ $t('origin.combined') }}</option>
					<option value="local">{{ $t('origin.local') }}</option>
					<option value="remote">{{ $t('origin.remote') }}</option>
				</ui-select>
			</ui-horizon-group>
			<div class="kidvdlkg" v-for="file in files">
				<div @click="file._open = !file._open">
					<div>
						<div class="thumbnail" :style="thumbnail(file)"></div>
					</div>
					<div>
						<header>
							<b>{{ file.name }}</b>
							<span class="username">@{{ file.user | acct }}</span>
						</header>
						<div>
							<div>
								<span style="margin-right:16px;">{{ file.type }}</span>
								<span>{{ file.datasize | bytes }}</span>
							</div>
							<div><mk-time :time="file.createdAt" mode="detail"/></div>
						</div>
					</div>
				</div>
				<div v-show="file._open">
					<ui-button @click="del(file)"><fa :icon="faTrashAlt"/> {{ $t('delete') }}</ui-button>
				</div>
			</div>
			<ui-button v-if="existMore" @click="fetch">{{ $t('@.load-more') }}</ui-button>
		</section>
	</ui-card>
</div>
</template>

<script lang="ts">
import Vue from 'vue';
import i18n from '../../i18n';
import { faCloud } from '@fortawesome/free-solid-svg-icons';
import { faTrashAlt } from '@fortawesome/free-regular-svg-icons';

export default Vue.extend({
	i18n: i18n('admin/views/drive.vue'),

	data() {
		return {
			sort: '+createdAt',
			origin: 'combined',
			limit: 10,
			offset: 0,
			files: [],
			existMore: false,
			faCloud, faTrashAlt
		};
	},

	watch: {
		sort() {
			this.files = [];
			this.offset = 0;
			this.fetch();
		},

		origin() {
			this.files = [];
			this.offset = 0;
			this.fetch();
		}
	},

	mounted() {
		this.fetch();
	},

	methods: {
		fetch() {
			this.$root.api('admin/drive/files', {
				origin: this.origin,
				sort: this.sort,
				offset: this.offset,
				limit: this.limit + 1
			}).then(files => {
				if (files.length == this.limit + 1) {
					files.pop();
					this.existMore = true;
				} else {
					this.existMore = false;
				}
				for (const x of files) {
					x._open = false;
				}
				this.files = this.files.concat(files);
				this.offset += this.limit;
			});
		},

		thumbnail(file: any): any {
			return {
				'background-color': file.properties.avgColor && file.properties.avgColor.length == 3 ? `rgb(${file.properties.avgColor.join(',')})` : 'transparent',
				'background-image': `url(${file.thumbnailUrl})`
			};
		},

		async del(file: any) {
			const process = async () => {
				await this.$root.api('drive/files/delete', { fileId: file.id });
				this.$root.dialog({
					type: 'success',
					text: this.$t('deleted')
				});
			};

			await process().catch(e => {
				this.$root.dialog({
					type: 'error',
					text: e.toString()
				});
			});
		}
	}
});
</script>

<style lang="stylus" scoped>
.pwnqwyet
	@media (min-width 500px)
		padding 16px

	.kidvdlkg
		padding 16px 0
		border-top solid 1px var(--faceDivider)

		> div:first-child
			display flex
			cursor pointer

			> div:nth-child(1)
				> .thumbnail
					display block
					width 64px
					height 64px
					background-size cover
					background-position center center

			> div:nth-child(2)
				flex 1
				padding-left 16px

				@media (max-width 500px)
					font-size 14px

				> header
					word-break break-word

					> .username
						margin-left 8px
						opacity 0.7

</style>