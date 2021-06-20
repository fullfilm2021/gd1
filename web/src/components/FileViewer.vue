<template>
	<v-container fluid>
		<portal to="navbar">
			<v-toolbar-items class="hidden-sm-and-down">
				<template v-for="seg in pathSegments">
					<v-icon :key="seg.path + '-icon'">mdi-menu-right</v-icon>
					<v-btn
						text
						class="text-none"
						:key="seg.path + '-btn'"
						@click="goPath(seg.path)"
						>{{ seg.name }}</v-btn
					>
				</template>
			</v-toolbar-items>
		</portal>
		<FileUploadDialog
			v-model="showUploadDialog"
			:uploadUrl="uploadUrl"
			@uploaded="uploadComplete"
		/>
		<v-row justify="center" v-if="uploadEnabled">
			<v-col md="8" lg="6" class="pt-0 pb-0">
				<v-btn
					v-text="$t('upload')"
					color="primary"
					@click="showUploadDialog = true"
				></v-btn>
			</v-col>
		</v-row>
		<v-row justify="center">
			<v-col md="8" lg="6">
				<v-card class="mx-auto" tile dark :loading="loading">
					<v-list-item three-line>
						<v-list-item-content>
							<div class="text-overline mb-4">ADMIN</div>
							<v-list-item-title class="text-h5 mb-1">
								Avinandan Jana
							</v-list-item-title>
							<v-list-item-subtitle class="font-weight-thin">
								admin@avinandan.com
							</v-list-item-subtitle>
						</v-list-item-content>

						<v-list-item-avatar size="100">
							<img
								src="https://i.imgur.com/M6NHJri.png"
								alt="Avinandan Jana"
							/>
						</v-list-item-avatar>
					</v-list-item>

					<v-card-actions>
						<v-btn
							icon
							tag="a"
							href="https://www.facebook.com/avinandanjana"
							target="_blank"
							><v-icon color="#4267B2"
								>mdi-facebook</v-icon
							></v-btn
						>
						<v-btn
							icon
							tag="a"
							href="https://wa.me/918016181551"
							target="_blank"
							><v-icon color="#4ac959"
								>mdi-whatsapp</v-icon
							></v-btn
						>
						<v-btn
							icon
							tag="a"
							href="https://www.instagram.com/avinandanjana"
							target="_blank"
							><v-img
								src="https://brandpalettes.com/wp-content/uploads/2018/10/Instagram-300x300.png"
								max-width="24"
							></v-img
						></v-btn>
						<v-btn
							icon
							tag="a"
							href="https://www.twitter.com/ghanajana"
							target="_blank"
							><v-icon color="#1da1f2">mdi-twitter</v-icon></v-btn
						>

						<v-btn
							tag="a"
							href="whatsapp://send?text=Latest HD Movies Download! [Super Fast Download] → https://bit.ly/3vH6eaM"
							data-action="share/whatsapp/share"
							icon
						>
							<v-icon>mdi-share-all</v-icon>
						</v-btn>
					</v-card-actions>
					<v-divider class="mx-5"></v-divider>
					<v-card-text class="font-weight-thin" v-for="seg in pathSegments" :key="seg.path" v-once>{{ seg.name }}</v-card-text>
					<v-divider class="mx-5"></v-divider>
					<template slot="progress">
						<v-progress-linear
							color="red accent-3"
							height="10"
							indeterminate
							rounded
						></v-progress-linear>
					</template>
					<v-list-item
						v-for="item in list"
						:key="item.id"
						@click.prevent="goPath(item.resourcePath, item.opener)"
						class="pl-0"
						tag="a"
						:href="getFileUrl(item.resourcePath)"
					>
						<v-list-item-avatar class="ma-0">
							<v-icon>{{ item.icon }}</v-icon>
						</v-list-item-avatar>
						<v-list-item-content class="py-2">
							<v-list-item-title
								v-text="item.fileName"
							></v-list-item-title>
							<v-list-item-subtitle v-if="!item.isFolder" class="font-weight-thin"
								>Size: {{ item.fileSize }} | Uploaded:
								{{ item.modifiedTime }}</v-list-item-subtitle
							>
							<v-list-item-subtitle
								class="font-weight-thin"
								v-if="item.isFolder"
								>Created:
								{{ item.modifiedTime }}</v-list-item-subtitle
							>
						</v-list-item-content>
						<v-list-item-action>
							<v-btn
								icon
								v-if="!item.isFolder && !item.isGoogleFile"
								tag="a"
								:href="getFileUrl(item.resourcePath)"
								download
								@click.stop
							>
								<v-icon dark> mdi-cloud-download </v-icon>
							</v-btn>
							<v-icon v-if="item.isFolder" dark right
								>mdi-folder-open</v-icon
							>
						</v-list-item-action>
					</v-list-item>
				</v-card>
			</v-col>
		</v-row>
	</v-container>
