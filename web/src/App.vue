<template>
	<v-app>
		<v-app-bar app dark class="elevation-1">
			<v-toolbar-title class="headline pointer mr-3">
				<router-link
					:to="{ path: '/', query: { rootId: $route.query.rootId } }"
					tag="span"
					>{{ title }}</router-link
				>
			</v-toolbar-title>
			<v-toolbar-items>
				<v-menu offset-y v-if="drives.length">
					<template v-slot:activator="{ on }">
						<v-btn text v-on="on" class="text-none">
							<v-icon left>mdi-cloud</v-icon>
							{{ currentDrive.text
							}}<v-icon>mdi-menu-down</v-icon>
						</v-btn>
					</template>
					<v-list>
						<v-list-item
							v-for="(item, index) in drives"
							:key="index.id"
							@click="changeDrive(item.value)"
						>
							<v-list-item-title>{{
								item.text
							}}</v-list-item-title>
						</v-list-item>
					</v-list>
				</v-menu>
			</v-toolbar-items>
			<portal-target name="navbar" slim />
			<v-spacer />
			<v-toolbar-items>
				<v-btn
					text
					class="text-none"
					tag="a"
					href="https://www.facebook.com/avinandanjana"
					target="_blank"
				>
					<v-icon left>mdi-facebook</v-icon> Avinandan Jana</v-btn
				>
			</v-toolbar-items>
		</v-app-bar>
		<!-- <v-row justify="center">
			<v-col md="8" lg="6">
				<v-card class="mx-auto" tile elevation="1"> </v-card>
			</v-col>
		</v-row> -->
		<v-content>
			<v-card
				class="mx-auto mt-2"
				max-width="350"
				tile
				elevation="5"
				dark
			>
				<v-list-item three-line>
					<v-list-item-content>
						<div class="text-overline mb-4">ADMIN</div>
						<v-list-item-title class="text-h5 mb-1">
							Avinandan Jana
						</v-list-item-title>
						<v-list-item-subtitle>
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
						><v-icon color="#4267B2">mdi-facebook</v-icon></v-btn
					>
					<v-btn
						icon
						tag="a"
						href="https://wa.me/918016181551"
						target="_blank"
						><v-icon color="#4ac959">mdi-whatsapp</v-icon></v-btn
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
					<!-- <v-btn
						icon
						tag="a"
						href="https://www.twitter.com/ghanajana"
						target="_blank"
						><v-icon color="#1da1f2">mdi-twitter</v-icon></v-btn
					> -->

					<v-btn icon @click="snack = true">
						<v-icon color="#FF0000">mdi-youtube</v-icon>
					</v-btn>
					<v-btn
						tag="a"
						href="whatsapp://send?text=Latest HD Movies Download! [Super Fast Download] → https://bit.ly/3vH6eaM"
						data-action="share/whatsapp/share"
						icon
					>
						<v-icon>mdi-share-all</v-icon>
					</v-btn>
				</v-card-actions>
			</v-card>
			<router-view />
		</v-content>
		<LoginDialog :show="showAuthInput" />
		<v-snackbar v-model="snack" :timeout="time" top color="error"
			>Sorry, currently its not available!</v-snackbar
		>
	</v-app>
</template>
<script>
import api from './api'
import LoginDialog from './components/LoginDialog.vue'

export default {
	props: {
		title: String,
	},
	data() {
		return {
			drives: [],
			value: {},
			snack: false,
			time: 2000,
			showAuthInput: false,
		}
	},
	computed: {
		currentDrive() {
			const id = this.$route.query.rootId || window.props.default_root_id
			return this.drives.find((d) => d.value === id)
		},
	},
	async created() {
		const ok =
			new URL(window.props.api).hostname === location.hostname ||
			(await api
				.get(window.props.api)
				.then(() => true)
				.catch((err) => {
					if (err.response.status === 401) {
						this.showAuthInput = true
						return false
					}
				}))
		if (!ok) return

		const { drives } = await api.get('/~_~_gdindex/drives').json()
		this.drives = [{ text: this.$t('mainDrive'), value: 'root' }].concat(
			drives.map((d) => ({
				value: d.id,
				text: d.name,
			}))
		)
	},
	methods: {
		changeDrive(drive) {
			const rootId =
				drive !== window.props.default_root_id ? drive : undefined
			const dest = { path: '/', query: { rootId } }
			if (
				dest.path === this.$route.path &&
				dest.query.rootId === this.$route.query.rootId
			) {
				return // vue-router forbid going to same location
			}
			this.$router.push({ path: '/', query: { rootId } })
		},
	},
	components: { LoginDialog },
}
</script>
