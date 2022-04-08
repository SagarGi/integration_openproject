<template>
	<div id="openproject_prefs" class="section">
		<SettingsTitle />
		<p class="settings-hint">
			{{ t('integration_openproject', 'If you want to allow your Nextcloud users to use OAuth to authenticate to a OpenProject instance, create an application in your OpenProject admin settings and put the Client ID (AppId) and the Client secret below.') }}
			<br><br>
			<span class="icon icon-details" />
			{{ t('integration_openproject', 'Make sure you set the "Redirect URI" to') }}
			<b> {{ redirect_uri }} </b>
		</p>
		<div class="grid-form">
			<label for="openproject-oauth-instance">
				<a class="icon icon-link" />
				{{ t('integration_openproject', 'OpenProject instance address') }}
			</label>
			<input id="openproject-oauth-instance"
				v-model="state.oauth_instance_url"
				type="text"
				:placeholder="t('integration_openproject', 'OpenProject address')"
				@input="onInput(true)">
			<label for="openproject-client-id">
				<a class="icon icon-category-auth" />
				{{ t('integration_openproject', 'Client ID') }}
			</label>
			<input id="openproject-client-id"
				v-model="state.client_id"
				type="password"
				:readonly="readonly"
				:placeholder="t('integration_openproject', 'Client ID of the OAuth app in OpenProject')"
				@focus="readonly = false"
				@input="onInput">
			<label for="openproject-client-secret">
				<a class="icon icon-category-auth" />
				{{ t('integration_openproject', 'Client secret') }}
			</label>
			<input id="openproject-client-secret"
				v-model="state.client_secret"
				type="password"
				:readonly="readonly"
				:placeholder="t('integration_openproject', 'Client secret of the OAuth app in OpenProject')"
				@focus="readonly = false"
				@input="onInput">
			<button v-if="state.nc_oauth_client === null"
				@click="onNextcloudOauthCreateClick">
				{{ t('integration_openproject', 'Create a Nextcloud OAuth client for OpenProject') }}
			</button>
			<span v-if="state.nc_oauth_client === null" />
			<label v-if="state.nc_oauth_client !== null"
				for="nextcloud-client-id">
				<a class="icon icon-category-auth" />
				{{ t('integration_openproject', 'Nextcloud client ID') }}
			</label>
			<input v-if="state.nc_oauth_client !== null"
				id="openproject-client-id"
				:value="state.nc_oauth_client.clientId"
				type="text"
				:readonly="true">
			<label v-if="state.nc_oauth_client !== null"
				for="nextcloud-client-secret">
				<a class="icon icon-category-auth" />
				{{ t('integration_openproject', 'Nextcloud client secret') }}
			</label>
			<input v-if="state.nc_oauth_client !== null"
				id="openproject-client-secret"
				:value="state.nc_oauth_client.clientSecret"
				type="text"
				:readonly="true">
		</div>
	</div>
</template>

<script>
import { loadState } from '@nextcloud/initial-state'
import { generateUrl } from '@nextcloud/router'
import axios from '@nextcloud/axios'
import { delay } from '../utils'
import { showSuccess, showError } from '@nextcloud/dialogs'
import '@nextcloud/dialogs/styles/toast.scss'
import SettingsTitle from '../components/settings/SettingsTitle'

export default {
	name: 'AdminSettings',

	components: {
		SettingsTitle,
	},

	data() {
		return {
			state: loadState('integration_openproject', 'admin-config'),
			// to prevent some browsers to fill fields with remembered passwords
			readonly: true,
			redirect_uri: window.location.protocol + '//' + window.location.host + generateUrl('/apps/integration_openproject/oauth-redirect'),
		}
	},
	methods: {
		onInput(resetNcOauthClient = false) {
			const that = this
			delay(() => {
				that.saveOptions()
			}, 2000)()
			if (resetNcOauthClient) {
				this.state.nc_oauth_client = null
			}
		},
		saveOptions() {
			const req = {
				values: {
					client_id: this.state.client_id,
					client_secret: this.state.client_secret,
					oauth_instance_url: this.state.oauth_instance_url,
				},
			}
			const url = generateUrl('/apps/integration_openproject/admin-config')
			axios.put(url, req)
				.then((response) => {
					showSuccess(t('integration_openproject', 'OpenProject admin options saved'))
				})
				.catch((error) => {
					showError(
						t('integration_openproject', 'Failed to save OpenProject admin options')
						+ ': ' + error.response.request.responseText
					)
				})
		},
		onNextcloudOauthCreateClick() {
			const url = generateUrl('/apps/integration_openproject/nc-oauth')
			axios.post(url).then((response) => {
				this.state.nc_oauth_client = response.data
			}).catch((error) => {
				showError(
					t('integration_openproject', 'Failed to create Nextcloud OAuth client')
					+ ': ' + error.response.request.responseText
				)
			})
		},
	},
}
</script>

<style scoped lang="scss">
.grid-form label {
	line-height: 38px;
}

.grid-form input {
	width: 100%;
}

.grid-form {
	max-width: 500px;
	display: grid;
	grid-template: 1fr / 1fr 1fr;
	margin-left: 30px;
}

#openproject_prefs .icon {
	display: inline-block;
	width: 32px;
}

#openproject_prefs .grid-form .icon {
	margin-bottom: -3px;
}
</style>