</template>
<script>
import { format } from 'date-fns'
import prettyBytes from 'pretty-bytes'
import nodeUrl from 'url'
import nodePath from 'path'
import api from '../api'
import ImageViewer from 'viewerjs'
import 'viewerjs/dist/viewer.css'
import FileUploadDialog from './FileUploadDialog'

const SUPPORTED_TYPES = {
	'application/epub+zip': 'epub',
	'video/mp4': 'video',
	'video/mkv': 'video',
	'image/png': 'image',
	'image/jpeg': 'image',
	'image/gif': 'image',
	'image/bmp': 'image',
	'application/pdf': 'pdf',
}
const ICON_NAME = {
	'application/vnd.google-apps.folder': 'mdi-folder',
	'application/epub+zip': 'mdi-book',
	'application/vnd.android.package-archive': 'mdi-android',
	'video/mp4': 'mdi-video',
	'video/x-msvideo': 'mdi-video',
	'video/x-flv': 'mdi-video',
	'video/x-ms-wmv': 'mdi-video',
	'video/webm': 'mdi-video',
	'video/x-matroska': 'mdi-video',
	'application/zip': 'mdi-archive',
	'application/x-7z-compressed': 'mdi-archive',
	'application/x-rar-compressed': 'mdi-archive',
	'application/x-gzip': 'mdi-archive',
	'image/png': 'mdi-file-image',
	'image/jpeg': 'mdi-file-image',
	'image/gif': 'mdi-file-image',
	'image/bmp': 'mdi-file-image',
	'application/msword': 'mdi-file-word',
	'application/vnd.openxmlformats-officedocument.wordprocessingml.document':
		'mdi-file-word',
	'application/vnd.ms-excel': 'mdi-file-excel',
	'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet':
		'mdi-file-excel',
	'application/vnd.openxmlformats-officedocument.presentationml.presentation':
		'mdi-file-powerpoint',
	'application/vnd.ms-powerpoint': 'mdi-file-powerpoint',
	'application/pdf': 'mdi-file-pdf',
	'text/x-sql': 'mdi-database',
	'application/vnd.google-apps.document': 'mdi-file-document-box',
	'application/vnd.google-apps.spreadsheet': 'mdi-google-spreadsheet',
	'application/vnd.google-apps.presentation': 'mdi-file-presentation-box',
	'text/plain': 'mdi-file-document',
}
export default {
	data() {
		return {
			list: [],
			loading: false,
			headers: [
				{
					text: this.$t('fileName'),
					value: 'fileName',
					class: ['fileName'],
				},
				{
					text: this.$t('modifiedTime'),
					value: 'modifiedTime',
					filterable: false,
					class: 'hidden-sm-and-down',
				},
				{
					text: this.$t('fileSize'),
					value: 'fileSize',
					filterable: false,
					class: 'hidden-sm-and-down',
				},
			],
			renderStart: null,
			uploadEnabled: window.props.upload,
			showUploadDialog: false,
		}
	},
	computed: {
		path() {
			return '/' + this.$route.params.path
		},
		pathSegments() {
			const list = this.path
				.split('/')
				.filter(Boolean)
				.map(decodeURIComponent)
			const ar = []
			for (let i = 0; i < list.length; i++) {
				ar.push({
					name: list[i],
					path: '/' + nodePath.join(...list.slice(0, i + 1)) + '/',
				})
			}
			return ar
		},
		uploadUrl() {
			const u = new URL(this.path, window.props.api)
			u.searchParams.set(
				'rootId',
				this.$route.query.rootId || window.props.default_root_id
			)
			return u.href
		},
	},
	methods: {
		goPath(path, opener) {
			const query = {
				rootId: this.$route.query.rootId,
			}
			if (opener) {
				query.opener = opener
			}
			this.$router.push({
				path: path
					.split('/')
					.map(decodeURIComponent)
					.map(encodeURIComponent)
					.join('/'),
				query,
			})
		},
		getFileUrl(path) {
			const { rootId } = this.$route.query
			let u = nodeUrl.resolve(
				window.props.api,
				path.split('/').map(encodeURIComponent).join('/')
			)
			if (rootId) {
				u += '?rootId=' + rootId
			}
			return u
		},
		async renderPath(path, rootId) {
			let renderStart = (this.renderStart = Date.now()) // Withous this, when user regret navigating a big folder, it will have some conflict.
			this.loading = true
			if (!rootId) {
				rootId = window.props.default_root_id
			}
			this.list = []
			const { files } = await api
				.post(path, {
					method: 'POST',
					qs: {
						rootId,
					},
				})
				.json()
			if (renderStart !== this.renderStart) {
				// User had initiated other folder navigation request
				return
			}
			this.list = files.map((f) => {
				f.mimeType = f.mimeType.replace('; charset=utf-8', '')
				const isFolder =
					f.mimeType === 'application/vnd.google-apps.folder'
				const isGoogleFile = f.mimeType.includes('vnd.google-apps')
				const resourcePath =
					nodeUrl.resolve(path, f.name) + (isFolder ? '/' : '')
				const o = {
					fileName: f.name,
					modifiedTime: format(
						new Date(f.modifiedTime),
						'dd/MM/yyyy HH:mm:ss'
					),
					isFolder,
					isGoogleFile,
					mimeType: f.mimeType,
					fileSize: f.size ? prettyBytes(parseInt(f.size)) : '',
					resourcePath,
					icon: ICON_NAME[f.mimeType] || 'mdi-file',
				}
				if (f.mimeType in SUPPORTED_TYPES) {
					o.opener = SUPPORTED_TYPES[f.mimeType]
				}
				return o
			})
			this.loading = false
		},
		handlePath(path, query) {
			if (path.substr(-1) === '/') {
				this.renderPath(path, query.rootId)
				return true
			} else {
				let u = nodeUrl.resolve(window.props.api, path)
				//if (Math.random() < 10) return
				if (
					query.rootId &&
					query.rootId !== window.props.default_root_id
				) {
					u += '?rootId=' + query.rootId
				}
				if (query.opener) {
					if (query.opener === 'image') {
						const img = new Image()
						img.src = u
						img.style.display = 'none'
						document.body.appendChild(img)
						img.onload = () => {
							const viewer = new ImageViewer(img)
							viewer.show()
							img.addEventListener('hide', () => {
								viewer.destroy()
								img.remove()
							})
						}

						return
					}
					this.$router.push({
						path: '/~viewer/' + query.opener,
						query: { urlBase64: btoa(u) },
					})
				} else {
					location.href = u
				}
			}
		},
		uploadComplete() {
			this.showUploadDialog = false
			this.renderPath(this.path, this.$route.query.rootId)
		},
	},
	created() {
		this.handlePath(this.path, this.$route.query)
	},
	beforeRouteUpdate(to, from, next) {
		const fullyEncoded = to.params.path
			.split('/')
			.map(decodeURIComponent)
			.map(encodeURIComponent)
			.join('/') // because vue-router's encoding is a little bit weird...
		if (this.handlePath('/' + fullyEncoded, to.query)) {
			next()
		}
	},
	components: {
		FileUploadDialog,
	},
}
</script>
<style scoped>
.fake-tr {
	display: table-row;
	vertical-align: inherit;
	border-color: inherit;
	color: inherit;
	text-decoration: none;
}
.theme--light.v-data-table
	tbody
	.fake-tr:hover:not(.v-data-table__expanded__content) {
	background: #eeeeee;
}
.theme--light.v-data-table
	tbody
	.fake-tr:not(:last-child)
	td:not(.v-data-table__mobile-row) {
	border-bottom: 1px solid rgba(0, 0, 0, 0.12);
}
.line-height {
	height: 48px;
	line-height: 48px;
}
.wrapper {
	display: flex;
	align-items: center;
}
.icon-wrapper {
	box-sizing: border-box;
	overflow: hidden;
	text-overflow: ellipsis;
	white-space: nowrap;
}
</style>
