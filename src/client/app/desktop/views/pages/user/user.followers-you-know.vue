<template>
<div class="vahgrswmbzfdlmomxnqftuueyvwaafth">
	<p class="title">%fa:users%%i18n:@title%</p>
	<p class="initializing" v-if="fetching">%fa:spinner .pulse .fw%%i18n:@loading%<mk-ellipsis/></p>
	<div v-if="!fetching && users.length > 0">
	<router-link v-for="user in users" :to="user | userPage" :key="user.id">
		<img :src="user.avatarUrl" :alt="user | userName" v-user-preview="user.id"/>
	</router-link>
	</div>
	<p class="empty" v-if="!fetching && users.length == 0">%i18n:@no-users%</p>
</div>
</template>

<script lang="ts">
import Vue from 'vue';

export default Vue.extend({
	props: ['user'],
	data() {
		return {
			users: [],
			fetching: true
		};
	},
	mounted() {
		(this as any).api('users/followers', {
			userId: this.user.id,
			iknow: true,
			limit: 16
		}).then(x => {
			this.users = x.users;
			this.fetching = false;
		});
	}
});
</script>

<style lang="stylus" scoped>
root(isDark)
	background isDark ? #282C37 : #fff
	border solid 1px rgba(#000, 0.075)
	border-radius 6px

	> .title
		z-index 1
		margin 0
		padding 0 16px
		line-height 42px
		font-size 0.9em
		font-weight bold
		color isDark ? #e3e5e8 : #888
		box-shadow 0 1px rgba(#000, 0.07)

		> i
			margin-right 4px

	> div
		padding 8px

		> a
			display inline-block
			margin 4px

			> img
				width 48px
				height 48px
				vertical-align bottom
				border-radius 100%

	> .initializing
	> .empty
		margin 0
		padding 16px
		text-align center
		color #aaa

		> i
			margin-right 4px

.vahgrswmbzfdlmomxnqftuueyvwaafth[data-darkmode]
	root(true)

.vahgrswmbzfdlmomxnqftuueyvwaafth:not([data-darkmode])
	root(false)

</style>
