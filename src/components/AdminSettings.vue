<!--
  - @copyright 2019 Christoph Wurst <christoph@winzerhof-wurst.at>
  -
  - @author 2019 Christoph Wurst <christoph@winzerhof-wurst.at>
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
  - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
  - GNU Affero General Public License for more details.
  -
  - You should have received a copy of the GNU Affero General Public License
  - along with this program.  If not, see <http://www.gnu.org/licenses/>.
  -->

<template>
	<div id="ldap-write-support-admin-settings" class="section">
		<h2>{{ t('ldap_write_support', 'Writing') }}</h2>
		<h3>{{ t('ldap_write_support', 'Switches')}}</h3>
		<ul>
			<ActionCheckbox :checked="switches.createPreventFallback"
							@change.stop.prevent="toggleSwitch('createPreventFallback', !switches.createPreventFallback)">
				{{ t('ldap_write_support', 'Prevent fallback to other backends when creating users or groups.')}}
			</ActionCheckbox>
			<ActionCheckbox :checked="switches.createRequireActorFromLdap"
							@change.stop.prevent="toggleSwitch('createRequireActorFromLdap', !switches.createRequireActorFromLdap)">
				{{ t('ldap_write_support', 'To create users, the acting (sub)admin has to be provided by LDAP.')}}
			</ActionCheckbox>
			<ActionCheckbox :checked="switches.newUserGenerateUserID"
							@change.stop.prevent="toggleSwitch('newUserGenerateUserID', !switches.newUserGenerateUserID, 'core')">
				{{ t('ldap_write_support', 'A random user ID has to be generated, i.e. not being provided by the (sub)admin.')}}
			</ActionCheckbox>
			<ActionCheckbox :checked="switches.newUserRequireEmail"
							@change.stop.prevent="toggleSwitch('newUserRequireEmail', !switches.newUserRequireEmail, 'core')">
				{{ t('ldap_write_support', 'An LDAP user must have an email address set.')}}
			</ActionCheckbox>
			<ActionCheckbox :checked="switches.hasAvatarPermission"
							@change.stop.prevent="toggleSwitch('hasAvatarPermission', !switches.hasAvatarPermission)">
				{{ t('ldap_write_support', 'Allow users to set their avatar')}}
			</ActionCheckbox>
		</ul>
		<h3>{{ t('ldap_write_support', 'User template') }}</h3>
		<p>{{ t('ldap_write_support', 'LDIF template for creating users. Following placeholders may be used') }}</p>
		<ul class="disc">
			<li><span class="mono">{UID}</span> – {{ t('ldap_write_support', 'the user id provided by the (sub)admin') }}</li>
			<li><span class="mono">{PWD}</span> – {{ t('ldap_write_support', 'the password provided by the (sub)admin') }}</li>
			<li><span class="mono">{BASE}</span> – {{ t('ldap_write_support', 'the LDAP node of the acting (sub)admin or the configured user base') }}</li>
		</ul>
		<textarea class="mono" v-on:change="setUserTemplate" v-model="templates.user">{{templates.user}}</textarea>
	</div>
</template>

<script>
	import {
		ActionCheckbox,
	} from 'nextcloud-vue';

	export default {
		name: 'AdminSettings',
		props: {
			templates: {
				user: Object,
				userDefault: Object,
			},
			switches: {
				createRequireActorFromLdap: Boolean,
				createPreventFallback: Boolean,
				hasAvatarPermission: Boolean,
				newUserGenerateUserID: Boolean,
				newUserRequireEmail: Boolean,
			}
		},
		components: {
			ActionCheckbox
		},
		methods: {
			setUserTemplate() {
				if(this.templates.user === "") {
					let self = this;
					OCP.AppConfig.deleteKey('ldap_write_support', 'template.user', {
						success: function() {
							self.templates.user = self.templates.userDefault;
						}
					});
					return;
				}
				OCP.AppConfig.setValue('ldap_write_support', 'template.user', this.templates.user);
			},
			toggleSwitch(prefKey, state, appId = 'ldap_write_support') {
				this.switches[prefKey] = state;
				let value = (state | 0).toString();
				if(appId === 'core') {
					// the database key has a slighlty different style, need to transform
					prefKey = 'newUser.' + prefKey.charAt(7).toLowerCase() + prefKey.slice(8);
					value = value === '1' ? 'yes' : 'no';
				}

				OCP.AppConfig.setValue(appId, prefKey, value);
			}
		}
	}
</script>
